# GPS详解

GPS本身并不复杂，但是因为GPS本身定位比网络还慢的原因用好GPS还是需要费点事的。

## GPS相关类说明(android.location包)

### 主要必须涉及到的类

* LocationManager   用于发起定位请求
* LocationListener  用于监听定位信息的有关更新(包括位置变化、相关定位设备状态改变或用户打开关闭相关定位设备)
* Location          定位信息
* Criteria          帮助开发者选取最好用的定位设备

采用GPS定位时往往需要监听卫星状态，根据卫星状态去决定是否继续采用GPS定位还是采用其他替补定位方式。在android的location包中同样提供了相关功能，但是在API 24以上定位相关包结构有所更改。

### API 23及以下版本

* GpsStatus     所有搜索到的卫星状态信息，其中有一个GpsStatus.Listener用于监听卫星状态的改变。
* GpsSatellite  单个的卫星信息，包括卫星的方位、高度、伪随机噪声码、信噪比等信息，具体的可进入源码查看。

### API 24及以上版本

* GnssStatus    相当于将GpsStatus以及GpsSatellite整合在了一起

API 24只是对定位接口进行了一些接口上的改进，除了使用起来更加方便以外没有什么其他优势。

### 不需要用到的类

* Address、Geocoder、GnssClock、GnssMeasurement、GnssMeasurementsEvent、GnssNavigationMessage、SettingInjectorService、LocationProvider这些class 用于通过经纬度获取地理信息以及自定义一些定位实现等，只有定位需求的问题并不需要这些。

## GPS相关基础知识说明

国测局坐标(火星坐标,GCJ02)。国内出版的各种地图系统(包括电子形式),必须至少采用GCJ-02对地理位置进行首次加密，而在国内正常销售、使用的GPS芯片获取的定位信息也必须通过GCJ-02进行处理，处理成加密后的坐标。这将导致国外未采用GCJ02坐标系统的地图在采用了GCJ坐标系统的GPS上不能正常使用，反之亦然。

LocationManager中包含有几种定位模式，NETWORK_PROVIDER、GPS_PROVIDER、PASSIVE_PROVIDER这几种(API 28中又提供了一种WIFI模式)，但在当前常见手机中可用的只有两个GPS_PROVIDER已经PASSIVE_PROVIDER。需要注意的是带GPS的Android设备一般肯定支持GPS，但是可以访问网络的手机却并不一定提供NETWORK_PROVIDER的支持(虽然系统可能不支持网络定位，但我们的程序却可以自己去实现基站定位)。

不要相信国内系统的LocationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER))的返回值，国内被改过的系统中isProviderEnabled用于判断GPS还是可以的，判断其他定位方式就算了。

* [一篇关于GPS定位写得最详实清晰的文章之一](https://blog.csdn.net/zhangbijun1230/article/details/80958036)
* [GPS的一些浅显知识兼介绍一下GPS测试仪](https://blog.csdn.net/shanghaibao123/article/details/48520235)

## GPS定位实现(以API 23及以下版本为例)

Android GPS定位主要通过LocationManager类来实现，其中提供了很多方法，大体上可以分为这么几类方法：

### Provider相关方法

* createProvider
* getAllProviders
* getProviders
* getProvider
* getBestProvider

在国内没有实际用处，只需要GPS即可。

### 请求定位相关方法

* requestLocationUpdates
* requestSingleUpdate
* requestLocationUpdates

