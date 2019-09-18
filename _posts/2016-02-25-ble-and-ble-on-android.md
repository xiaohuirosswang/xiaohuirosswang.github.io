---
layout: post
title: "BLE and BLE on Android"
description: ""
category: [BLE, Android]
---

### Android 系统中的 BLE 以及开发者可以获取的数据

Android 应用进行蓝牙数据采集时需要下面的两个权限：

	<uses-permission android:name="android.permission.BLUETOOTH"/>
	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

前者在请求蓝牙连接、接受蓝牙连接以及接收蓝牙数据时需要，后者在进行蓝牙设备扫描以及更改蓝牙配置时需要。

[Android BLE 文档](http://developer.android.com/intl/zh-cn/guide/topics/connectivity/bluetooth-le.html) 提到：如果开发者希望自己的 App 只支持兼容 BLE 的 Android 设备，可以在 AndroidManifest.xml 文件中进行添加如下权限声明：

	<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>

`required="false"` 的时候，App 安装时不检查设备是否支持 BLE，开发者可以在代码中进行运行时检查。

我们可以通过监听蓝牙设备连接广播来监控设备蓝牙配对情况，获取下面的信息：

- 设备自身蓝牙硬件信息，比如蓝牙 Mac 地址，连接状态等。
- 已扫描到的蓝牙设备名称、蓝牙 Mac 地址等。
- 已配对的蓝牙设备名称、蓝牙 Mac 地址、UUID 列表（可以提供的 Services 列表）

已配对蓝牙设备支持的 Services 的 UUID 列表和该设备能够提供的功能是有 mapping 关系的，比如血压、遥控、心率、车载等。

[Services](https://developer.bluetooth.org/gatt/services/Pages/ServicesHome.aspx) 有官方已接受并成为规范的 Services 列表：

![](/images/services.png)

[Universally Unique Identifier (UUID)](http://bluetooth-pentest.narod.ru/doc/assigned_numbers_-_service_discovery.html) 的说明为：

> The Bluetooth Service Discovery Protocol (SDP) specification defines a way to represent a range of UUIDs (which are nominally 128-bits) in a shorter form. A reserved range of 232 values can be represented using 32-bits (denoted uuid32). Of these, a sub-range of 216 values can be represented using only 16 -bits (denoted uuid16). Any value in the 232 range that is not assigned in this document is reserved pending future revisions of this document. In other words, no value in this range may be used except as specified in this or future revisions of this document . UUID values outside of this range can be allocated as described in [ISO-11578](http://www.iso.org/iso/en/CatalogueDetailPage.CatalogueDetail?CSNUMBER=2229&scopelist=) for any purpose the allocater desires.

其中，__Base Universally Unique Identifier (UUID)__ 的定义为：

> The Base UUID is used for calculating 128-bit UUIDs from 'short UUIDs' (uuid16 and uuid32) as described in the SDP Specification See Service Discovery Protocol (SDP), Bluetooth SIG..

		Mnemonic	UUID
		BASE_UUID	00000000-0000-1000-8000-00805F9B34FB

生成 UUID 的时候，0000__XXXX__-0000-1000-8000-00805F9B34FB 里面的 `XXXX` 会被替换为规范中的值。

[Android BLE 常见的 Profiles 和 UUID](http://willings.me/article/android-bluetooth-profile-uuid/) 以及 [ATT 和 GATT 概述](http://legendmohe.net/2015/01/16/%E7%BF%BB%E8%AF%91att%E5%92%8Cgatt%E6%A6%82%E8%BF%B0/) 两篇文章讨论了蓝牙设备类型和 UUID 的对应关系，对于我们使用获取到的和一个设备进行过蓝牙配对的 UUID 进行配对蓝牙设备类型 mapping，进而进行人群画像方面的参考方面很有帮助。

从 [Android Example - Bluetooth Discover and List Devices and Services](http://digitalhacksblog.blogspot.ru/2012/05/android-example-bluetooth-discover-and.html) 这篇文章的测试看，监听蓝牙设备扫描结果的广播并获取扫描到的蓝牙设备支持的 Services 的代码如下：

	private final BroadcastReceiver ActionFoundReceiver = new BroadcastReceiver(){
 
		@Override
		public void onReceive(Context context, Intent intent) {
		 String action = intent.getAction();
		 if(BluetoothDevice.ACTION_FOUND.equals(action)) {
		   BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
		   out.append("\n  Device: " + device.getName() + ", " + device);
		   btDeviceList.add(device);
		 } else {
		   if(BluetoothDevice.ACTION_UUID.equals(action)) {
		     BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
		     Parcelable[] uuidExtra = intent.getParcelableArrayExtra(BluetoothDevice.EXTRA_UUID);
		     for (int i=0; i<uuidExtra.length; i++) {
		       out.append("\n  Device: " + device.getName() + ", " + device + ", Service: " + uuidExtra[i].toString());
		     }
		   } else {
		     if(BluetoothAdapter.ACTION_DISCOVERY_STARTED.equals(action)) {
		       out.append("\nDiscovery Started...");
		     } else {
		       if(BluetoothAdapter.ACTION_DISCOVERY_FINISHED.equals(action)) {
		         out.append("\nDiscovery Finished");
		         Iterator<bluetoothdevice> itr = btDeviceList.iterator();
		         while (itr.hasNext()) {
		           // Get Services for paired devices
		           BluetoothDevice device = itr.next();
		           out.append("\nGetting Services for " + device.getName() + ", " + device);
		           if(!device.fetchUuidsWithSdp()) {
		             out.append("\nSDP Failed for " + device.getName());
		           }
		            
		         }
		       }
		     }
		   }
		  }
		}
	};

可以得到的结果为：

![](/images/DiscoverDevicesAndServices.png)

### BLE (Bluetooth Low Energy / Bluetooth Smart)

[Bluetooth Development Portal](https://developer.bluetooth.org/TechnologyOverview/Pages/BLE.aspx) 对 BLE 的诞生背景介绍：

> When the Bluetooth SIG announced the formal adoption of Bluetooth® Core Specification version 4.0, it included the hallmark Bluetooth Smart (low energy) feature. This final step in the adoption process opened the door for qualification of all Bluetooth product types to version 4.0 and higher.
> 
> Bluetooth Smart (low energy) wireless technology features:
>
>- Ultra-low peak, average and idle mode power consumption  
>- Ability to run for years on standard coin-cell batteries  
>- Low cost  
>- Multi-vendor interoperability  
>- Enhanced range


[Introduction to Bluetooth Low Energy](https://learn.adafruit.com/introduction-to-bluetooth-low-energy?view=all) 对 BLE 的定义为：

> __Bluetooth Low Energy (BLE)__, sometimes referred to as "__Bluetooth Smart__", is a light-weight subset of classic Bluetooth and was introduced as part of the Bluetooth 4.0 core specification. While there is some overlap with classic Bluetooth, BLE actually has a completely different lineage and was started by Nokia as an in-house project called 'Wibree' before being adopted by the Bluetooth SIG.

#### BLE 目前的支持情况

Bluetooth 4.0 和 Bluetooth Low Energy (蓝牙 4.0 的子集) 在下列的主流平台得到支持:

- iOS 5+ (iOS7+ preferred)
- Android 4.3+ (numerous bug fixes in 4.4+)
- Apple OS X 10.6+
- Windows 8+ (XP, Vista and 7 only support Bluetooth 2.1)
- GNU/Linux Vanilla BlueZ 4.93+

[Bluetooth Development Portal](https://developer.bluetooth.org/TechnologyOverview/Pages/BLE.aspx) 提到：

> The latest Bluetooth specification uses a service-based architecture based on the __attribute protocol (ATT)__. All communication in low energy takes place over the __Generic Attribute Profile (GATT)__. An application or another profile uses the GATT profile so a client and server can interact in a structured way.

最新的蓝牙规范使用基于 __(ATT)__ 协议的服务型架构，所有 BLE 之间的通信都依赖于 __GATT__，所以下面需要了解一些 __GATT__ 的概念。

#### GATT (Generic Attribute Profile)

[GATT 官方介绍](https://developer.bluetooth.org/TechnologyOverview/Pages/gatt.aspx) 中提到：

> __Generic Attribute Profile (GATT)__ is built on top of the __Attribute Protocol (ATT)__ and establishes common operations and a framework for the data transported and stored by the __Attribute Protocol__. __GATT__ defines two roles: Server and Client. The __GATT__ roles are not necessarily tied to specific __GAP__ roles and may be specified by higher layer profiles. __GATT__ and __ATT__ are not transport specific and can be used in both __BR/EDR__ and __LE__. However, __GATT__ and __ATT__ are mandatory to implement in __LE__ since it is used for discovering services.

[Android BLE 文档](http://developer.android.com/intl/zh-cn/guide/topics/connectivity/bluetooth-le.html) 在介绍 __GATT__ 时提到：

> Generic Attribute Profile (GATT)—The GATT profile is a general specification for sending and receiving short pieces of data known as "attributes" over a BLE link. All current Low Energy application profiles are based on GATT.

Google 的介绍比较扼要：__GATT__ 是在 BLE 连接之间，发送和接受短数据块的一般性规范。所有 BLE 应用的配置文件都基于 __GATT__。

明确了 __GATT__ 的主要定义后，再来看看官方定义中提到的其他概念：

- __Attribute Protocol (ATT)__，[Android BLE 文档](http://developer.android.com/intl/zh-cn/guide/topics/connectivity/bluetooth-le.html) 的说明如下：

	> ATT is optimized to run on BLE devices. To this end, it uses as few bytes as possible. Each attribute is uniquely identified by a Universally Unique Identifier (UUID), which is a standardized 128-bit format for a string ID used to uniquely identify information. 

- __Basic Rate/Enhanced Data Rate (BR/EDR)__，[Bluetooth 官方文档](https://developer.bluetooth.org/TechnologyOverview/Pages/BREDR.aspx)的说明如下：

	> The Bluetooth RF (physical layer) operates in the unlicensed ISM band at 2.4GHz. The system employs a frequency hop transceiver to combat interference and fading, and provides many FHSS carriers. RF operation uses a shaped, binary frequency modulation to minimize transceiver complexity. __The symbol rate is 1 Megasymbol per second (Msps) supporting the bit rate of 1 Megabit per second (Mbps) or, with Enhanced Data Rate, a gross air bit rate of 2 or 3Mb/s. These modes are known as Basic Rate and Enhanced Data Rate respectively__.

	而且，Basic Rate 保证了来自不同生产商的蓝牙设备之间可以正常通信。

至于上面提到的 __Generic Access Profile (GAP)__ 在 BLE 

#### __Generic Access Profile (GAP)__

[Introduction to Bluetooth Low Energy](https://learn.adafruit.com/introduction-to-bluetooth-low-energy?view=all) 文章中对 __GAP__ 的说明为：

> GAP is an acronym for the Generic Access Profile, and it controls connections and advertising in Bluetooth. GAP is what makes your device visible to the outside world, and determines how two devices can (or can't) interact with each other.

__GAP__ 控制着蓝牙设备之间的连接和广播，定义了一个蓝牙设备是否可被扫描到，也决定着两个蓝牙设备之间是否可以进行交互以及交互的方式。

[GATT Profile 简介](http://www.race604.com/gatt-profile-intro/) 在对 [Introduction to Bluetooth Low Energy](https://learn.adafruit.com/introduction-to-bluetooth-low-energy?view=all) 翻译的基础上，添加了一些实际使用场景中的实例（iBeacon 的实现基础），里面详细介绍了:

-  __GAP__ 定义的最重要的两种角色（__Peripheral (外围角色) & Central (中心角色)__）
-  广播和扫描响应数据 （__Advertising and Scan Response Data__）
-  广播机制以及广播网络拓补图解析
-  __GATT__ 连接网络拓补图解析
-  __GATT__ 交互机制以及数据结构
	- [Profiles](https://developer.bluetooth.org/TechnologyOverview/Pages/Profiles.aspx)

		> To use Bluetooth® wireless technology, a device must be able to interpret certain Bluetooth profiles. Bluetooth profiles are definitions of possible applications and specify general behaviors that Bluetooth enabled devices use to communicate with other Bluetooth devices. There is a wide range of Bluetooth profiles describing many different types of applications or use cases for devices. By following the guidance provided by the Bluetooth specification, developers can create applications to work with other Bluetooth devices.
	- [Services](https://developer.bluetooth.org/gatt/services/Pages/ServicesHome.aspx)

		> Services are collections of characteristics and relationships to other services that encapsulate the behavior of part of a device.
	- [Characteristics](https://developer.bluetooth.org/gatt/characteristics/Pages/CharacteristicsHome.aspx)

		> Characteristics are defined attribute types that contain a single logical value. 

		![](http://jlog.qiniudn.com/microcontrollers_GattStructure.png)

通过 __GAP__ 建立 BLE 连接后，就可以通过 __GATT__ 配置文件进行数据交互了，[需要注意](http://www.race604.com/gatt-profile-intro/) 的是：

- __GATT__ 连接的独占性

	> The most important thing to keep in mind with GATT and connections is that connections are exclusive. What is meant by that is that __a BLE peripheral can only be connected to one central device (a mobile phone, etc.) at a time!__ As soon as a peripheral connects to a central device, it will stop advertising itself and other devices will no longer be able to see it or connect to it until the existing connection is broken.

	GATT 连接是独占的。也就是一个 BLE 外设同时只能被一个中心设备连接。一旦外设被连接，它就会马上停止广播，这样它就对其他设备不可见了。当设备断开，它又开始广播。

- Service 和 Characteristics 各自 UUID 的长度 
	UUID 有 16 bit 的，或者 128 bit 的。16 bit 的 UUID 是官方通过认证的，需要花钱购买，128 bit 是自定义的，这个就可以自己随便设置。

### 更多参考资源

- [Android GATT 连接过程源码分析](http://www.race604.com/gatt-connect/) 对于理解 Android 平台 BLE 的实现很有帮助。
- [BLE 广播数据解析](http://www.race604.com/ble-advertising/) 详细解析了 __GAP__ 连接时广播数据的结构和解析过程。
- [Android 的蓝牙简介](http://www.race604.com/android-bluetooth-intro/)
- [Assigned Numbers - Service Discovery](http://bluetooth-pentest.narod.ru/doc/assigned_numbers_-_service_discovery.html)，官方已经接受并成为规范的 Services、Attributes 以及 Protocols


