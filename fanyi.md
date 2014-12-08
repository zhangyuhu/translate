
###蓝牙低功耗说明
这一章描述了ble协议栈的不同层，这些层包含的组件，和他们的概念
####2.1 GAP层(通用接入规范)
GAP是蓝牙协议栈最底层的应用程序接口。它包含管理广播和连接等的一些参数。
note：

#####2.1.1  角色
#####2.1.2  广播
#####2.1.3  扫描
#####2.1.4  初始化
#####2.1.5  连接


####2.2GATT(通用属性配置)
GATT是数据实际上被用来转移的一层

#####2.2.1 角色
除了GAP中的角色之外，BLE也定义的两个角色，一个GATT服务器和一个GATT客户端，那完全独立于GAP的角色。这个概念意味着当他包含数据的时候作为一个服务器，当作为一个访问的时候，它是一个客户端
对于led，按键的例子，外围设备（带有led灯和按键——作为服务器的角色，中心设备作为客户端的角色
#####2.2.2 GATT的层级
一个GATT服务器通过调用一个属性表来组织数据并且那些包含的真实的数据就是属性
######2.2.2.1 属性
一个属性包括一个句柄，一个UUID，和一个值。一个句柄是GATT表中属性的索引，并且是唯一的对于每个设备中的每个属性。UUID包含有属性的数据类型的信息，对于理解包含的属性值是关键的。UUID将不是唯一的在一个设备内有几个属性可能带有相同的UUID。
######2.2.2.2 特征值
一个特征至少包含两个属性：一个特征的定义和一个持有特征值得属性
对于LED按键的例子，我们使用特征代表当前的按键状态和当前led灯的状态
######2.2.2.3 描述符
任何宣称不是特征值的属性，都被定义成一个描述符。一个描述符可以提供更多的关于特征值得信息，例如可读的特征值得描述
然而有一个特殊的描述符特别值得一提：（CCCD),这个描述符被增加到任何支持通知或者知识属性的特征值里。详情查看2.2.5节，第10页，On-air operations and properties.
写“1”到CCCD打开通知，写“2”打开指示。写“0”关闭通知和指示
对于S110 SoftDevice 这个描述符被自动增加到通知和指示被设置的特征值的地方
######2.2.2.4 服务
一个服务包含一个或者更多的特征值，那是一个逻辑相关的特征值集合   
对于LED灯按键的例子，所有的特征值都被分组到一个服务  
######2.2.2.4 规范
一个规范被定义成一个或多个服务用例的配置文件。一个配置文档包括服务的信息和它怎样被使用。因此，这个文档包含各种各样的广播信息，和被使用的连接的时间间隔，安全性是否被需要等等   
值得注意的是一个规范并在属性表中并没有一个属性   
对于led和按键的例子，规范没有显示的描述   
#####2.2.3与标准相对的自定义服务和特征值
Bluetooth SIG基于GATT层的协议栈定义了规范，服务，特征值和属性的序号。然而蓝牙低功耗所有服务的实现是应用程序的一部分，而不是协议栈的一部分，那就意味着应用程序可以支持任何想要的规范和服务，只要协议栈支持GATT   
因为支持应用中的规范和服务，所以在程序中实现自定义的服务是可能的    
对于LED灯和按键的例子，没有涵盖Bluetooth SIG服务的用例，因此它将被实现为带着两个特征值得一个自定义服务

#####2.2.4 UUIDs

######2.2.4.1 UUID

######2.2.4.2 Alias

#####2.2.5 空中操作和属性

######2.2.5.1 写和不带反馈的写
 
######2.2.5.2 读

######2.2.5.3  通知和指示


###迷你的BLE应用预览

####3.2 S110 SoftDevice
####3.3 广播
####3.4 连接参数
SDK来自于一个称为  ble_conn_params 模块，它管理着连接参数的更新，随着需要。它对于应用调用的SoftDevice的API进行了封装，包括处理定时器的请求。   
在初始化结构体 ble_conn_params_init_t中，不同的参数更新程序能被设置。例如，它是否应该开始连接，在写一个特定CCCD的时候是否开始，哪种连接参数应该被使用，在请求被发送前是否延时，等等。  

####3.5 服务
####3.6 特征值

###LED Button应用例程 
####4.1 代码预览
一个应用怎样工作的预览在下面章节被提供帮助你理解和使用代码

#####4.1.1 代码分离
例程代码被分成三个文件：
 >>
  * main.c
  * ble_lbs.c
  * ble_libs.h

这个结构与其他的SDK例程是相似的，main.c执行主程序，分开的服务文件执行服务和它自己的代码。所有的I/O处理由应用决定  
#####4.1.2  code流程
任何运行在NRF51822芯片的BLE应用的基础流程是初始化所有需要的部分，开始广播，接下俩可能进入低功耗模式等待BLE事件。在事件收到的时候，它会转到相应的BLE服务和模块。一个事件，但不限于以下之一：

>>
 * 一个对等的设备连接NRF51822
 * 一个对等的设备写一个特征值
 * 一个指示广播有超时的指示  

流程是应用非常的模块化，并且一个服务能被增加到应用中通过初始化它，并确保它的事件句柄在事件到来的时候被调用

#####4.1.3  进入Keil项目 

我们建议你编译例程在你开始浏览它之前。在它被编译之后，你能右键进入任何的函数，变量，类型，或者宏定义，进入它定义的地方，通过选择 Go To Definition Of 或者 Go To Reference To  
Go To Definition Of带你进入实际方法的实现（即，源代码文件），Go To Reference To带你进入他的头文件声明。对于不可用的源代码， 你可以进入它的声明在SoftDeviceAPI中定义的（这应该是针对没有开放出的一部分）。你不论以何种方式进入函数的声明，都将展示给你可用的文档对于问题的方法。这是一个非常强大的工具使你能熟悉API和例程项目。
####4.2 安装
nRF51822评估套件需要这个应用例程。然而用开发套件修改项目也可以工作。
#####4.2.1 安装评估板
因为nRF51822评估套件有个板载的SEGGER芯片，你可以通过usb线缆连接评估板并立即使它工作。  
#####4.2.2 安装应用
许多引用代码以创建一个应用和服务开始，因此第一步是从SDK复制代码：
1.进入Board\nrf6310\ble\ble_app_template 文件夹。（注意：我没有找到ble文件夹，我的是在s110下）。
2.复制这个文件夹 Board\pca10001\ble\ 并且重命名它为  ble_app_lbs.
3.在ble_app_libs的子文件夹Arm里面改变项目的名字从ble_app_template改为ble_app_libs 
#####4.2.3 安装服务
SDK并没有一个模板服务。但是它有一个电源服务的实现，那是个简单的服务的实现，并且是一个好的自定义服务的起始点。按照下面的步骤开始：
1. 从 Source\ble\ble_services复制ble_bas.c文件到 Board/pca10001/ble/ble_app_lbs/文件夹下
2. 从 Include\ble\ble_services复制ble_bas.h到 Board/pca10001/ble/ble_app_lbs/文件夹下
3. 将ble_bas.c重命名为ble_libs.c,ble_bas.h重命名为ble_lbs.h
4. 双击工程中的服务文件夹，选择新创建的ble_lbs.c增加ble_lbs.c文件到你的Keil项目
因为这是一个特定服务的应用，因此更好是一个应用文件夹而不是SDK服务文件夹。
####4.3 实现服务
这个服务被很通用的实现以至于它很容易的被重用在其他的应用程序中。目标是为了打开应用通过初始化，处理事件，和提供IO实现来使用服务。这类似于预定义服务的实现方式。
#####4.3.1 设计API
ble_lbs.h头文件实现了应用程序需要实现的各种结构体，一个事件句柄和三个API方法：

	uint32_t ble_bas_init(ble_bas_t * p_bas, const ble_bas_init_t * p_bas_init);
	void ble_bas_on_ble_evt(ble_bas_t * p_bas, ble_evt_t * p_ble_evt);
	uint32_t ble_bas_battery_level_update(ble_bas_t * p_bas, uint8_t battery_level);   
	注意：内容已经被移到了这个文档的代码片段  

在上面的的代码中，ble_bas_t被用来作为服务的引用然而ble_bas_init_t包含初始化参数以后并不一定有用。所有的AOI方法都以一个指向服务实例的指针作为第一个参数。  
为了设计一个相似的API为led button服务，按下面的步骤来：
1.找到并替换所有的ble_bas为ble_lbs
2.考虑一下来自应用的服务的需求。LED Button服务需要知道按键什么时候状态已经改变，并且被发送到中心设备。因此，你需要增加一个方法调用在按键状态改变的时候。  

	uint32_t ble_lbs_init(ble_lbs_t * p_lbs, const ble_lbs_init_t * p_lbs_init);
	void ble_lbs_on_ble_evt(ble_lbs_t * p_lbs, ble_evt_t * p_ble_evt);
	uint32_t ble_lbs_on_button_change(ble_lbs_t * p_lbs, uint8_t button_state);

3.在这有两种数据结构需要被实现，ble_lbs_t 和 ble_lbs_init_t  

#####4.3.2 数据结构的实现
在上面16页4.3.1节的API中，一些数据结构还没有被实现尽管被用：ble_lbs_t和ble_init_t。  
我们可以基于电源服务的相似的数据结构，看起来像下面这样。

	typedef struct
	{
    	ble_bas_evt_handler_t         evt_handler;                    
    	bool                          support_notification;           
    	ble_report_ref_t *        p_report_ref;                   
    	uint8_t                       initial_batt_level;             
    	ble_cccd_security_mode_t  battery_level_char_attr_md;     
    	ble_gap_conn_sec_mode_t       battery_level_report_read_perm; 
	} ble_bas_init_t;
	typedef struct ble_bas_s
	{
    	ble_bas_evt_handler_t         evt_handler;                    
   		uint16_t                      service_handle;                 
    	ble_gatts_char_handles_t      battery_level_handles;          
    	uint16_t                      report_ref_handle;              
    	uint8_t                       battery_level_last;             
    	uint16_t                      conn_handle;                    
    	bool                          is_notification_supported;      
	} ble_bas_t;

上面代码中的初始化结构体包含一个事件处理，一些可选的参数，初始值，和安全模式关于这个服务信息的。这个服务的结构另一方面包含服务的状态，像句柄，当前的电源值，通知是否打开等等。  
电源服务使用了一个通用的事件处理为了让应用程序周期性的开始和停止电源值。led button服务并不依赖与任何启动和停止事件。因此用一个单一的方法就足够了在特征值被写入的时候。  

这个句柄唯一的事情是适用于初始化，在初始化结构体中，它将变成唯一的成员。  

	typedef struct
	{
    	ble_lbs_led_write_handler_t   led_write_handler;                   
	} ble_lbs_init_t;

函数指针将被定义成下面这样：

	typedef void (*ble_lbs_led_write_handler_t) (ble_lbs_t * p_lbs, uint8_t new_state);

然而，下面的参数需要跟踪状态：

>>
 * 对于服务的句柄
 * 特征值和连接句柄
 * UUID类型
 * led写处理

这些参数将给出像下面的服务结构体： 

	typedef struct ble_lbs_s
	{         
		uint16_t                     service_handle;                 
		ble_gatts_char_handles_t     led_char_handles;          
		ble_gatts_char_handles_t     button_char_handles;          
		uint8_t                      uuid_type;
		uint8_t                      current_led_state;             
		uint16_t                     conn_handle;  
		ble_lbs_led_write_handler_t  led_write_handler;
	} ble_lbs_t;

电源服务事件声明在ble_lbs.h文件中被删除

#####初始化  
1.在源文件ble_lbs.c中，替换所有出现的ble_bas和p_bas为ble_lbs和p_lbs.
2.删除ble_lbs_battery_level_updata()函数注释掉所有的其他方法通过使用#if 0和#endif在ble_lbs.c文件的第一个方法之前，在初始化函数之上。不要完全删除它们，因为它们中的一些方法在后面会被使用到。
 ######设计服务初始化 
在初始化之后，首先看看初始化函数，现在调用ble_lbs_init。下面的参数你不需要将被移除：
1.移除 evt_handler,  is_notification_supported, 和battery_level_last
2.重命名 evt_handler为led_write_handler,再初始化结构体和服务结构体
	p_lbs->led_write_handler         = p_lbs_init->led_write_handler;

UUID处理需要重新加工因为服务奖被用一个自定义的UUID来实现代替 Bluetooth  SIG中定义的UUID  
首先定义一个base UUID 一种方法是在nRFgo Studio中：
1.打开nRFgo Studio
2.在 nRF8001 Setup 菜单中，选择Edit 128-bit UUIDs并且点击Add new

这将创建一个个新的，随机的UUID你能用来自定义服务。 
新建的Base UUID必须被包含进源代码文件中作为一个字节数组，但是只在一个地方需要
1.为了增加可读性，在头文件ble_lbs.h中包含她作为一个宏定义，连同服务和特征值的别名。
	#define LBS_UUID_BASE {0x23, 0xD1, 0xBC, 0xEA, 0x5F, 0x78, 0x23, 0x15, 0xDE, 0xEF, 
	0x12, 0x12, 0x00, 0x00, 0x00, 0x00}
	#define LBS_UUID_SERVICE 0x1523
	#define LBS_UUID_LED_CHAR 0x1525
	#define LBS_UUID_BUTTON_CHAR 0x1524  

在服务初始化中：
2.增加下面的UUID到协议栈列表，然后安装服务并使用它。在ble_lbs_init()中第一次被增加  

	ble_uuid128_t base_uuid = LBS_UUID_BASE;
	err_code = sd_ble_uuid_vs_add(&base_uuid, &p_lbs->uuid_type);
	if (err_code != NRF_SUCCESS)
	{
    	return err_code;
	}
上面的代码片段将增加自定义Base UUID到协议栈，并且存储类型通过sd_ble_uuid_vs_add()在服务结构体中：

3.使用类型在为LED Button服务设置UUID的时候：
	ble_uuid.type = p_lbs->uuid_type;
	ble_uuid.uuid = LBS_UUID_SERVICE;
	err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, 
	&p_lbs->service_handle);
	if (err_code != NRF_SUCCESS)
	{
		return err_code;
	}

以上的代码只是增加了一个空的服务，一次特征值需要被增加。下面的章节将解释怎样实现特征值。

######4.3.3.2 实现按键特征值
这个服务将有两个特征值，一个控制led状态，一个反应按键状态。两个静态的方法需要被创建增加特征值到ble_lbs.c中以按键状态开始。
按键特征值通知按键状态改变，但是也允许对等设备读取按键状态。这是非常相似的行为