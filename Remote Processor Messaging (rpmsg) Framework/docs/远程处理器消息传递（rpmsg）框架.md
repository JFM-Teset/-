本文档介绍rpmsg总线以及如何编写rpmsg驱动程序。要了解如何为新平台添加rpmsg支持，请查看remoteproc.txt.[本文参考链接](www.kernel.org/doc/Documentation/rpmsg.txt "With a Title"). 

# 介绍

现代SoC通常采用非对称多处理（AMP）配置的异构远程处理器设备，这些设备可能正在运行不同的操作系统实例，无论是Linux还是任何其他种类的实时OS。

例如，OMAP4具有双Cortex-A9，双Cortex-M3和C64x + DSP。通常，双Cortex-A9在SMP配置中运行Linux，并且其他三个内核（两个M3内核和一个DSP）中的每个内核 在AMP配置中运行自己的RTOS实例。

通常，AMP远程处理器采用专用的DSP编解码器和多媒体硬件加速器，因此通常用于从主应用程序处理器卸载CPU密集型多媒体任务。

这些远程处理器还可用于控制对延迟敏感的传感器，驱动随机硬件块或仅在主CPU空闲时执行后台任务。

这些远程处理器的用户可以是用户态应用程序（例如与远程OMX组件通信的多媒体框架），也可以是内核驱动程序（控制仅可由远程处理器访问的硬件，代表远程处理器保留由内核控制的资源等）。

Rpmsg是基于virtio的消息总线，它允许内核驱动程序与系统上可用的远程处理器进行通信。 反过来，如果需要，驱动程序然后可以公开适当的用户空间接口。

在编写向用户区公开rpmsg通信的驱动程序时，请记住，远程处理器可能直接访问系统的物理内存和其他敏感硬件资源（例如，在OMAP4上，远程核心和硬件加速器可能直接访问物理内存） ，gpio bank，dma控制器，i2c总线，gptimers，邮箱设备，hwspinlocks等）。（待续）



每个rpmsg设备都是与远程处理器的通信通道（因此rpmsg设备称为通道）。 通道由文本名称标识，并具有本地（“源”）rpmsg地址和远程（“目标”）rpmsg地址。

当驱动程序开始在通道上侦听时，其rx回调与唯一的rpmsg本地地址（32位整数）绑定。 这样，当入站消息到达时，rpmsg内核会根据它们的目标地址将它们分派给适当的驱动程序（这是通过使用入站消息的有效负载调用驱动程序的rx处理程序来完成的）。



# 用户API接口

```
::

  int rpmsg_send(struct rpmsg_channel *rpdev, void *data, int len);
```

将消息发送到给定通道上的远程处理器。呼叫者应指定通道地址，要发送的数据，及其长度（以字节为单位）。该消息将在指定的通道上发送，即其源和目标地址字段将设置为该通道的src和dst地址。

如果没有可用的TX缓冲区，则该功能将阻塞直到可用的TX缓冲区可用（即，直到远程处理器消耗了tx缓冲区并将其放回virtio的已使用描述符环上），否则将超时15秒。 发生后者时，将返回                        -ERESTARTSYS。

该功能只能从处理器上下文中调用（目前）。成功返回0，失败返回适当的错误值。

```
::

  int rpmsg_sendto(struct rpmsg_channel *rpdev, void *data, int len, u32 dst);
```

在给定的通道上将消息发送到远程处理器，再发送到调用方提供的目标地址。调用方应指定通道，要发送的数据，其长度（以字节为单位）和显式的目标地址。然后，消息将使用通道的src地址和用户提供的dst地址（因此通道的dst地址将被忽略）发送到通道所属的远程处理器。

如果没有可用的TX缓冲区，则该功能将阻塞直到可用的TX缓冲区可用（即，直到远程处理器消耗了tx缓冲区并将其放回virtio的已使用描述符环上），否则将超时15秒。 发生后者时，将返回                        -ERESTARTSYS。

该功能只能从处理器上下文中调用（目前）。成功返回0，失败返回适当的错误值。



```
::

  int rpmsg_send_offchannel(struct rpmsg_channel *rpdev, u32 src, u32 dst,
							void *data, int len);
```

使用用户提供的src和dst地址将消息发送到远程处理器。调用方应指定通道，要发送的数据，其长度（以字节为单位）以及显式的源地址和目标地址。然后，消息将被发送到该通道所属的远程处理器，但是该通道的src和dst 地址将被忽略（并且将使用用户提供的地址代替）。

如果没有可用的TX缓冲区，则该功能将阻塞直到可用的TX缓冲区可用（即，直到远程处理器消耗了tx缓冲区并将其放回virtio的已使用描述符环上），否则将超时15秒。 发生后者时，将返回                        -ERESTARTSYS。

该功能只能从处理器上下文中调用（目前）。成功返回0，失败返回适当的错误值。

```
::

  int rpmsg_trysend(struct rpmsg_channel *rpdev, void *data, int len);
```

在给定的通道上向远程处理器发送一条消息。 调用方应指定通道，要发送的数据及其长度（以字节为单位）。 该消息将在指定的通道上发送，即其源和目标地址字段将设置为该通道的src和dst地址。

如果没有可用的发送缓冲区，该功能将立即返回**-ENOMEM**，而无需等到一个可用。

该功能只能从处理器上下文中调用（目前）。成功返回0，失败返回适当的错误值。

```
::

  int rpmsg_trysendto(struct rpmsg_channel *rpdev, void *data, int len, u32 dst)
```

在给定的通道上将消息发送到远程处理器，再发送到用户提供的目标地址。用户应指定通道，要发送的数据，其长度（以字节为单位）以及明确的目标地址。然后，将使用该通道的src地址和用户提供的dst地址将该消息发送到该通道所属的远程处理器（因此将忽略该通道的dst地址）。

如果没有可用的发送缓冲区，该功能将立即返回**-ENOMEM**，而无需等到一个可用。

该功能只能从处理器上下文中调用（目前）。成功返回0，失败返回适当的错误值。

```
::

  int rpmsg_trysend_offchannel(struct rpmsg_channel *rpdev, u32 src, u32 dst,
							   void *data, int len);
```

使用用户提供的源地址和目标地址将消息发送到远程处理器。

用户应指定通道，要发送的数据，其长度（以字节为单位）以及显式的源地址和目标地址。然后将消息发送到该通道所属的远程处理器，但发送到该通道的src和dst 地址将被忽略（并且将使用用户提供的地址代替）。

如果没有可用的发送缓冲区，该功能将立即返回**-ENOMEM**，而无需等到一个可用。

该功能只能从处理器上下文中调用（目前）。成功返回0，失败返回适当的错误值。

```
::

  struct rpmsg_endpoint *rpmsg_create_ept(struct rpmsg_channel *rpdev,
		void (*cb)(struct rpmsg_channel *, void *, int, void *, u32),
		void *priv, u32 addr);
```

通过rpmsg_endpoint结构，系统中的每个rpmsg地址都绑定到一个rx回调（因此，当入站消息到达时，它们将由rpmsg总线使用适当的回调处理程序调度）。

此功能允许驱动程序创建这样的endpoint，并由此将回调以及可能存在的私有数据也绑定到rpmsg地址（可以是预先已知的一个，也可以是为其动态分配的）。

简单的rpmsg驱动程序不需要调用rpmsg_create_ept，因为当它们被rpmsg总线探测时已经为它们创建了端点（使用它们在注册rpmsg总线时提供的rx回调）。

所以事情应该只适用于简单的驱动程序：它们已经有一个endpoint，它们的rx回调绑定到其rpmsg地址，并且当相关的入站消息到达时（即，其dst地址等于其rpmsg通道的src地址的消息）， 调用驱动程序的处理程序进行处理。

也就是说，更复杂的驱动程序可能确实需要分配其他rpmsg地址，并将其绑定到不同的rx回调。为了实现这一点，那些驱动程序需要调用这个函数。驱动程序应提供其通道（这样新端点将绑定到其通道所属的同一远程处理器），一个rx回调函数，一个可选的私有数据（在调用rx回调时提供）。以及他们要与回调绑定的地址。 如果addr是RPMSG_ADDR_ANY，则rpmsg_create_ept将为它们动态分配一个可用的rpmsg地址（驱动程序应该有一个很好的理由来解释为什么不在此处始终使用RPMSG_ADDR_ANY）。

成功则返回指向端点的指针，错误则返回NULL。

```
::

  void rpmsg_destroy_ept(struct rpmsg_endpoint *ept);
```

销毁现有的rpmsg endpoint。 用户应提供一个指针到先前使用rpmsg_create_ept()创建的rpmsg端点.

```
::

  int register_rpmsg_driver(struct rpmsg_driver *rpdrv);
```

向rpmsg总线注册rpmsg驱动程序。 用户应提供一个指向rpmsg_driver结构的指针，该结构包含驱动程序的         -> probe()和-> remove()函数，一个rx回调以及一个id_table，该表指定此驱动程序有意使用的通道的名称。

```
::

  void unregister_rpmsg_driver(struct rpmsg_driver *rpdrv);
```

从rpmsg总线注销rpmsg驱动程序。用户应提供指向先前注册的rpmsg_driver结构的指针。 成功返回0，失败返回适当的错误值。

# 典型用法

以下是一个简单的rpmsg驱动程序，它在probe()上发送“ hello！”消息,并且每当收到传入消息时，它将其内容转储到控制台。

```
::

  #include <linux/kernel.h>
  #include <linux/module.h>
  #include <linux/rpmsg.h>

  static void rpmsg_sample_cb(struct rpmsg_channel *rpdev, void *data, int len,
						void *priv, u32 src)
  {
	print_hex_dump(KERN_INFO, "incoming message:", DUMP_PREFIX_NONE,
						16, 1, data, len, true);
  }

  static int rpmsg_sample_probe(struct rpmsg_channel *rpdev)
  {
	int err;

	dev_info(&rpdev->dev, "chnl: 0x%x -> 0x%x\n", rpdev->src, rpdev->dst);

	/* send a message on our channel */
	err = rpmsg_send(rpdev, "hello!", 6);
	if (err) {
		pr_err("rpmsg_send failed: %d\n", err);
		return err;
	}

	return 0;
  }

  static void rpmsg_sample_remove(struct rpmsg_channel *rpdev)
  {
	dev_info(&rpdev->dev, "rpmsg sample client driver is removed\n");
  }

  static struct rpmsg_device_id rpmsg_driver_sample_id_table[] = {
	{ .name	= "rpmsg-client-sample" },
	{ },
  };
  MODULE_DEVICE_TABLE(rpmsg, rpmsg_driver_sample_id_table);

  static struct rpmsg_driver rpmsg_sample_client = {
	.drv.name	= KBUILD_MODNAME,
	.id_table	= rpmsg_driver_sample_id_table,
	.probe		= rpmsg_sample_probe,
	.callback	= rpmsg_sample_cb,
	.remove		= rpmsg_sample_remove,
  };
  module_rpmsg_driver(rpmsg_sample_client);
```

# rpmsg通道的分配

目前，我们仅支持rpmsg通道的动态分配.

只有设置了VIRTIO_RPMSG_F_NS virtio设备功能的远程处理器才有可能启动此功能,也意味着远程处理器支持动态名称公告消息服务。

启用此功能后，rpmsg设备（即通道）的创建是完全动态的：远程处理器通过发送名称服务消息（包含远程服务的名称和rpmsg addr，请参阅结构体**rpmsg_ns_msg**）来宣布远程rpmsg服务的存在。

然后，此消息由rpmsg总线处理，该总线继而动态创建并注册rpmsg通道（代表远程服务）。 如果/当注册了相关的rpmsg驱动程序时，总线将立即对其进行探测，然后可以开始向远程服务发送消息了。







