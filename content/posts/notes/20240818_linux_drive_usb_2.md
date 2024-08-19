---
title: "USB 知识笔记（二）"
date: 2024-08-18T01:31:42+08:00
lastmod: 2024-08-18T01:31:42+08:00
author: ["hacper"]
tags:
    - Linux
    - USB
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: true # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true

---

## linux usb 驱动代码

### linux/drivers/usb 的代码结构和作用
在Linux内核源代码中，drivers/usb/ 目录包含与USB子系统相关的驱动程序和核心代码。

### 1. `drivers/usb/core/`
- **作用**：包含USB子系统的核心代码，负责USB设备的发现、配置和管理。
- **主要文件和功能**：
  - `usb.c`：USB核心初始化和管理功能。
  - `hub.c`：处理USB集线器（Hub）的功能。
  - `hcd.c`：主机控制器驱动的通用代码。
  - `urb.c`：处理USB请求块（URB）相关的功能。

### 2. `drivers/usb/host/`
- **作用**：包含USB主机控制器驱动程序（HCD），负责与USB硬件控制器进行通信。
- **主要文件和功能**：
  - `ehci-hcd.c`：EHCI（增强型主机控制器接口）驱动。
  - `ohci-hcd.c`：OHCI（开放主机控制器接口）驱动。
  - `uhci-hcd.c`：UHCI（通用主机控制器接口）驱动。
  - `xhci-hcd.c`：xHCI（扩展主机控制器接口）驱动。

### 3. `drivers/usb/storage/`
- **作用**：包含USB大容量存储设备的驱动程序，例如USB闪存驱动和外部硬盘驱动。
- **主要文件和功能**：
  - `usb-storage.c`：通用USB存储设备驱动。
  - `uas.c`：USB附加存储（UAS）驱动，支持更高效的传输协议。

### 4. `drivers/usb/gadget/`
- **作用**：包含USB设备模式（Gadget）的驱动程序，主要用于嵌入式系统，使设备能够作为USB外设连接到主机。
- **主要文件和功能**：
  - `udc/`：USB设备控制器驱动程序。这个驱动是针对具体 CPU 平台的，如果找不到现成的，就要自己实现
  - `composite.c`：复合USB设备驱动框架。
  - `function/`：包含具体功能驱动，如USB网络、串口和存储功能。

### 5. `drivers/usb/serial/`
- **作用**：包含USB串行设备的驱动程序，例如USB到串口转换器。
- **主要文件和功能**：
  - `usb-serial.c`：通用USB串行驱动框架。
  - `pl2303.c`：Prolific PL2303 USB串行转换器驱动。
  - `ftdi_sio.c`：FTDI USB串行转换器驱动。

### 6. `drivers/usb/misc/`
- **作用**：包含其他不属于上述分类的USB设备驱动程序。
- **主要文件和功能**：
  - `usbtest.c`：USB测试驱动，用于测试和验证USB子系统。
  - `legousbtower.c`：LEGO USB塔驱动。

### 7. `drivers/usb/mon/`
- **作用**：包含USB监控（usbmon）驱动程序，用于捕获和监控USB流量。
- **主要文件和功能**：
  - `mon.c`：USB监控驱动核心代码。

### 8. `drivers/usb/typec/`
- **作用**：包含USB Type-C和USB Power Delivery（USB PD）相关的驱动程序。
- **主要文件和功能**：
  - `typec.c`：USB Type-C核心代码。
  - `tcpm/`：Type-C端口管理器相关驱动。

| **目录**               | **作用**                                                              | **主要文件和功能**                                                                                  |
|------------------------|-----------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------|
| `drivers/usb/core/`    | USB子系统的核心代码，设备发现、配置和管理                               | `usb.c`、`hub.c`、`hcd.c`、`urb.c`                                                                  |
| `drivers/usb/host/`    | USB主机控制器驱动                                                     | `ehci-hcd.c`、`ohci-hcd.c`、`uhci-hcd.c`、`xhci-hcd.c`                                              |
| `drivers/usb/storage/` | USB大容量存储设备驱动                                                 | `usb-storage.c`、`uas.c`                                                                            |
| `drivers/usb/gadget/`  | USB设备模式驱动，嵌入式系统使用                                       | `udc/`、`composite.c`、`function/`                                                                  |
| `drivers/usb/serial/`  | USB串行设备驱动                                                       | `usb-serial.c`、`pl2303.c`、`ftdi_sio.c`                                                            |
| `drivers/usb/misc/`    | 其他USB设备驱动                                                       | `usbtest.c`、`legousbtower.c`                                                                       |
| `drivers/usb/mon/`     | USB监控驱动，用于捕获和监控USB流量                                     | `mon.c`                                                                                             |
| `drivers/usb/typec/`   | USB Type-C和USB PD相关驱动                                            | `typec.c`、`tcpm/`                                                                                  |


### usb core 的作用

USB Core 负责实现一些核心的功能，为别的设备驱动程序提供服务，提供一个用于访问和控制 USB 硬件的接口，而不用去考虑系统当前存在哪种主机控制器。

内核中 USB 子系统的结构
![](https://github.com/hacperme/picx-images-hosting/raw/master/20240718/image.3k7wgstrqt.webp)

### usb core 代码流程

linux/drivers/usb/core/usb.c

```c
subsys_initcall(usb_init);
module_exit(usb_exit);
MODULE_LICENSE("GPL");
```



```c
static int __init usb_init(void)
{
	int retval;

	// 检查是否禁用USB支持
	if (usb_disabled()) {
		pr_info("%s: USB support disabled\n", usbcore_name);
		return 0;
	}

	// 初始化内存池最大值
	usb_init_pool_max();

	// 初始化USB调试文件系统
	usb_debugfs_init();

	// 注册ACPI支持
	usb_acpi_register();

	// 注册USB总线类型
	retval = bus_register(&usb_bus_type);
	if (retval)
		goto bus_register_failed;

	// 注册总线通知器
	retval = bus_register_notifier(&usb_bus_type, &usb_bus_nb);
	if (retval)
		goto bus_notifier_failed;

	// 初始化USB主设备号
	retval = usb_major_init();
	if (retval)
		goto major_init_failed;

	// 注册usbfs驱动程序
	retval = usb_register(&usbfs_driver);
	if (retval)
		goto driver_register_failed;

	// 初始化USB设备I/O
	retval = usb_devio_init();
	if (retval)
		goto usb_devio_init_failed;

	// 初始化USB集线器
	retval = usb_hub_init();
	if (retval)
		goto hub_init_failed;

	// 注册USB通用设备驱动程序
	retval = usb_register_device_driver(&usb_generic_driver, THIS_MODULE);
	if (!retval)
		goto out;

	// 错误处理和清理步骤
	usb_hub_cleanup();
hub_init_failed:
	usb_devio_cleanup();
usb_devio_init_failed:
	usb_deregister(&usbfs_driver);
driver_register_failed:
	usb_major_cleanup();
major_init_failed:
	bus_unregister_notifier(&usb_bus_type, &usb_bus_nb);
bus_notifier_failed:
	bus_unregister(&usb_bus_type);
bus_register_failed:
	usb_acpi_unregister();
	usb_debugfs_cleanup();
out:
	return retval;
}

/*
 * Cleanup
 */
static void __exit usb_exit(void)
{
	/* This will matter if shutdown/reboot does exitcalls. */
	if (usb_disabled())
		return;

	usb_release_quirk_list();
	usb_deregister_device_driver(&usb_generic_driver);
	usb_major_cleanup();
	usb_deregister(&usbfs_driver);
	usb_devio_cleanup();
	usb_hub_cleanup();
	bus_unregister_notifier(&usb_bus_type, &usb_bus_nb);
	bus_unregister(&usb_bus_type);
	usb_acpi_unregister();
	usb_debugfs_cleanup();
	idr_destroy(&usb_bus_idr);
}
```

__init 标记， 表明这个函数仅在初始化期间使用，在模块被装载之后，它占用的资源就会释放掉用于它处

 __init 的 定 义 在
include/linux/init.h 文件中

```c
/* These macros are used to mark some functions or 
 * initialized data (doesn't apply to uninitialized data)
 * as `initialization' functions. The kernel can take this
 * as hint that the function is used only during the initialization
 * phase and free up used memory resources after
 *
 * Usage:
 * For functions:
 * 
 * You should add __init immediately before the function name, like:
 *
 * static void __init initme(int x, int y)
 * {
 *    extern int z; z = x * y;
 * }
 *
 * If the function has a prototype somewhere, you can also add
 * __init between closing brace of the prototype and semicolon:
 *
 * extern int initialize_foobar_device(int, int, int) __init;
 *
 * For initialized data:
 * You should insert __initdata or __initconst between the variable name
 * and equal sign followed by value, e.g.:
 *
 * static int init_variable __initdata = 0;
 * static const char linux_logo[] __initconst = { 0x32, 0x36, ... };
 *
 * Don't forget to initialize data not at file scope, i.e. within a function,
 * as gcc otherwise puts the data into the bss section and not into the init
 * section.
 */

/* These are for everybody (although not all archs will actually
   discard it in modules) */
#define __init		__section(.init.text) __cold  __latent_entropy __noinitretpoline __nocfi
```

通常，编译器将函数放在.text 节，变量放在.data 节或.bss 节，使用 section 属性，可以让编译器将函数或变量放在指定的节中。那么前面对__init 的定义便表示将它修饰的代码放在.init.text 节。

连接器可以把相同节的代码或数据安排在一起，比如__init 修饰的所有代码都会被放在.init.text 节里，初始化结束后就可以释放这部分内存。

```c
#define ___define_initcall(fn, id, __sec) \
	static initcall_t __initcall_##fn##id __used \
		__attribute__((__section__(#__sec ".init"))) = fn;

#define __define_initcall(fn, id) ___define_initcall(fn, id, .initcall##id)

#define subsys_initcall(fn)		__define_initcall(fn, 4)
```

__define_initcall，它用于将指定的函数指针 fn 放到 initcall.init 节里，而对于具体的 subsys_initcall 宏，则是把 fn 放到.initcall.init 的子节.initcall4.init 里。

各个子节的顺序是确定的，即先调用.initcall1.init 中的函数指针，再调用.initcall2.init 中的函数指针等。__init 修饰的初始化函数在内核初始化过程中调用的顺序和.initcall.init 节里函数指针的顺序有关，不同的初始化函数被放在不同的子节中，因此，也就决定了它们的调用顺序。

实际执行函数调用的地方在/init/main.c do_initcalls



这个函数 `usb_init` 是Linux内核USB子系统初始化的一部分。它使用了一系列步骤来初始化USB子系统，包括注册总线、设备驱动、创建调试文件系统、处理ACPI（高级配置和电源接口）等。函数标记为 `__init`，表明它是在内核初始化时调用的，并且在初始化完成后可以被丢弃以节省内存。

以下是 `usb_init` 函数的详细解释：

1. **检查是否禁用USB支持**：
   ```c
   if (usb_disabled()) {
       pr_info("%s: USB support disabled\n", usbcore_name);
       return 0;
   }
   ```
   如果USB支持被禁用，则打印信息并返回。

2. **初始化内存池最大值**：
   ```c
   usb_init_pool_max();
   ```
   初始化内存池的最大值，用于管理USB内存分配。

3. **初始化USB调试文件系统**：
   ```c
   usb_debugfs_init();
   ```
   创建调试文件系统的接口，用于调试和开发。

4. **注册ACPI支持**：
   ```c
   usb_acpi_register();
   ```
   注册ACPI支持，用于电源管理和设备配置。

5. **注册USB总线类型**：
   ```c
   retval = bus_register(&usb_bus_type);
   if (retval)
       goto bus_register_failed;
   ```
   注册USB总线类型，使内核能够识别和管理USB设备。

6. **注册总线通知器**：
   ```c
   retval = bus_register_notifier(&usb_bus_type, &usb_bus_nb);
   if (retval)
       goto bus_notifier_failed;
   ```
   注册总线通知器，用于监控USB总线上设备的连接和断开事件。

7. **初始化USB主设备号**：
   ```c
   retval = usb_major_init();
   if (retval)
       goto major_init_failed;
   ```
   初始化USB主设备号，分配主设备号用于设备文件系统中的USB设备。

8. **注册usbfs驱动程序**：
   ```c
   retval = usb_register(&usbfs_driver);
   if (retval)
       goto driver_register_failed;
   ```
   注册usbfs驱动程序，提供用户空间访问USB设备的接口。

9. **初始化USB设备I/O**：
   ```c
   retval = usb_devio_init();
   if (retval)
       goto usb_devio_init_failed;
   ```
   初始化USB设备的I/O操作，提供设备通信的基础设施。

10. **初始化USB集线器**：
    ```c
    retval = usb_hub_init();
    if (retval)
        goto hub_init_failed;
    ```
    初始化USB集线器，管理USB设备的拓扑结构和连接。

11. **注册USB通用设备驱动程序**：
    ```c
    retval = usb_register_device_driver(&usb_generic_driver, THIS_MODULE);
    if (!retval)
        goto out;
    ```

12. **错误处理和清理步骤**：
    如果在上述任何一步发生错误，函数会跳转到相应的标签进行清理操作，以确保之前的初始化步骤不会影响系统的稳定性。



`usb_exit` 函数用于在模块卸载时进行清理工作。它被标记为 `__exit`，意味着该函数只会在模块卸载时调用。函数确保在卸载USB子系统时，所有的资源都能被正确释放，防止内存泄漏和其他潜在问题。

### linux 设备模型

Linux 的设备模型里面包含三个主角，总线、设备、驱动，也就是 bus、device、driver。



```c
/**
 * struct bus_type - The bus type of the device
 *
 * @name:	The name of the bus.
 * @dev_name:	Used for subsystems to enumerate devices like ("foo%u", dev->id).
 * @bus_groups:	Default attributes of the bus.
 * @dev_groups:	Default attributes of the devices on the bus.
 * @drv_groups: Default attributes of the device drivers on the bus.
 * @match:	Called, perhaps multiple times, whenever a new device or driver
 *		is added for this bus. It should return a positive value if the
 *		given device can be handled by the given driver and zero
 *		otherwise. It may also return error code if determining that
 *		the driver supports the device is not possible. In case of
 *		-EPROBE_DEFER it will queue the device for deferred probing.
 * @uevent:	Called when a device is added, removed, or a few other things
 *		that generate uevents to add the environment variables.
 * @probe:	Called when a new device or driver add to this bus, and callback
 *		the specific driver's probe to initial the matched device.
 * @sync_state:	Called to sync device state to software state after all the
 *		state tracking consumers linked to this device (present at
 *		the time of late_initcall) have successfully bound to a
 *		driver. If the device has no consumers, this function will
 *		be called at late_initcall_sync level. If the device has
 *		consumers that are never bound to a driver, this function
 *		will never get called until they do.
 * @remove:	Called when a device removed from this bus.
 * @shutdown:	Called at shut-down time to quiesce the device.
 *
 * @online:	Called to put the device back online (after offlining it).
 * @offline:	Called to put the device offline for hot-removal. May fail.
 *
 * @suspend:	Called when a device on this bus wants to go to sleep mode.
 * @resume:	Called to bring a device on this bus out of sleep mode.
 * @num_vf:	Called to find out how many virtual functions a device on this
 *		bus supports.
 * @dma_configure:	Called to setup DMA configuration on a device on
 *			this bus.
 * @dma_cleanup:	Called to cleanup DMA configuration on a device on
 *			this bus.
 * @pm:		Power management operations of this bus, callback the specific
 *		device driver's pm-ops.
 * @need_parent_lock:	When probing or removing a device on this bus, the
 *			device core should lock the device's parent.
 *
 * A bus is a channel between the processor and one or more devices. For the
 * purposes of the device model, all devices are connected via a bus, even if
 * it is an internal, virtual, "platform" bus. Buses can plug into each other.
 * A USB controller is usually a PCI device, for example. The device model
 * represents the actual connections between buses and the devices they control.
 * A bus is represented by the bus_type structure. It contains the name, the
 * default attributes, the bus' methods, PM operations, and the driver core's
 * private data.
 */
struct bus_type {
	const char		*name;
	const char		*dev_name;
	const struct attribute_group **bus_groups;
	const struct attribute_group **dev_groups;
	const struct attribute_group **drv_groups;

	int (*match)(struct device *dev, struct device_driver *drv);
	int (*uevent)(const struct device *dev, struct kobj_uevent_env *env);
	int (*probe)(struct device *dev);
	void (*sync_state)(struct device *dev);
	void (*remove)(struct device *dev);
	void (*shutdown)(struct device *dev);

	int (*online)(struct device *dev);
	int (*offline)(struct device *dev);

	int (*suspend)(struct device *dev, pm_message_t state);
	int (*resume)(struct device *dev);

	int (*num_vf)(struct device *dev);

	int (*dma_configure)(struct device *dev);
	void (*dma_cleanup)(struct device *dev);

	const struct dev_pm_ops *pm;

	bool need_parent_lock;
};

/**
 * struct device_driver - The basic device driver structure
 * @name:	Name of the device driver.
 * @bus:	The bus which the device of this driver belongs to.
 * @owner:	The module owner.
 * @mod_name:	Used for built-in modules.
 * @suppress_bind_attrs: Disables bind/unbind via sysfs.
 * @probe_type:	Type of the probe (synchronous or asynchronous) to use.
 * @of_match_table: The open firmware table.
 * @acpi_match_table: The ACPI match table.
 * @probe:	Called to query the existence of a specific device,
 *		whether this driver can work with it, and bind the driver
 *		to a specific device.
 * @sync_state:	Called to sync device state to software state after all the
 *		state tracking consumers linked to this device (present at
 *		the time of late_initcall) have successfully bound to a
 *		driver. If the device has no consumers, this function will
 *		be called at late_initcall_sync level. If the device has
 *		consumers that are never bound to a driver, this function
 *		will never get called until they do.
 * @remove:	Called when the device is removed from the system to
 *		unbind a device from this driver.
 * @shutdown:	Called at shut-down time to quiesce the device.
 * @suspend:	Called to put the device to sleep mode. Usually to a
 *		low power state.
 * @resume:	Called to bring a device from sleep mode.
 * @groups:	Default attributes that get created by the driver core
 *		automatically.
 * @dev_groups:	Additional attributes attached to device instance once
 *		it is bound to the driver.
 * @pm:		Power management operations of the device which matched
 *		this driver.
 * @coredump:	Called when sysfs entry is written to. The device driver
 *		is expected to call the dev_coredump API resulting in a
 *		uevent.
 * @p:		Driver core's private data, no one other than the driver
 *		core can touch this.
 *
 * The device driver-model tracks all of the drivers known to the system.
 * The main reason for this tracking is to enable the driver core to match
 * up drivers with new devices. Once drivers are known objects within the
 * system, however, a number of other things become possible. Device drivers
 * can export information and configuration variables that are independent
 * of any specific device.
 */
struct device_driver {
	const char		*name;
	const struct bus_type	*bus;

	struct module		*owner;
	const char		*mod_name;	/* used for built-in modules */

	bool suppress_bind_attrs;	/* disables bind/unbind via sysfs */
	enum probe_type probe_type;

	const struct of_device_id	*of_match_table;
	const struct acpi_device_id	*acpi_match_table;

	int (*probe) (struct device *dev);
	void (*sync_state)(struct device *dev);
	int (*remove) (struct device *dev);
	void (*shutdown) (struct device *dev);
	int (*suspend) (struct device *dev, pm_message_t state);
	int (*resume) (struct device *dev);
	const struct attribute_group **groups;
	const struct attribute_group **dev_groups;

	const struct dev_pm_ops *pm;
	void (*coredump) (struct device *dev);

	struct driver_private *p;
};

/*
 * The type of device, "struct device" is embedded in. A class
 * or bus can contain devices of different types
 * like "partitions" and "disks", "mouse" and "event".
 * This identifies the device type and carries type-specific
 * information, equivalent to the kobj_type of a kobject.
 * If "name" is specified, the uevent will contain it in
 * the DEVTYPE variable.
 */
struct device_type {
	const char *name;
	const struct attribute_group **groups;
	int (*uevent)(const struct device *dev, struct kobj_uevent_env *env);
	char *(*devnode)(const struct device *dev, umode_t *mode,
			 kuid_t *uid, kgid_t *gid);
	void (*release)(struct device *dev);

	const struct dev_pm_ops *pm;
};
```



`struct bus_type` 是 Linux 内核中用于定义总线类型的结构体。它提供了总线类型的基本属性和操作函数指针，使内核能够统一管理和操作不同类型的总线。总线类型在设备模型中起到桥梁作用，将设备与驱动程序连接起来。以下是 `struct bus_type` 结构体的详细解释：

1. **基本属性**
   - `const char *name`：总线的名称。
   - `const char *dev_name`：总线设备的名称前缀。

2. **属性组**
   - `const struct attribute_group **bus_groups`：总线的属性组。
   - `const struct attribute_group **dev_groups`：总线上的设备的属性组。
   - `const struct attribute_group **drv_groups`：总线上驱动程序的属性组。

3. **操作函数指针**
   - `int (*match)(struct device *dev, struct device_driver *drv)`：匹配函数，用于确定设备是否与驱动程序匹配。
   - `int (*uevent)(const struct device *dev, struct kobj_uevent_env *env)`：处理设备事件（如添加、移除）的函数。
   - `int (*probe)(struct device *dev)`：探测函数，在设备与驱动程序匹配时调用。
   - `void (*sync_state)(struct device *dev)`：同步设备状态的函数。
   - `void (*remove)(struct device *dev)`：移除设备时调用的函数。
   - `void (*shutdown)(struct device *dev)`：关机时调用的函数。

4. **电源管理**
   - `int (*online)(struct device *dev)`：使设备上线的函数。
   - `int (*offline)(struct device *dev)`：使设备下线的函数。
   - `int (*suspend)(struct device *dev, pm_message_t state)`：挂起设备时调用的函数。
   - `int (*resume)(struct device *dev)`：恢复设备时调用的函数。
   - `const struct dev_pm_ops *pm`：电源管理操作函数指针集合。

5. **其他操作**
   - `int (*num_vf)(struct device *dev)`：获取设备的虚拟功能数量。
   - `int (*dma_configure)(struct device *dev)`：配置DMA的函数。
   - `void (*dma_cleanup)(struct device *dev)`：清理DMA配置的函数。
   - `bool need_parent_lock`：指示在操作设备时是否需要父设备的锁。



`struct bus_type` 提供了定义和操作总线类型的机制，使内核能够统一管理不同类型的总线。通过这个结构体，内核可以：

1. **注册总线类型**：使用 `bus_register` 函数注册总线类型。
2. **匹配设备和驱动程序**：通过 `match` 函数确定设备是否与驱动程序匹配。
3. **处理设备事件**：通过 `uevent` 函数处理设备添加、移除等事件。
4. **探测和移除设备**：通过 `probe` 和 `remove` 函数在设备与驱动程序匹配时进行初始化和清理。
5. **管理电源状态**：通过 `suspend`、`resume` 等函数管理设备的电源状态。
6. **配置DMA**：通过 `dma_configure` 和 `dma_cleanup` 函数配置和清理DMA资源。





`struct device_type` 是 Linux 内核中用于定义设备类型的结构体。它提供了一种机制来区分同一总线或类下的不同设备类型，并允许设备类型携带特定的信息和操作函数。

1. **基本属性**
   - `const char *name`：设备类型的名称。如果指定了 `name`，则在设备事件（uevent）中会包含在 `DEVTYPE` 变量中。

2. **属性组**
   - `const struct attribute_group **groups`：设备类型的属性组。用于定义和管理与该设备类型相关的属性。

3. **操作函数指针**
   - `int (*uevent)(const struct device *dev, struct kobj_uevent_env *env)`：处理设备事件（如添加、移除）的函数。用于生成设备事件。
   - `char *(*devnode)(const struct device *dev, umode_t *mode, kuid_t *uid, kgid_t *gid)`：生成设备节点名称的函数。用于设置设备文件的名称和权限。
   - `void (*release)(struct device *dev)`：释放设备时调用的函数。用于清理设备资源。

4. **电源管理**
   - `const struct dev_pm_ops *pm`：电源管理操作函数指针集合。用于管理设备的电源状态。



`struct device_type` 提供了一种机制来定义和管理不同类型的设备，使内核能够更灵活地处理同一总线或类下的多种设备类型。通过这个结构体，内核可以：

1. **区分设备类型**：通过 `name` 字段和 `uevent` 函数区分不同类型的设备，并在设备事件中包含设备类型信息。
2. **管理设备属性**：通过 `groups` 字段定义和管理设备类型的属性组。
3. **生成设备节点**：通过 `devnode` 函数生成设备文件的名称和权限。
4. **释放设备资源**：通过 `release` 函数在设备释放时进行清理操作。
5. **管理设备电源**：通过 `pm` 字段管理设备的电源状态。



`struct device_driver` 是 Linux 内核中用于定义设备驱动程序的基本结构体。它提供了驱动程序的基本属性和操作函数指针，使内核能够统一管理和操作不同的设备驱动程序。



1. **基本属性**
   - `const char *name`：设备驱动程序的名称。
   - `const struct bus_type *bus`：驱动程序所属的总线类型。

2. **模块信息**
   - `struct module *owner`：模块的所有者。
   - `const char *mod_name`：内建模块使用的名称。

3. **sysfs绑定属性**
   - `bool suppress_bind_attrs`：是否通过 sysfs 禁用绑定/解绑操作。

4. **探测类型**
   - `enum probe_type probe_type`：探测类型（同步或异步）。

5. **匹配表**
   - `const struct of_device_id *of_match_table`：设备树匹配表。
   - `const struct acpi_device_id *acpi_match_table`：ACPI匹配表。

6. **操作函数指针**
   - `int (*probe) (struct device *dev)`：探测函数，用于查询特定设备是否存在，是否能与驱动程序工作，并将驱动程序绑定到设备。
   - `void (*sync_state)(struct device *dev)`：在所有与此设备相关的状态跟踪消费者成功绑定到驱动程序后调用，同步设备状态到软件状态。
   - `int (*remove) (struct device *dev)`：当设备从系统中移除时调用，解除设备与驱动程序的绑定。
   - `void (*shutdown) (struct device *dev)`：系统关闭时调用，使设备进入安静状态。
   - `int (*suspend) (struct device *dev, pm_message_t state)`：将设备置于睡眠模式时调用，通常进入低功耗状态。
   - `int (*resume) (struct device *dev)`：将设备从睡眠模式恢复时调用。
   - `const struct attribute_group **groups`：驱动程序核心自动创建的默认属性组。
   - `const struct attribute_group **dev_groups`：设备绑定到驱动程序后附加的额外属性组。

7. **电源管理**
   - `const struct dev_pm_ops *pm`：匹配此驱动程序的设备的电源管理操作。

8. **内核转储**
   - `void (*coredump) (struct device *dev)`：当 sysfs 条目被写入时调用，设备驱动程序应调用 `dev_coredump` API 生成 uevent。

9. **私有数据**
   - `struct driver_private *p`：驱动程序核心的私有数据，只有驱动程序核心可以访问。



`struct device_driver` 提供了定义和操作设备驱动程序的机制，使内核能够统一管理不同类型的设备驱动程序。通过这个结构体，内核可以：

1. **注册驱动程序**：使用 `driver_register` 函数注册驱动程序。
2. **匹配设备和驱动程序**：通过 `probe` 函数查询设备是否存在并能与驱动程序工作，并将驱动程序绑定到设备。
3. **处理设备移除**：通过 `remove` 函数在设备移除时进行清理操作。
4. **管理电源状态**：通过 `suspend`、`resume` 等函数管理设备的电源状态。
5. **生成设备事件**：通过 `uevent` 函数处理设备事件，并在设备事件中包含设备类型信息。
6. **处理内核转储**：通过 `coredump` 函数处理内核转储请求。



### linux 内核中总线是如何发现设备的？设备又是如何和驱动对应起来的?

在 Linux 内核中，总线发现设备以及设备与驱动程序的匹配是通过一套标准的机制实现的。以下是这个过程的详细解释：

### 总线发现设备

1. **总线类型（Bus Type）**：每种总线类型（如 PCI、USB、I2C 等）都有一个 `struct bus_type` 结构体，用于定义总线的操作和属性。

2. **设备注册（Device Registration）**：当一个设备连接到系统时，驱动程序会调用适当的函数来注册该设备。这个过程通常涉及创建一个 `struct device` 结构体，并将其添加到总线的设备列表中。例如，USB 设备的注册是通过 USB 子系统完成的。

3. **总线扫描（Bus Scanning）**：某些总线（如 PCI 总线）会在启动时或热插拔时扫描总线以发现新设备。这些设备信息会被添加到内核的设备列表中。

### 设备与驱动的匹配

1. **设备与驱动描述符**：每个设备和驱动都有一些描述符，用于标识它们的类型和功能。设备的描述符通常存储在设备的 `struct device` 结构体中，而驱动的描述符则存储在 `struct device_driver` 结构体中。

2. **匹配函数（Match Function）**：每个总线类型都有一个匹配函数，用于比较设备和驱动的描述符。如果匹配成功，则认为该驱动可以处理这个设备。匹配函数通常是 `struct bus_type` 结构体中的 `match` 函数指针。

3. **设备绑定驱动**：如果匹配成功，内核会调用驱动的 `probe` 函数来初始化设备，并将设备绑定到该驱动。`probe` 函数是 `struct device_driver` 结构体中的一个函数指针。

### 具体流程

以下是一个更详细的示例，说明设备与驱动的匹配流程：

1. **定义总线类型**：
    
    ```c
    struct bus_type my_bus_type = {
        .name = "my_bus",
        .match = my_bus_match, // 匹配函数
        // 其他字段...
};
    
    int my_bus_match(struct device *dev, struct device_driver *drv)
    {
        // 比较设备和驱动的描述符，返回匹配结果
        return !strcmp(dev->name, drv->name);
    }
```
    
2. **注册总线**：
    ```c
    int __init my_bus_init(void)
    {
        return bus_register(&my_bus_type);
    }
    ```

3. **注册设备**：
    ```c
    struct device my_device = {
        .name = "my_device",
        .bus = &my_bus_type,
        // 其他字段...
    };

    int __init my_device_init(void)
    {
        return device_register(&my_device);
    }
    ```

4. **注册驱动**：
    ```c
    struct device_driver my_driver = {
        .name = "my_device",
        .bus = &my_bus_type,
        .probe = my_device_probe, // 驱动的 probe 函数
        // 其他字段...
    };

    int __init my_driver_init(void)
    {
        return driver_register(&my_driver);
    }

    int my_device_probe(struct device *dev)
    {
        // 初始化设备
        printk("Device %s is probed\n", dev->name);
        return 0;
    }
    ```

5. **设备绑定驱动**：
    当设备和驱动都注册后，内核会调用 `my_bus_match` 函数进行匹配。如果匹配成功，会调用 `my_device_probe` 函数进行设备初始化。



usb bus的定义在 usb/core/drive.c 

```c
const struct bus_type usb_bus_type = {
	.name =		"usb",
	.match =	usb_device_match,
	.uevent =	usb_uevent,
	.need_parent_lock =	true,
};
```



### usb 接口

设备可以有多个接口，每个接口代表一个功能，每个接口对应着一个驱动。Linux 设备模型中的 device 落实在 USB 子系统，成了两个结构：一个是 struct usb_device，一个是 structusb_interface。一个 USB 键盘，上面带一个扬声器，因此有两个接口，那肯定得要两个驱动程序：一个是键盘驱动程序，一个是音频流驱动程序

```c
struct usb_interface {
	/* 该接口的备用设置数组，无特定顺序 */
	struct usb_host_interface *altsetting;

	/* 当前活动的备用设置 */
	struct usb_host_interface *cur_altsetting;
	unsigned num_altsetting;	/* 备用设置的数量 */

	/* 如果有接口关联描述符，则列出相关联的接口 */
	struct usb_interface_assoc_descriptor *intf_assoc;

	int minor;			/* 该接口绑定的次设备号 */
	enum usb_interface_condition condition;	/* 绑定状态 */
	unsigned sysfs_files_created:1;	/* sysfs 属性已创建 */
	unsigned ep_devs_created:1;	/* 端点 "设备" 已创建 */
	unsigned unregistering:1;	/* 正在进行注销 */
	unsigned needs_remote_wakeup:1;	/* 驱动程序需要远程唤醒 */
	unsigned needs_altsetting0:1;	/* 待切换到备用设置 0 */
	unsigned needs_binding:1;	/* 需要延迟解绑/重新绑定 */
	unsigned resetting_device:1;	/* true：复位后进行带宽分配 */
	unsigned authorized:1;		/* 用于接口授权 */
	enum usb_wireless_status wireless_status;
	struct work_struct wireless_status_work;

	struct device dev;		/* 接口特定的设备信息 */
	struct device *usb_dev;
	struct work_struct reset_ws;	/* 在原子上下文中的复位操作 */
};
```

`struct usb_interface` 是 Linux 内核中用于表示 USB 接口的结构体。一个 USB 设备可以有多个接口，每个接口可以有多个备用设置（alternate settings），并且每个接口可以绑定到一个特定的驱动程序。以下是 `struct usb_interface` 结构体的详细解释：



1. **备用设置（Alternate Settings）**
   - `struct usb_host_interface *altsetting`：该接口的备用设置数组。备用设置允许接口在不同配置之间切换，以支持不同的操作模式。
   - `struct usb_host_interface *cur_altsetting`：当前活动的备用设置。
   - `unsigned num_altsetting`：备用设置的数量。

2. **接口关联描述符（Interface Association Descriptor）**
   - `struct usb_interface_assoc_descriptor *intf_assoc`：如果存在接口关联描述符，则它列出相关联的接口。用于描述属于同一功能的多个接口。

3. **接口状态**
   - `int minor`：该接口绑定的次设备号。
   - `enum usb_interface_condition condition`：接口绑定的状态。

4. **标志字段**
   - `unsigned sysfs_files_created:1`：sysfs 属性是否已创建。
   - `unsigned ep_devs_created:1`：端点 "设备" 是否已创建。
   - `unsigned unregistering:1`：是否正在进行注销。
   - `unsigned needs_remote_wakeup:1`：驱动程序是否需要远程唤醒。
   - `unsigned needs_altsetting0:1`：是否待切换到备用设置 0。
   - `unsigned needs_binding:1`：是否需要延迟解绑/重新绑定。
   - `unsigned resetting_device:1`：复位后是否进行带宽分配。
   - `unsigned authorized:1`：用于接口授权。

5. **无线状态**
   - `enum usb_wireless_status wireless_status`：无线状态。
   - `struct work_struct wireless_status_work`：处理无线状态的工作结构体。

6. **设备信息**
   - `struct device dev`：接口特定的设备信息。每个接口在设备模型中表示为一个设备。
   - `struct device *usb_dev`：指向父 USB 设备的指针。

7. **复位操作**
   - `struct work_struct reset_ws`：在原子上下文中的复位操作。



`struct usb_interface` 结构体提供了管理 USB 接口的基础设施。它包含了接口的备用设置、接口关联信息、接口状态、设备信息等。通过这些字段，内核能够有效地管理和操作 USB 接口，包括处理设备绑定、解绑、复位和状态变更等操作。

在实际使用中，当一个 USB 设备被插入时，内核会通过 USB 子系统创建并初始化 `usb_interface` 结构体，并通过匹配和绑定机制将其与适当的驱动程序关联。

#### 接口的设置

```c
struct usb_host_interface {
	struct usb_interface_descriptor	desc;

	int extralen;
	unsigned char *extra;   /* Extra descriptors */

	/* array of desc.bNumEndpoints endpoints associated with this
	 * interface setting.  these will be in no particular order.
	 */
	struct usb_host_endpoint *endpoint;

	char *string;		/* iInterface string, if present */
};
```



`struct usb_host_interface` 是 Linux 内核中用于表示 USB 接口的特定设置（alternate setting）的结构体。每个 USB 接口可以有多个设置，每个设置对应一组特定的端点配置。以下是 `struct usb_host_interface` 结构体的详细解释：

1. **接口描述符（Interface Descriptor）**
   - `struct usb_interface_descriptor desc`：表示该设置的 USB 接口描述符。描述符包含有关接口的各种信息，如端点数量、接口类和子类等。

2. **额外描述符（Extra Descriptors）**
   - `int extralen`：额外描述符的长度。
   - `unsigned char *extra`：指向额外描述符的指针。这些描述符可能包含特定于设备的信息，这些信息不在标准的接口描述符中。

3. **端点数组（Endpoint Array）**
   - `struct usb_host_endpoint *endpoint`：一个包含与此接口设置相关的端点的数组。端点是 USB 通信的基本单元，每个端点都有特定的地址和属性。
     - `desc.bNumEndpoints` 指出端点的数量。
     - 这些端点在数组中没有特定的顺序。

4. **接口字符串（Interface String）**
   - `char *string`：接口字符串的指针（如果存在）。这个字符串通常用于描述接口的用途或功能。



1. **接口描述符（Interface Descriptor）**：
   - `desc` 字段包含一个 `usb_interface_descriptor` 结构体，这是一个标准的 USB 描述符，用于描述接口的特定设置。它包括了以下信息：
     - `bLength`：描述符的长度。
     - `bDescriptorType`：描述符的类型。
     - `bInterfaceNumber`：接口编号。
     - `bAlternateSetting`：备用设置编号。
     - `bNumEndpoints`：端点数量。
     - `bInterfaceClass`：接口类。
     - `bInterfaceSubClass`：接口子类。
     - `bInterfaceProtocol`：接口协议。
     - `iInterface`：接口字符串索引。

2. **额外描述符（Extra Descriptors）**：
   - `extra` 字段指向额外的描述符，这些描述符不在标准的接口描述符中。`extralen` 表示这些额外描述符的总长度。额外描述符可以包含特定于设备的额外信息，通常用于设备特有的功能。

3. **端点数组（Endpoint Array）**：
   - `endpoint` 字段是一个指向 `usb_host_endpoint` 结构体的指针数组，每个端点结构体描述一个端点的属性。端点是数据传输的终端点，每个端点有特定的地址和类型（如控制端点、中断端点、批量端点、同步端点）。
   - `desc.bNumEndpoints` 表示该接口设置所包含的端点数量。

4. **接口字符串（Interface String）**：
   - `string` 字段是一个指向接口字符串的指针（如果存在）。这个字符串通常用于描述接口的用途或功能，例如 "Audio Control Interface"。



`struct usb_host_interface` 结构体提供了 USB 接口的特定设置的详细信息，包括接口描述符、额外描述符、端点数组和接口字符串。每个 USB 接口可以有多个这样的设置，每个设置对应一组不同的端点配置，以支持不同的操作模式。这使得 USB 设备能够根据需要在不同配置之间切换，从而实现不同的功能。



USB 描述符主要有四种：设备描述符、配置描述符、接口描述符和端点描述符。协议中规定一个 USB 设备是必须支持这四种描述符



#### 端点

#### 设备

#### 设备配置
