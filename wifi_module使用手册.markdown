# wifi\_module使用手册

## 概述
wifi\_module旨在封装WifiManager及ConnectivityManager，提供简单可靠的接口，帮助实现Android开发中对网络状态监听，网络连接类型获取，扫描Wifi热点，连入指定Wifi热点，获取当前网络信息等功能。
本模块提供的集成方式为Jar包。
项目目录中包含如下内容：
- Lib：common\_wifi\_module.jar
- Sample:用于展示API的基本用法，包括获取当前网络状态，网络类型，相关信息，扫描和连入Wifi等。
- Doc：主要存放Lib相关接口参考文档,可离线查看index.html获取API的具体说明。


## 功能说明
- Wifi\_module能为使用者提供如下功能：
- 获取当前网络连接状态(有无网络)
- 获取当前网络连接类型
- 网络连接/断开状态监听
- 获取当前连入网络的相关信息
- 扫描附近可用的Wifi网络
- 获取设备已经记录的Wifi网络
- 连接指定Wifi网络
- 打开/关闭Wifi模块
- 断开当前Wifi网络
- 忘记指定Wifi网络

## 适配说明
Android 2.2 及以上的所有系统，API level 8。

## 准备工作
### 添加播放器 SDK 到 App 工程
请参考以下步骤,将播放器 SDK 添加到 App 工程中:
- 创建一个 Android 工程;
- 将common\_wifi\_module.jar添加到 App 工程的 libs 目录下;
- 打开工程“Properties” \> “Java Build Path”\>“Add Jars”;
- 浏览“common\_wifi\_module.jar”完成添加。

### 权限声明
	<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

## NetStateManager使用说明
### 初始化NetStateManager
	NetStateManager(Context context)

### 获取当前网络连接状态
	boolean isNetworkConnected()
	返回值：	@return true:  有已连接的网络
		@return false: 没有已连接的网络

### 获取当前网络可用状态
	boolean isNetworkAvailable()
	返回值：	@return true:  有可用的网络
		@return false: 没有可用的网络

### 获取当前网络连接类型
	NetType getAPNType()
	返回值：	@ NetType.WIFI		当前网络类型为Wi-Fi
		@ NetType.BLUETOOTH	当前网络类型为蓝牙
		@ NetType.MOBILE	当前网络类型为Mobile
		@ NetType.ETHERNET	当前网络类型为以太网
		@ NetType.NONE		当前没有网络连接
		@ NetType.UNKNOWN	当前网络类型未知

## EthernetUtils使用说明
### 获取以太网连接的IP地址
	static String getLocalIPAddress ()
	返回值：	当前已连接的以太网连接的IP地址

### 获取以太网连接的MAC地址
	static String getDevMac ()
	返回值：	当前已连接的以太网连接的MAC地址

## WifiSettingManager使用说明
### 初始化WifiSettingManager
	WifiSettingManager(Context context)

### 设置Wifi模块可用状态
	boolean setWifiEnable(boolean wifiEnable)
	参数：	@true	设置Wi-Fi为可用状态
		@false	设置Wi-Fi为不可用状态
	返回值：	@true	设置成功
		@false	设置失败

### 扫描附近可用Wifi网络
	void search(WifiSearchListener listener) 
	参数：interface WifiSearchListener

接口WifiSearchListener需实现的方法：
	void onSearchWifiFailed(ErrorType errorType)
	方法说明：扫描网络发生错误
	接口数据：扫描错误的类型
		@ SEARCH_WIFI_TIMEOUT	扫描Wi-Fi超时
		@ NO_WIFI_FOUND	没有Wi-Fi网络
与
	void onSearchWifiSuccess(List<AccessPoint> results)
	方法说明：成功返回扫描结果
	接口数据：扫描结果返回AccessPoint列表

代码示例：
	mWifiSettingManager.search(new WifiSearchListener() {
		@Override
		public void onSearchWifiFailed(ErrorType errorType) {
			if (errorType == ErrorType.NO\_WIFI\_FOUND) { 
				Log.e(TAG, "error is NO\_WIFI\_FOUND");
			} else {
				Log.e(TAG, "error is SEARCH\_WIFI\_TIMEOUT");
			}
		}
	
		@Override
		public void onSearchWifiSuccess(List<AccessPoint> aps) 
			mAccessPoints = aps;
			mListView.setAdapter(mAdapter);
		}
	});


### 获取当前设备已经保存的Wifi网络配置信息
	List<AccessPoint> getConfiguredNetworks()
	返回值：当前当前设备已经保存的Wifi网络的列表

### 连入指定网络
	public void connect(AccessPoint accessPoint, boolean saveConfig, WifiConnectListener listener)
	参数说明：accessPoint：	需要包含如下信息:
		ssid 		SSID 
		securityMode 	加密方式 
		password	密码
		userName 	用户名(only in EAP)
		saveConfig：	是否记住网络
		listener：	网络连接是否成功的监听接口

接口WifiConnectListener需要实现的方法
	public void OnWifiConnectCompleted()
	方法说明：指定Wi-Fi连接成功
与
	public void OnWifiConnectFailed()
	方法说明：指定Wi-Fi连接失败

代码示例：
	mWifiSettingManager.connect(accessPoint, false, new WifiConnectListener() {
		@Override
		public void OnWifiConnectCompleted() {
		   handler.sendEmptyMessage(CONNECT_RESULT_SUCCESS);
		}
	
		@Override
		public void OnWifiConnectFailed() {
			handler.sendEmptyMessage(CONNECT_RESULT_FAILED);
		}
	});

由于不同的Wi-Fi网络可能会有不同的加密方式，而不同的加密方式在连接时的会各有差别，wifi\_module虽然可以判断指定Wi-Fi网络的加密类型，并根据类型采用对应的方法进行连接，但在开发过程中，采用不同加密方式的Wi-Fi网络在连接过程中的交互和页面展现也应有相应的差别。Wi-Fi的加密方式差别体现在AccessPoint. securityMode和AccessPoint. securityString中，其中AccessPoint. securityString为加密类型的字符串表达，用于在信息展示时使用，而AccessPoint. securityMode是一个SecurityMode类型的字段，SecurityMode是一个枚举类，包含常见WI-Fi加密类型，它们是——OPEN，WEP，PSK和EAP。

#### OPEN
OPEN代表该Wi-Fi网络没有加密，即没有密码，在连入此类Wi-Fi网络时，不需要用户输入密码，也不需要在connect方法的AccessPoint参数中的password中添加密码信息，也不需要弹出输入密码的对话框。

#### WEP or PSK
这两种都是常见的Wi-Fi网络加密方式，其中PSK还有很多子类，但只是在Wi-Fi网络相关信息展示时才需要区分，连接时wifi\_module可以自动判断类型并根据类别进行配置的相关调整。连接使用这两种加密方式的Wi-Fi网络时，需要弹出对话框由用户输入密码，并且要将密码字符串赋值到connect方法的AccessPoint参数中password字段。

#### EAP
EAP是一种比较少见的Wi-Fi网络加密方式，与WEP和PSK的不同在于，这种方式加密的Wi-Fi网络在连接时不仅需要用户输入密码，还需要用户提供正确的用户名，故当要连接的Wi-Fi网络为EAP加密时，需要弹出对话框由用户输入用户名与密码，并将两者分别赋值到connect方法的AccessPoint参数中userName和password字段。

#### 其他API
	public boolean isWifiEnabled() 
	方法说明：判断当前Wi-Fi是否可用
	返回值：	@true	当前Wi-Fi可用
		@false	当前Wi-Fi不可用
> 
	public static int calculateSignalLevel(int rssi, int numLevels)
	方法说明：计算新号等级
	参数说明：rssi		信号强度
		numLevels	信号分级个数
	返回值：	计算后的信号等级
> 
	public boolean disableNetwork(int netId)
	方法说明：使指定的网络不可用
	参数说明：网络ID
	返回值：	@true	设置成功
		@false	设置失败
> 
	public String getSSID()
	方法说明：获取当前SSID
	返回值：	SSID
> 
	public int getNetworkId()
	方法说明：获取当前netId
	返回值：	当前netId
> 
	public int getIPAddress()
	方法说明：获取已连接Wi-Fi的IP地址
	返回值：	当前已连接Wi-Fi的IP地址
> 
	public String getMacAddress()
	方法说明：获取已连接Wi-Fi的MAC地址
	返回值：	当前已连接Wi-Fi的MAC地址
> 
	public String getBSSID()
	方法说明：获取已连接Wi-Fi的BSSID
	返回值：	当前已连接Wi-Fi的BSSID
> 
	public int getLinkSpeed() 
	方法说明：获取已连接Wi-Fi的连接速度
	返回值：	当前已连接Wi-Fi的连接速度
> 
	public int getRssi() 
	方法说明：获取已连接Wi-Fi的信号强度
	返回值：	当前已连接Wi-Fi的信号强度
> 
	public String getDns1() 
	方法说明：获取已连接Wi-Fi的首选DNS地址
	返回值：	当前已连接Wi-Fi的首选DNS地址
> 
	public String getDns2() 
	方法说明：获取已连接Wi-Fi的备选DNS地址
	返回值：	当前已连接Wi-Fi的备选DNS地址
> 
	public String getGateway()
	方法说明：获取已连接Wi-Fi的网关地址
	返回值：	当前已连接Wi-Fi的网关地址
> 
	public List<AccessPoint> getConfiguredNetworks()
	方法说明：获取已经记住的网络
	返回值：	获取已经记住的网络
> 
	public boolean reassociate()
	方法说明：重新连接Wi-Fi网络，即使该网络是已经被连接上的
	返回值：	@true	连接成功
		@false	连接失败
> 
	public boolean reconnect()
	方法说明：重新连接一个未连接上的WIFI网络
	返回值：	@true	连接成功
		@false	连接失败
> 
	public boolean removeNetwork(int netId) 
	方法说明：移除某一个网络
	参数说明：网络ID
	返回值：	@true	移除成功
		@false	移除失败
> 
	public WifiConfiguration isExsits(String SSID)
	方法说明：查看以前是否也配置过这个网络
	参数说明：SSID 网络名称
	返回值：	如果已经配置过则返回配置config，若未配置过，则返回null
> 
	public static String FormatIP(int IpAddress) 
	方法说明：IP地址转化为字符串格式
	参数说明：转换格式前的IP地址
	返回值：	格式转后后的字符串

