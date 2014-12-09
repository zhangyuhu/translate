
###2蓝牙低功耗说明
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
如第7页2.2.2节“GATT hierarchy”所述，所有的属性都有一个UUID，一个UUID是一个128位的数字是全球唯一的。蓝牙核心规范将它们分为baseUUID和一个别名。

######2.2.4.1 base UUID
下面的baseUUID被Bluetooth SIG用在他们所有定义的属性中，所有Bluetooth SIG属性，最后的16位（别名）足够对他们进行唯一识别了。

	Full UUID — 0x00001234-0000-1000-8000-00805F9B34FB
	Corresponding Base UUID — 0x0000xxxx-0000-1000-8000-00805F9B34FB

不要使用Bluetooth SIG Base UUID对于任何自定义的开发。相反，生成你自己的Base UUID来创建新的UUIDs。

	For the LED Button example, 0x0000xxxx-1212-EFDE-1523-785FEABCD123 will be used as the
	base.

######2.2.4.2 Alias
别名是被定义的Base UUID的其余的16位。The Bluetooth Core Specification对于怎样设计别名没有任何规则和建议，因此你使用任何你想到的计划。

	For the LED Button example, 0x1523 is used for the service, 0x1524 for the LED characteristic,
and 0x1525 for the button state characteristic

#####2.2.5 空中操作和属性
所有的空中操作通过使用句柄产生，能唯一的标识每个属性。
特征值依赖于它的属性。可能的属性如下：

* 写
* 无反馈的写
* 读
* 通知
* 指示

更多的属性被定义在蓝牙规格书里面，但是这些更加常用
######2.2.5.1 写和不带反馈的写
 写和不带反馈的写允许GATT客户端写一个到为GATT服务端的特征值。它们中的不同是不带反馈的写没有任何应用应答，只有广播应答也会发生。 
######2.2.5.2 读
读属性时GATT客户端从GATT服务器读一个特征值成为可能。
######2.2.5.3  通知和指示
通知和指示  
通知和指示允许GATT服务器使GATT客户端了解特征值得变化。通知和指示的不同是只用有应用级别的应答，然而通知没有。  
对于LED和案件的例程，特征值可以控制LED灯并且对于当前的按键状态有两种自定义的特征值用在LED 按键服务中。
对于LED的特征值，中心设备需要能设置它的值，并且可以读它。既然应用程序级别的应答不需要，你能用不带应答的写和读属性。 
对于按键特征值，客户端需要被通知在按键状态被改变的时候。但是应用级别的应答不需要，在这里只有通知属性被需要。
注意：GATT和它所代表的协议的描述Bluetooth Core Specification的卷三，F和G部分。
控制led的特征值和对于当前按键状态的特征值是两个自定义的特征值被用在这里的LED Button服务中。   


###3 最低要求的BLE应用预览
这章提供了一个最低要求的BLE应用的预览在nRF51822设备上使用S110 SoftDevice
####3.1初始化预览 
有几个初始化调用通常被执行作为Ble应用的一部分。下面提供了一个初始化调用的列表在稍后的文档中将作更多的解释。  


上面描述的方法用作输入参数的结构体。这结构体指定了一个配置或选项的设置以及代码中的注释使它们更好的理解。  
你能进入主循环在广播开始后。
####3.2 S110 SoftDevice
你必须打开S110 SoftDevice为了使用它，那将使得能够独自访问广播外设。
查看S110 nRF51822 SoftDevice Specification得到更多硬件资源使用的详细信息。
####3.3 广播
用于广播的结构体类型在ble_gap.h头文件的ble_gap_conn_sec_mode_t和ble_advdata.h的ble_advdata_t 

	err_code = ble_gap_device_name_set(&device_name_sec_mode, DEVICE_NAME
	strlen(DEVICE_NAME));
	err_code = ble_gap_appearance_set(BLE_APPEARANCE_UNKNOWN);
	err_code = ble_advdata_set(&advdata);
注意：安全模式通过ble_gap_device_name_set()应用于唯一的设备
广播参数(ble_gap_adv_params_t) need to be passed to ble_gap_adv_start():

	err_code = ble_gap_adv_start(&m_adv_params);
####3.4 连接参数
SDK来自于一个称为  ble_conn_params 模块，它管理着连接参数的更新，随着需要。它对于应用调用的SoftDevice的API进行了封装，包括处理定时器的请求。   
在初始化结构体 ble_conn_params_init_t中，不同的参数更新程序能被设置。例如，它是否应该开始连接，在写一个特定CCCD的时候是否开始，哪种连接参数应该被使用，在请求被发送前是否延时，等等。  

ble_conn_params_init()函数然后携带一个ble_conn_params_init_t struct结构体，它带着初始化了的连接参数（ble_gap_conn_params_t）将被需要。

		err_code = ble_conn_params_init(&cp_init);
ble_conn_params.h文件确保连接参数决定主机可接受。如果不可接受，它将要求改变他们。在在尝试设置一些列的数字改变他们没有成功之后，它将断开或者给出一个事件返回应用程序这依赖于配置。
####3.5 服务
服务可以通过ble_gatts_service_add()增加。你不应该在应用代码中这样做，相反应将服务分成几个文件。一个服务有一个类型，主要或次要的，但是在实际中，只有主要的服务被使用。service_uuid变量是你想使用的服务的UUID。service_handle变量是一个输出变量，那将被一个唯一的句柄填充在服务被创建时。句柄在后边用来识别服务。

	err_code = ble_gatts_service_add(BLE_GATTS_SVC_TYPE_PRIMARY,
								&p_lbs->service_uuid,
								&p_lbs->service_handle );
####3.6 特征值
特征值通过ble_gatts_characteristic_add(),来增加，带有四个参数，这个调用只应该发生在服务文件里，不在应用里。
第一个参数是一个应该被增加的服务的句柄。第二个参数是一个特征值得元数据的结构体，它包含有操作变量的信息（读，写，通知，等等），第三个参数是一个值属性的描述符，它包含它的UUID，长度和初始化值。最后的参数被一个特征值得唯一的句柄填充。这个句柄在后面被用来识别这个特征值。例如，识别哪个特征值发生写事件。

	sd_ble_gatts_characteristic_add(p_lbs->service_handle, &char_md,
								&attr_char_value,
								&p_lbs->led_char_handles);

###LED Button应用例程 
LED按键应用例程被创建为了给你提供一个怎样使用基于nRF51822芯片蓝牙低功耗的环境。他是简单的BLE应用，演示了在BLE上的双向通信。在它运行的时候，你将能切换LED输出在nRF51822芯片上来自中心设备，并且收到一个通知在按键输入被切换的时候在nRF51822上。  
应用被实现作为一个服务，带有两个特征值--LED特征值，能被用来远程控制led灯通过没有反馈的写操作，按键特征值，发送一个通知在按键被按下或释放的时候
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

#####4.3.3 初始化  
1.在源文件ble_lbs.c中，替换所有出现的ble_bas和p_bas为ble_lbs和p_lbs.
2.删除ble_lbs_battery_level_updata()函数注释掉所有的其他方法通过使用#if 0和#endif在ble_lbs.c文件的第一个方法之前，在初始化函数之上。不要完全删除它们，因为它们中的一些方法在后面会被使用到。
######4.3.3.1 设计服务初始化 
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
按键特征值通知按键状态改变，但是也允许对等设备读取按键状态。这是与电源服务中电源值是非常相似的行为，因此你能像下面基于你的实现：
1.找到调用 battery_level_char_add 的方法并且重命名为 button_char_add，如果找到并且替换后尽快工作的话，参数名字也应该是正确的。button_char_add方法期望找到一个参数声明是否支持通知，但是在这种情况下通知支持是想要的。   
2.移除if语句检查is_notification_supported参数，但是代码要保留。（意思应该是直接把用不到的注释掉）   
3.确保 char_md fields  被设置为支持通知。
4.移走所有的关于report声明的代码路径，  p_report_ref参数（来自于if语句中的每个东西，检查p_report_ref是否被设置）
有一个标志位在电源服务初始化中用来设置安全模式为CCCD，并且被存储在ble_gap_conn_sec_mode_t 这个结构体中。这样的结构体很容易通过使用来自 ble_gap.h中的以ble_gap_conn_sec_mode开头的宏来设置。有不同的宏可以使用，依赖于你想要求的这个属性的安全等级。 
你想要支持一种安全模式可以通过设置两个值，加密模式和安全等级，使用宏。有不同的宏对于不同的使用情况，例如，要求加密，给出给出正常的访问或者只是部分访问：
1.使用 BLE_GAP_CONN_SEC_MODE_SET_OPEN，这将使得CCCD可读和可写在任何环节下，加密或不加。这同样适用于读和写参数对于按键自己的状态。    

	BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cccd_md.read_perm);
	BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cccd_md.write_perm);
	memset(&attr_md, 0, sizeof(attr_md));
	BLE_GAP_CONN_SEC_MODE_SET_OPEN(&attr_md.read_perm);
	BLE_GAP_CONN_SEC_MODE_SET_OPEN(&attr_md.write_perm);

2.确保你没有移除vloc字段的设置，那将决定变量是放置在栈内存中或是应用程序内存中。  
3.设置唯一的权限在将结构体使用memset清零后  
4.设置类型和值对于得到的正确的UUID值 

	ble_uuid.type = p_lbs->uuid_type;
	ble_uuid.uuid = LBS_UUID_BUTTON_CHAR;

5.初始化值是不重要的，因此你可以将p_initial_value 初始化为NULL
6.确保存储特征值得句柄在正确的地方，向下面一样修改最终的调用方法。

	return ble_gatts_characteristic_add(p_lbs->service_handle, &char_md,
									&attr_char_value,
									&p_lbs->button_char_handles);    

通过移除上面方法中的#endif你能编译这个应用程序并且移除任何没有使用的变量根据编译出来的警告（initial_battery_level, encoded_report_ref, init_len, err_code）   

######4.3.3.3 实现LED特征值    
LED特征值需要可读和可写，无任何通知：
1.复制增加的按键特征值得方法，重命名为led_char_add 
2.移除cccd_md的引用   
3.增加写属性代替通知属性（打开写这个特征值） 

	char_md.char_props.write = 1;  

4.改变UUID为 LBS_UUID_LED_CHAR 

	ble_uuid.type = p_lbs->uuid_type;
	ble_uuid.uuid = LBS_UUID_LED_CHAR;

句柄应该被存储在变量 led_char_handles 中代替 button_char_handles 最终调用的方法像下面这样：

	return ble_gatts_characteristic_add(p_lbs->service_handle, &char_md,
									&attr_char_value,
									&p_lbs->led_char_handles);

再编译之后，你能看到现在没有被使用到的属性，移除它们（cccd_md, err_code）  

######4.3.3.4 增加特征值 
在增加完创建特征值得方法后，你需要调用它们在服务的初始化方法的后面。下面是怎样使用的例程：  

	// Add characteristics
	err_code = button_char_add(p_lbs, p_lbs_init);
	if (err_code != NRF_SUCCESS)
	{
		return err_code;
	}
	err_code = led_char_add(p_lbs, p_lbs_init);
	if (err_code != NRF_SUCCESS)
	{
		return err_code;
	}
	return NRF_SUCCESS;  

因为有任何错误都将立即被返回，如果你到达了函数的末尾，说明你成功了。

如果你想现在进行测试，你你能到4.4.1节 23页的“Modifying the template for the Evaluation Kit“和24页的4.4.3“Including the service”节看看，完成进行测试将在29页第五章“Testing the Application”。在测试之后，你就可以连接设备并且发现所有的服务，但是更进一步的动作将不会工作。处理协议栈事件和按键事件需要在服务中被实现。   

#####4.3.4 处理协议栈事件  

一个协议栈事件发生在连接建立，写一个特征值或者描述符等等时间的时候。对于应用程序来说，给led写特征值是你需要的。然而，为了使通知正确的工作，你需要存储连接句柄，你能连接或者断开事件。  
作为API的一部分，你定义一个调用 ble_lbs_on_ble_evt 的方法被用来处理协议栈事件。这被分成了不同的方法来处理不同的事件基于一个简单的switch-case语句。    
######4.3.4.1 存储连接句柄  
电源服务已经存储了连接句柄，查找和替换前面还没有任何改变的部分
######4.3.4.2 移除CCCD写的处理   
前面存在的事件句柄监听CCCD的写事件并通过它们传递给应用的电源服务句柄，然而，在这个程序中，这并不重要。   
文档中发送通知的方法，sd_ble_gatts_hvx() 如果CCCD没有被打开它将不允许通知被发送，因此你不需要再应用中检查出了在SoftDevice里面进行检查之外。    
你可以移除所有当前的在 on_write方法中的内容了。   
######4.3.4.3 LED特征值写处理 
在led特征值被改写的时候你添加到数据结构的函数指针将允许应用程序通知。这应该在on_write方法中被处理。 
写时的基本的任务是确认写入了正确的特征值，确认它有正确的大小，及一个处理被设置。如果这所有的都是正确的，处理就能被调用无论写入了什么值。因此，on_write的内容变成了下面这样：

	ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
	if ((p_evt_write->handle == p_lbs->led_char_handles.value_handle) &&
		(p_evt_write->len == 1) &&
		(p_lbs->led_write_handler != NULL))
	{
		p_lbs->led_write_handler(p_lbs, p_evt_write->data[0]);
	}  

实际的LED灯的切换留给应用程序，使服务容易的按原样重用，尽管不同的引脚分配给LED和按键。  

#####4.3.5 按键按压处理  
你已经增加了一个API方法在按键被按下的时候让服务知道。但还没有被实现，因此你需要增加它通过从头文件中复制定义。在处理按键按压的时候，你想发送一个通知到对等的设备带着新的按键状态。SoftDevice API 通过调用 sd_ble_gatts_hvx,并带着一个连接句柄和一个ble_gatts_hvx_params_t作为参数。然后接管进程在值被通知到的时候。  
在ble_gatts_hvx_params_t结构体中，你 可以设置你是否想要一个通知或者指示，属性句柄被通知，新的数据和新的数据长度。方法看上去像下面：  

	uint32_t ble_lbs_on_button_change(ble_lbs_t * p_lbs, uint8_t button_state)
	{
		ble_gatts_hvx_params_t params;
		uint16_t len = sizeof(button_state);
		memset(&params, 0, sizeof(params));
		params.type = BLE_GATT_HVX_NOTIFICATION;
		params.handle = p_lbs->button_char_handles.value_handle;
		params.p_data = &button_state;
		params.p_len = &len;
		return sd_ble_gatts_hvx(p_lbs->conn_handle, &params);
	}

那也是可能的第一次设置特征值通过使用sd_ble_gatts_value_set()，然后通知没有设置值或者长度通过调用sd_ble_gatts_hvx()。然而使用sd_ble_gatts_hvx()做事情是必要的，这是一个清楚的方法。使用方法sd_ble_gatts_value_set()来更新可读的值（但是不是notify-able），因为函数并没有发送通知
服务结论，例程的其他部分将实现应用程序   

#### 4.4 应用实现  
#####4.4.1 为评估套件修改模板  
一些修改是必须的对于使用评估套件的模板应用程序，而不是开发套件。  
你需要修改main.c，包括对于正确的套件的引脚定义。向下面那样改变ble_nrf6310_pins.h 为 ble_eval_board_pins.h：

	#include "ble_eval_board_pins.h"

1.移除包含的ble_error_log.h,，那是不需要的  
2.在main.c里，在button_init中改变WAKEUP_BUTTON_PIN NRF6310_BUTTON_0 to
EVAL_BOARD_BUTTON_0的定义    
3.移除nrf_gpio_pin_set(ASSERT_LED_PIN_NO)来自应用程序的错误处理函数  
4.移除leds_init()中的GPIO_LED_CONFIG(ASSERT_LED_PIN_NO)

代替assert LED,你需要一个在服务中使用的LED灯，因此增加led引脚的定义在main.c的最上面   

	#define LEDBUTTON_LED_PIN_NO EVAL_BOARD_LED_0

然后，在leds_init()，配置它作为一个LED： 

	GPIO_LED_CONFIG(LEDBUTTON_LED_PIN_NO);

在SDKs 4.1.0和更新版本中，默认应用错误处理执行重置在错误发生的时候，但是为了开发使用提供的调试模块是非常有用的。你应该确保取消调试处理的注释，为重置添加注释：

	void app_error_handler(uint32_t error_code, uint32_t line_num, const uint8_t *
		p_file_name)
	{
		// [Comment removed from snippet for brevity]
		ble_debug_assert_handler(error_code, line_num, p_file_name);
		// On assert, the system can only recover with a reset.
		//NVIC_SystemReset();
	}

在实际的产品中，重置是唯一的恢复选项，但是为了开发，日志是非常有用的。   

再此也建议改变项目中的蓝牙的名字，通过改变DEVICE_NAME在main.c中定义的像“LedButtonDemo”.  

#####4.4.2 使用调度
SDK包含了调度模块来提供一种机制用来处理事件和来自主程序中中断处理的中断。这能确保所有的中断程序被非常快速的执行。   
我们一应用程序的模板开始，调度被默认打开。如果你不想使用它，你能移除它的初始化和主循环中对它的调用（scheduler_init(), app_sched_execute()），并且设置SDK模块的初始化函数的最后一个参数为false（ble_stack_handler, app_timer,
app_button）  
为了得到关于调度的更详细的信息，请查看nRF51 SDK 文档。  
#####4.4.3 包含服务  
为了使用你创建的服务，你需要增加代码到模板应用中。
在main.c文件中，有一个调用services_init的方法在初始化LED 按键服务的地方必须发生：
1.增加ble_lbs.h的包含在main.c的最顶部：

	#include "ble_lbs.h"

2.如果还没有做，增加服务的原文件到项目。右键服务文件夹在Project explorer的左边，点击Add file并且选择ble_lbs.c文件. 
3为服务增加一个数据结构作为一个全局的变量在main.c文件中：

	static ble_lbs_t m_lbs;
	注意：尽管这作为一个静态变量存储在main.c文件中，，m_lbs，它将经常出现作为这个变量的指针，因此被称为p_lbs.  

现在你可以在services_init中做初始化了：

	static void services_init(void)
	{
		uint32_t err_code;
		ble_lbs_init_t init;
		init.led_write_handler = led_write_handler;
		err_code = ble_lbs_init(&m_lbs, &init);
		APP_ERROR_CHECK(err_code);
	}

在第22页的4.3.4.3节“Handling LED characteristic writes” 中，我们创建的服务调用led_write_handler在服务结构体中当LED特征值被写入的时候。我们传递的函数将通过上面的init结构被调用，但是这个函数我们还没有定义在应用中。 
5.设置LED输出在给出的led_write_handler状态，向下面这样：

	 Static void led_write_handler(ble_lbs_t * p_lbs, uint8_t led_state)
	{
		if (led_state)
		{
			nrf_gpio_pin_set(LEDBUTTON_LED_PIN_NO);
		}
		else
		{
			nrf_gpio_pin_clear(LEDBUTTON_LED_PIN_NO);
		}
	}

6.最后，增加服务的事件句柄为应用的事件调度：

	static void ble_evt_dispatch(ble_evt_t * p_ble_evt)
	{
		on_ble_evt(p_ble_evt);
		ble_conn_params_on_ble_evt(p_ble_evt);
		ble_lbs_on_ble_evt(&m_lbs, p_ble_evt);
	}

#####4.4.4 测试它
现在可以做个快速的测试了，如29页第五章中“Testing the Application”所示，通过使用主控制器，你应该能打开LED通过向led特征值写‘1’  
#####4.4.5 按键处理  
为了完成应用，你需要定义怎样处理按键按下。使用 app_button模块那是SDK的一部分。这个模块讲给出一个回调对于每次按键按下，本质上是使特征值切换它的状态对于每次按下。
在buttons_init，设置你想使用的按键，这里使用评估套件里的按键1   
1.增加一个新的定义，只是为了使代码更加可读： 

	#define LEDBUTTON_BUTTON_PIN_NO EVAL_BOARD_BUTTON_1

2.为了安装引脚在buttons_init中，增加按键配置数组像下面这样： 

	static app_button_cfg_t buttons[] =
	{
		{WAKEUP_BUTTON_PIN, false, NRF_GPIO_PIN_PULLUP, NULL},
		{LEDBUTTON_BUTTON_PIN_NO, false, NRF_GPIO_PIN_PULLUP, button_event_handler},
	};
		APP_BUTTON_INIT(buttons, sizeof(buttons) / sizeof(buttons[0]),
		BUTTON_DETECTION_DELAY, true);

评估套件上的按键是低电平有效，一次设为false，但是它们没有外部上拉。你将打开内部上拉通过使用NRF_GPIO_PIN_PULLUP。按键唤醒无关紧要，因此句柄设置为NULL，在这之后，你将初始化模块。   
3.在button_event_handler中，发送上次相反的状态通过LED按键服务的API方法。这将确保按键的状态如下图所示的切换在每次按下时：  

	static void button_event_handler(uint8_t pin_no)
	{
		static uint8_t send_push = 1;
		uint32_t err_code;
		switch(pin_no)
		{
			case LEDBUTTON_BUTTON_PIN_NO:
			err_code = ble_lbs_on_button_change(&m_lbs, send_push);
			if (err_code != NRF_SUCCESS &&
				err_code != BLE_ERROR_INVALID_CONN_HANDLE &&
				err_code != NRF_ERROR_INVALID_STATE)
			{
				APP_ERROR_CHECK(err_code);
			}
			send_push = !send_push;
			break;
		default:
			break;
		}
	}


另外定义一个方法，你需要确保按键模块被打开。默认，模板应用建议连接或断开事件，在这使用case也是ok的。取消掉调用app_button_enable() 和 app_button_disable()的定义：

	switch (p_ble_evt->header.evt_id)
	{
		case BLE_GAP_EVT_CONNECTED:
			…
			/* … */
			err_code = app_button_enable();
			break;
		case BLE_GAP_EVT_DISCONNECTED:
			…
			/* … */
			err_code = app_button_disable();
			if (err_code == NRF_SUCCESS)
			{
				advertising_start();
			}
			break;

你现在要准备测试所要求的函数应用了，详细描述在29页第五章中“Testing the Application”。然而，为了能容易的区别服务在中心设备扫描的时候，增加服务的uuid为广播包是非常有用的。 
#####4.4.6 增加服务UUID为广播包 
使用在广播包里的UUID打开中心设备使用这些信息决定它是否将要连接。在上面第6页2.1.2节广播中有提到，一个广播包可以包含31个字节，但是如果需要额外的数据，一个扫描反馈应可以发送。  
你将需要增加自定义的UUID，它是16个字节，针对于扫描包因为在它的广播包里没有地方了。     
广播数据被安装在main.c函数中的advertising_init()函数中，那将设置数据结构并且调用ble_advdata_set()。这个方法携带两个相同类型的参数一个针对于广播包一个针对扫描反馈。你必须增加一个结构体通过扫描反馈参数。  
UUID然后被设置为LBS_UUID_SERVICE,并且类型是ble_lbs_t结构体中uuid_type字段。  
advertising_init()应该向下面：

	static void advertising_init(void)
	{
		uint32_t err_code;
		ble_advdata_t advdata;
		ble_advdata_t scanrsp;
		uint8_t flags = BLE_GAP_ADV_FLAGS_LE_ONLY_LIMITED_DISC_MODE;
		// YOUR_JOB: Use UUIDs for service(s) used in your application.
		ble_uuid_t adv_uuids[] = {{LBS_UUID_SERVICE, m_lbs.uuid_type}};
		// Build and set advertising data
		memset(&advdata, 0, sizeof(advdata));
		advdata.name_type = BLE_ADVDATA_FULL_NAME;
		advdata.include_appearance = true;
		advdata.flags.size = sizeof(flags);
		advdata.flags.p_data = &flags;
		memset(&scanrsp, 0, sizeof(scanrsp));
		scanrsp.uuids_complete.uuid_cnt = sizeof(adv_uuids) / sizeof(adv_uuids[0]);
		scanrsp.uuids_complete.p_uuids = adv_uuids;
		err_code = ble_advdata_set(&advdata, &scanrsp);
		APP_ERROR_CHECK(err_code);
	}

m_lbs结构体中的uuid_type被使用在这儿，确保在main里services_init()被设置，在advertising_init()之前被调用：  

	int main(void)
	{
		…
		services_init();
		advertising_init();
		…

现在应用完全完成了。

###5 测试应用