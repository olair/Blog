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

需要了解，在设置界面或者下拉菜单中的GPS按钮，仅仅是一个GPS的总开关而已，打开它GPS并不会开始定位，只是打开了GPS的访问权限，使具有权限的APP可以去请求GPS进行定位，真正GPS开始工作以及停止工作需要应用程序去控制。

国测局坐标(火星坐标,GCJ02)。国内出版的各种地图系统(包括电子形式),必须至少采用GCJ-02对地理位置进行首次加密，而在国内正常销售、使用的GPS芯片获取的定位信息也必须通过GCJ-02进行处理，处理成加密后的坐标。这将导致国外未采用GCJ02坐标系统的地图在采用了GCJ坐标系统的GPS上不能正常使用，反之亦然。

LocationManager中包含有几种定位模式，NETWORK_PROVIDER、GPS_PROVIDER、PASSIVE_PROVIDER这几种(API 28中又提供了一种WIFI模式)，但在当前常见手机中可用的只有两个GPS_PROVIDER已经PASSIVE_PROVIDER。需要注意的是带GPS的Android设备一般肯定支持GPS，但是可以访问网络的手机却并不一定提供NETWORK_PROVIDER的支持(虽然系统可能不支持网络定位，但我们的程序却可以自己去实现基站定位)。

不要相信国内系统的LocationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER))的返回值，国内被改过的系统中isProviderEnabled用于判断GPS还是可以的，判断其他定位方式就算了。

* [一篇关于GPS定位写得最详实清晰的文章之一](https://blog.csdn.net/zhangbijun1230/article/details/80958036)
* [GPS的一些浅显知识兼介绍一下GPS测试仪](https://blog.csdn.net/shanghaibao123/article/details/48520235)

## GPS定位相关方法(以API 23及以下版本为例)

Android GPS定位主要通过LocationManager类来实现，其中提供了很多方法，大体上可以分为这么几类方法：

### Provider相关方法

* createProvider
* getAllProviders
* getProviders
* getProvider
* getBestProvider

这几个方法用于获取位置提供商，但是在国内没有实际用处，只需要GPS即可。

### 请求定位相关方法(只看方法名，忽略具体参数)

* requestLocationUpdates (开始GPS定位，GPS开始工作)
* requestSingleUpdate (开始GPS定位，GPS开始工作)
* removeUpdates (停止GPS定位，GPS有可能会停止工作，因为有可能其他的APP也启用了GPS)

这几个方法开始请求GPS定位，用于获取位置信息。

### 添加GPS状态监听

* addGpsStatusListener
* removeGpsStatusListener

GPS定位过程中，卫星会处于不断变化中，这几个方法可以注册卫星状态监听器用于实时监测卫星状态的变化。

## GPS定位步骤

### Android中 GPS定位极为简单，首先拿到LocationManager对象：

`LocationManager mLocationManager = (LocationManager) context.getSystemService(LOCATION_SERVICE);`

### 创建定位成功的监听器

```java
LocationListner listener = new LocationListener() {

    @Override
    public void onLocationChanged(Location location) {
        // 收到位置信息(onLocationChanged 不要纠结于方法名，下面会明白)
    }

    @Override
    public void onStatusChanged(String provider, int status, Bundle extras) {
        // 定位提供程序状态发生改变(国内只考虑GPS，而GPS一般不会回调这个方法)
    }

    @Override
    public void onProviderEnabled(String provider) {
        // GPS打开
    }

    @Override
    public void onProviderDisabled(String provider) {
        // GPS 关闭
    }
  
};
```

### 将定位监听器注册到LocationManager上去

上面提到 *请求定位相关方法*，这些以request开头的方法，可以将我们写好的定位监听注册到系统的定位进程上(其实是注册在了你的LocationManager对象上，而LocationManager对象中一个内部类对象注册在系统进程上，IPC相关)，系统定位进程收到定位请求开始根据具体参数执行定位请求，当系统进程成功获取到定位信息后回去遍历通知所有注册过来的位置监听器(LocationListener)。当然，如果你的设备只支持GPS，你定位请求使用NETWORK_PROVIDER正常情况下是不会通知你的位置监听器的。具体看下request相关方法：

如在 *请求定位相关方法* 中看到的，定位相关方法主要有两类，分别是requestLocationUpdates、requestSingleUpdate。其中requestLocationUpdates方法用于表达不断地获取位置信息，具体的看其中一个

`public void requestLocationUpdates(String provider, long minTime, float minDistance,LocationListener listener)`

该方法第一个参数表示定位设备类型(国内手机直接使用GPS_PROVIDER)，第二个表示多长时间才回调一次LocationListener，第三个参数表示距离上一次定位位置超过多远才回调一次LocationListener。

需要注意的是，上面的方法看似很完美，其实却存在很大缺陷，原因有三：

* GPS定位是一个缓慢的过程。GPS冷启动一般需要40s左右才能收到定位回调，热启动定位还算较快一般数秒内可以定位成功。
* 而且在室内没有GPS信号，意味着在室内是不可能通过GPS定位成功的
* 即使是在室外，也受到你所处的环境的限制，空旷的地方定位速度比高楼林立的地方要准确很多快很多。

而你注册的LocationListener只有等到你定位成功的时候他的`onLocationChange(Location )`方法才会被回调。也就是说，很多时候，你在`requestLocationUpdates`中设置了minTime、minDistance会发现实际效果并不好，这个时候不要奇怪很正常。

你会发现，在重载的几个`requestLocationUpdates`方法中，有时候会有一个Criteria或者Looper对象，这两个对象都是给开发者提供一些便利的工具。

* 其中Criteria对象是帮助你在不同Android设备上选择不同的定位设备，通过Criteria中参数的配置，会选择一个最适合的定位设备进行定位。但是在国内就没必要了，直接用GPS就行，你没有其他选择。
* Looper对象是帮助你在你指定的线程中回调你注册的LocationListener对象中的各个方法。

request相关方法中还有一类requestSingleUpdate，该类requestSingle方法表示进行单次定位，即定位成功以后便会自动解绑掉注册在系统进程的LocationManager。

> 需要注意的是，不管是上面requestLocationUpdates方法或者是requestSingleUpdate方法，从该方法被调用开始直至成功获取到定位信息，这段时间内GPS芯片将处于活跃状态，而活跃的GPS芯片会影响到系统休眠，导致系统不能进入休眠状态，造成耗电。具体点说，调用requestLocationUpdates将导致GPS一直处于活跃状态，而调用requestSingleUpdate方法会导致第一次回调onLocationChange之前GPS芯片一直处于活跃状态。所以，在某些状况下设备必须具备主动停掉GPS的机制，当然这种机制要使用LocationManager的removeUpdates去实现。

## 卫星状态监听，并根据卫星状态判断是否继续定位

上面提到，为了节省Android设备的电量，启动GPS定位后某些情况下必须要关掉GPS。如果GPS正常定位，数秒内定位成功，这种情况下很好处理，我们采用requestSingleUpdate方法实现单次定位即可，但是如果处于室内，没有GPS信号的情况下我们将长时间处于定位状态，电池电量将很快耗尽。而对于requestLocationUpdates方法更是需要在某种情况下去停掉GPS。

我们最重要的任务就是要找到关掉GPS的时机。基于上面分析可以找到这个时机，就是，当我们确定GPS在这种情况下永远都不能成功定位时，就需要关闭掉GPS。但是怎么确认GPS永远也定不到位呢？这就需要去通过GPS芯片收到的GPS卫星信号去做出判断。于是就引出了我们对于GPS卫星状态的监听。

### 创建卫星状态监听器

```java
GpsStatus.Listener gpsStatusListener = new GpsStatus.Listener() {
    @Override
    public void onGpsStatusChanged(int event) {
        // GPS 用于报告GPS的状态变化，包含以下四种状态：
        // GpsStatus.GPS_EVENT_STARTED      开始GPS定位，表示GPS开始定位，注意和LocationListener中的onProviderEnabled进行区分
        // GpsStatus.GPS_EVENT_STOPPED      停止GPS定位，注意和LocationListener中的onProviderDisabled进行区分
        // GpsStatus.GPS_EVENT_FIRST_FIX    表示GPS自启动以来首次定位成功
        // GpsStatus.GPS_EVENT_SATELLITE_STATUS GPS卫星状态改变(定位过程中GPS的卫星状态一直处于变化中的，该方法会不断地进行回调)
    }
};
```

### 监听卫星状态

```java
GpsStatus.Listener gpsStatusListener = new GpsStatus.Listener() {
    @Override
    public void onGpsStatusChanged(int event) {
        if (event == GpsStatus.GPS_EVENT_SATELLITE_STATUS) {
            // 需要通过LocationManager拿到具体的卫星状态
            GpsStatus gpsStatus = locationManager.getGpsStatus(null);
            // 然后获取卫星状态的迭代器
            Iterator<GpsSatellite> iterator = gpsStatus.getSatellites().iterator();
            while (iterator.hasNext()) {
                GpsSatellite satellite = iterator.next();
                // 该方法获取到一个卫星的信噪比(信号噪声比)，信噪比越大表示信号越强。
                // 通过GpsSatellite对象你可以获取到很多信息，可以直接查看GpsSatellite对象。
                float nr = satellite.getSnr();
            }
        }
    }
};
```

有了卫星的信噪比就可以根据信噪比的好坏去判断是否能定位成功了，正常情况下信噪比能达到30就达到了及格线，但是需要有三颗以上卫星同时及格才可以定位成功。但是测试发现，信噪比在27左右的时候也是能定位成功的只是定位时间较长。还有一点需要注意，你或许会奇怪GPS卫星明明只有14颗，为什么我收到的卫星个数甚至多于24，搞硬件的告诉我的答案是，现在的GPS天线、芯片不仅仅回去检查GPS卫星获取定位，而是直接集成了几种定位系统，随便观测到什么只要达标就能定位成功(听着意思是各卫星之间也是相互兼容的，不需要纠结，可以确认的是达标后确实能定位成功)。而且你回发现其中有一个值最大，一般是位于你头顶的那颗，如果在你头顶没有遮盖物的情况下位于你头顶的那颗信号都不行，那就信号真的很差了。

> 可以定位的标准为，三颗以上信噪比达到25以上。

### 基于卫星状态的定位实现

于是，我们在GpsStatus.Listener中去判断卫星状态，判断达到25以上的卫星是否有3颗，有的话就进行定位，否则停止定位。代码很简单如下：

```java
// 1. 拿到LocationManager对象
locationManager = (LocationManager) getSystemService(LOCATION_SERVICE);

// 2. 创建定位监听
locationListener = new LocationListener() {
    @Override
    public void onLocationChanged(Location location) {
        // 定位成功回调
    }
    ...(省略其他三个方法)
};

// 3. 创建卫星状态监听
GpsStatus.Listener gpsStatusListener = new GpsStatus.Listener() {
    @Override
    public void onGpsStatusChanged(int event) {
        if (event == GpsStatus.GPS_EVENT_SATELLITE_STATUS) {
            // 计算达标卫星个数
            int validSatelliteCount = 0;
            GpsStatus gpsStatus = locationManager.getGpsStatus(null);
            Iterator<GpsSatellite> iterator = gpsStatus.getSatellites().iterator();
            while (iterator.hasNext()) {
                GpsSatellite satellite = iterator.next();
                float nr = satellite.getSnr();
                if (nr > 25) {
                    validSatelliteCount++;
                }
            }
            if (validSatelliteCount < 3) {
                // 不达标取消定位
                locationManager.removeUpdates(locationListener);
            }
        }
    }
};

// 添加卫星监听开始定位
locationManager.addGpsStatusListener(gpsStatusListener);
locationManager.requestSingleUpdate(LocationManager.GPS_PROVIDER, locationListener, getMainLooper());
```

## GPS定位方案改进版

梳理了GPS的定位流程以及相关要点，实现了一个简单版的GPS定位实现。但是这样依然不完美，GPS定位方案依然需要改进。
假设GPS需要每五分钟定位一次，使用requestLocationUpdates方法时间参数设置为5min对于普通移动设备来说显然不合适，定位失败的情况下电量消耗会过大。所以呢，我们定位方案改进如下：

考虑GPS热启动的话一般5s内可以定位成功(GPS 芯片特别差就另当别论)，所以我们为GPS定位过程加上一个超时时间5s，GPS定位开始后，5s内定位成功就成功返回并关掉GPS定位，出现失败的时候，我们应该去计算刚才5s内每次卫星状态改变时是否都达到了定位标准。如果5s内有30%以上的概率达到了定位标准很可能GPS处于冷启动状态，或者GPS设备偶尔处于有遮盖物的地方，则延长定位时间直至定位成功或者卫星状态达标概率低于30%。