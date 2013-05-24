---
layout: post
title: Core Location Framework学习
categories:
- Mac
tags:
- Mac
- IT
- 定位
- Geo

---

在Apple开发中，尤其是移动设备开发，经常会使用Core Location Framework，这个框架可以使得iOS设备获取当前的地理位置。本文就具体到Core Location 框架中，查看其声明源码。

###CLHeading.h
代表了一个可以通过(x,y,z)三维空间坐标确定磁北极位置的向量。精确的Heading(方位)定位，同时也需要时间信息(即通过空间加时间四维坐标来确定位置)

typedef double CLHeadingComponentValue;

* 代表一个地理磁场数据类型，以[微特斯拉][tesla]为单位，在三维空间确定设备的坐标。

extern const CLLocationDegrees kCLHeadingFilterNone

* CLLocationManager用作为方位信息的过滤标志，表明不需要过滤最小移动量，用户的任何移动都会被通知。

@property(readonly, nonatomic) CLLocationDirection magneticHeading;

* 以度数来代表方向，0度代表磁北极方位，方向是参照设备的上部方向，不考虑设备的摆放方向，也不考虑用户的面对方向。
* 度数的范围为0~359.9

@property(readonly, nonatomic) CLLocationDirection trueHeading;

* 以度数来代表方向，0度表示真正的北极方向，方向是参照设备的上部方向，不考虑设备的摆放方向，也不考虑用户的面对方向。
* 度数的范围是0~359.9
* 众所周知，磁北极方向和地球北极方向是不重合的，所以，这里有必要区分magnetHeading和trueHeading.

@property(readonly, nonatomic) CLLocationDirection headingAccuracy;

* 代表了magnetHeading(磁北极方向)和真正的trueHeading(地理北极)在以度为度量单位上的最大偏差。不能取负值。

@property(readonly, nonatomic) CLHeadingComponentValue x;

@property(readonly, nonatomic) CLHeadingComponentValue y;

@property(readonly, nonatomic) CLHeadingComponentValue z;

@property(readonly, nonatomic) NSDate *timestamp;

* 分别代表了三维空间位置坐标和时间坐标。


###CLLocation.h

代表带有精度和时间信息的地理坐标。

typedef double CLLocationDegrees;

* 以[WGS84(World Geodetic System一1984 Coordinate System)][wgs84]坐标系统为基础，以度为单位，代表了经度或者纬度的类型。
* 度数可以是正的，代表了北方和东方；或者为负值，代表了南方和西方。

typedef double CLLocationAccuracy;

* 用来代表以米为单位的位置精度类型，越低的值，代表越精确的位置信息，不允许负值。

typedef double CLLocationSpeed;

* 代表了设备以m/s为单位的移动速度类型。

typedef double CLLocationDirection;

* 定义了以度为单位的方向类型，取值范围为0~359.9，不允许负值。

{% highlight c %}
typedef struct {
	CLLocationDegrees latitude;
	CLLocationDegrees longitude;
} CLLocationCoordinate2D;

{% endhighlight %}

* 定义了二位的平面坐标系，以经度和纬度为坐标方向。

typedef double CLLocationDistance;

* 定义了以米为单位的距离类型。

extern const CLLocationDistance kCLDistanceFilterNone;

* 定义CLLocationManager中使用的距离过滤器，表明定位服务是否过滤最小移动距离，如果不过滤，用户的任何移动都会被通知。

extern const CLLocationAccuracy kCLLocationAccuracyBestForNavigation;

extern const CLLocationAccuracy kCLLocationAccuracyBest;

extern const CLLocationAccuracy kCLLocationAccuracyNearestTenMeters;

extern const CLLocationAccuracy kCLLocationAccuracyHundredMeters;

extern const CLLocationAccuracy kCLLocationAccuracyKilometer;

extern const CLLocationAccuracy kCLLocationAccuracyThreeKilometers;

* 定义定位精度枚举，更精确级别的定位对移动设备意味着更多的能耗和时间，所以对于应用需要的精度，选择合适的定位精度。

extern const CLLocationDistance CLLocationDistanceMax;

*用来指定设备支持的最长距离。

extern const NSTimeInterval CLTimeIntervalMax;

* 用来指定设备支持的最大时间跨度。

extern const CLLocationCoordinate2D kCLLocationCoordinate2DInvalid;

* 用来指定非法的二维坐标。

BOOL CLLocationCoordinate2DIsValid(CLLocationCoordinate2D coord);

* 用来判断一个二维坐标是否合法。

CLLocationCoordinate2D CLLocationCoordinate2DMake(CLLocationDegrees latitude, CLLocationDegrees longitude);

* 使用经度和纬度值来创建一个二维坐标。

-(id)initWithLatitude:(CLLocationDegrees)latitude longitude:(CLLocationDegrees)longitude;

* 使用经度和纬度来初始化一个CLLocation类型。

-(id)initWithCoordinate:(CLLocationCoordinate2D)coordinate altitude:(CLLocationDistance)altitude horizontalAccuracy:(CLLocationAccuracy)hAccuracy verticalAccuracy:(CLLocationAccuracy)vAccuracy timestamp:(NSDate *)timestamp;

* 使用经度纬度，及在水平和竖直方向的精度、时间戳来初始化 一个CLLocation实例

-(id)initWithCoordinate:(CLLocationCoordinate2D)coordinate
    altitude:(CLLocationDistance)altitude
    horizontalAccuracy:(CLLocationAccuracy)hAccuracy
    verticalAccuracy:(CLLocationAccuracy)vAccuracy
    course:(CLLocationDirection)course
    speed:(CLLocationSpeed)speed
    timestamp:(NSDate *)timestamp;

@property(readonly, nonatomic) CLLocationCoordinate2D coordinate;

* 返回当前的二维位置信息，即经度和纬度。

@property(readonly, nonatomic) CLLocationDistance altitude;

* 返回当前的距离，正值表示海平面以上的距离，负值表示海平面以下的距离

@property(readonly, nonatomic) CLLocationAccuracy horizontalAccuracy;

* 返回当前的水平精度，负值表示当前纬度非法

@property(readonly, nonatomic) CLLocationAccuracy verticalAccuracy;

* 返回当前的垂直精度，负值表示经度非法。

@property(readonly, nonatomic) CLLocationDirection course ;

* 返回相对地理北极的偏度，取值范围在0~359.9，负值非法。

@property(readonly, nonatomic) CLLocationSpeed speed;

* 返回当前设备的移动速度。

@property(readonly, nonatomic) NSDate *timestamp;

* 返回当前位置点的定位时间。

-(CLLocationDistance)distanceFromLocation:(const CLLocation *)location;

* 返回两个位置间的横向位置(应当是欧氏距离吧)。


###CLRegion.h

定义了区域类型，但在现在的版本中，只支持圆形区域(是通过区域中心坐标和区域半径实现)。区域半径以米为单位。

-(id)initCircularRegionWithCenter:(CLLocationCoordinate2D)center radius:(CLLocationDistance)radius identifier:(NSString *)identifier;

* 使用区域原点，区域半径 和区域标志来初始化一个区域实例。

@property (readonly, nonatomic) CLLocationCoordinate2D center ;

@property (readonly, nonatomic) CLLocationDistance radius ;

@property (readonly, nonatomic) NSString *identifier;

* 返回区域信息。


-(BOOL)containsCoordinate:(CLLocationCoordinate2D)coordinate ;

* 判断一个坐标点是否在该区域内部。

###CLPlacemark.h

代表了一个地理位置上的地方信息，这些地区信息，可以是国家，州，城市和街道名称。

-(id)initWithPlacemark:(CLPlacemark *)placemark;
* 使用一个地区信息来初始化。

@property (nonatomic, readonly) CLLocation *location;

* 返回这个地区信息的位置信息。

@property (nonatomic, readonly) CLRegion *region;

* 返回一个地区信息的区域位置

@property (nonatomic, readonly) NSDictionary *addressDictionary;

* 返回一个ABCreateStringWithAddressDictionary表示的地址字典类型。

@property (nonatomic, readonly) NSString *name; // eg. Apple Inc.

@property (nonatomic, readonly) NSString *thoroughfare; // street address, eg. 1 Infinite Loop

@property (nonatomic, readonly) NSString *subThoroughfare; // eg. 1

@property (nonatomic, readonly) NSString *locality; // city, eg. Cupertino

@property (nonatomic, readonly) NSString *subLocality; // neighborhood, common name, eg. Mission District

@property (nonatomic, readonly) NSString *administrativeArea; // state, eg. CA

@property (nonatomic, readonly) NSString *subAdministrativeArea; // county, eg. Santa Clara

@property (nonatomic, readonly) NSString *postalCode; // zip code, eg. 95014

@property (nonatomic, readonly) NSString *ISOcountryCode; // eg. US

@property (nonatomic, readonly) NSString *country; // eg. United States

@property (nonatomic, readonly) NSString *inlandWater; // eg. Lake Tahoe

@property (nonatomic, readonly) NSString *ocean; // eg. Pacific Ocean

@property (nonatomic, readonly) NSArray *areasOfInterest; // eg. Golden Gate Park

* 以上为地址信息。

###CLGeocoder.h
地理位置编码器。

typedef void (^CLGeocodeCompletionHandler)(NSArray *placemarks, NSError *error);

* 定义地理编码处理器。

@property (nonatomic, readonly, getter=isGeocoding) BOOL geocoding;

* 判断是否在进行位置编码。

-(void)reverseGeocodeLocation:(CLLocation *)location completionHandler:(CLGeocodeCompletionHandler)completionHandler;

* 翻转解码请求。地理解码是通过地址信息，解析出定位坐标，而这里是通过定位坐标解析出placemark。 

-(void)geocodeAddressDictionary:(NSDictionary *)addressDictionary completionHandler:(CLGeocodeCompletionHandler)completionHandler;
-(void)geocodeAddressString:(NSString *)addressString completionHandler:(CLGeocodeCompletionHandler)completionHandler ;
-(void)geocodeAddressString:(NSString *)addressString inRegion:(CLRegion *)region completionHandler:(CLGeocodeCompletionHandler)completionHandler; 

* 通过地址信息，和区域位置解析出定位坐标。

-(void)cancelGeocode;

* 取消当前解码请求。

###CLLocationManager.h

位置管理器，用来接入定位服务。可以参照[CLLocationManager Class Reference][location-class-ref]来了解该类的功能定义。

+(BOOL)locationServicesEnabled ;
* 判断用户是否启动位置服务。
*  在启动位置更新操作之前，用户应当检查该方法的返回值来查看设备的位置服务是否启动。如果位置服务没有启动，而用户又启动了位置更新操作，那么Core Location 框架将会弹出一个让用户确认是否启动位置服务的对话框。

+(BOOL)headingAvailable;

* 如果设备支持方位服务(即设备能否返回方位数据)则返回YES，否则返回NO。

+(BOOL)significantLocationChangeMonitoringAvailable;

* 表明设备能否报告基于significant location changges的更新(significant location change监控，只是基于设备所链接的蜂窝塔的位置改变诊断，在精度要求不高的情况下，可以节省很多电量。)

+(BOOL)regionMonitoringAvailable;

* 表明设备是否支持区域监测，即使设备支持区域监测，用户也可能把当前应用设置为区域监测不可用，在启动或者初始化区域监测时应当检查。


+(CLAuthorizationStatus)authorizationStatus ;

* 检查应用的授权状态，应用在第一次启动时，会自动请求授权，应用应当明确被授权使用位置服务，并且位置服务当前出于运行状态，应用才能使用位置服务。

@property(assign, nonatomic) id<CLLocationManagerDelegate> delegate;

* 把位置管理的一些活动代理出去。

@property(assign, nonatomic) CLActivityType activityType ;

* 活动类型指明框架是否可以自动停止位置更新，这样当用户的位置很长时间没有改变时，系统可以暂停位置更新，来节省电能。

@property(assign, nonatomic) CLLocationDistance distanceFilter;

* 距离过滤，通过前后两次的位置信息来决定距离长度，当距离长度没有达到当前设定的distanceFilter时，不触发位置更新事件。


@property(assign, nonatomic) CLLocationAccuracy desiredAccuracy;

* 所需要的精度，应当根据应用场景来设置当前的最佳定位精度，更高的精度意味着在定位时将会消耗更多的能量和时间。定位服务可能不支持所设定的精度，但定位服务将会最大可能的满足所设定的精度。

@property(assign, nonatomic) BOOL pausesLocationUpdatesAutomatically ;

* 是否可以自动暂停位置更新操作。

@property(readonly, nonatomic) CLLocation *location;

* 上次接收到的位置信息。

@property(assign, nonatomic) CLLocationDegrees headingFilter ;

* 指定产生heading事件时的最小角度改变量。默认的最小headingFilter值为1度。

@property(assign, nonatomic) CLDeviceOrientation headingOrientation ;

* 指定设备的方位朝向参照方向，默认是参照设备的向上方向，但如果应用的布局方向在其他方向上，那么通过设置headingOrientation可以指定heading的参照方向。

@property(readonly, nonatomic) CLHeading *heading;

*返回最近一次的方位信息。

@property (readonly, nonatomic) CLLocationDistance maximumRegionMonitoringDistance;

* 检测区域的最大范围。这个值可能依赖于设备的硬件特性，当给出比该值更大的区域范围时，将会产生kCLErrorRegionMonitoringFailure


@property (readonly, nonatomic) NSSet *monitoredRegions;

* 当前正在监测的区域集合。不能直接把监测区域添加到该集合中，应当使用startMonitoringForRegion:desiredAccuracy:方法来增加想要监测的区域。

-(void)startUpdatingLocation;

-(void)stopUpdatingLocation;

-(void)startUpdatingHeading ;

-(void)stopUpdatingHeading ;

-(void)dismissHeadingCalibrationDisplay ;

-(void)startMonitoringSignificantLocationChanges ;

-(void)stopMonitoringSignificantLocationChanges;

* 以上方法为开启或者停止相应的服务。

		
-(void)stopMonitoringForRegion:(CLRegion *)region ;

-(void)startMonitoringForRegion:(CLRegion *)region ;

-(void)startMonitoringForRegion:(CLRegion *)region desiredAccuracy:(CLLocationAccuracy)accuracy ;

* 以指定的精度添加需要监测的区域，应用将要自己负责检测这些区域边界是否有重合的部分，来防止当用户在使用这些重合区域时而发出的重复的通知。如果有相同标记的区域已经出于被监控的状态，则从监控列表中移除该区域。操作的异步性可能导致区域不能立即处于被监控状态。

-(void)allowDeferredLocationUpdatesUntilTraveled:(CLLocationDistance)distance timeout:(NSTimeInterval)timeout ;

* 设定应用的位置更新处理的延迟时间，这样可以使设备处于低能耗状态。如果应用是在前台运行，LocationManagerDelegate仍会持续的接收到位置更新的请求，但是应用在后台处理时，则会使用此处设置的延迟时间，以节省设备的电能。


-(void)disallowDeferredLocationUpdates;

* 如果之前有设置enabled 那么则取消延迟位置更新的允许。

+(BOOL)deferredLocationUpdatesAvailable ;

* 表明设备是否支持延迟更新。

###CLLocationManagerDelegate.h


-(void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations ;

* 告知代理，新位置已经可以使用了，尽管这个方法实现在协议中是可选的，但最好实现它。

-(void)locationManager:(CLLocationManager *)manager didUpdateHeading:(CLHeading *)newHeading ;

*告知代理，方位信息已经被位置管理器更新，如果位置管理器有使用startUpdateingHeading这个方法，则应当实现本方法。


-(BOOL)locationManagerShouldDisplayHeadingCalibration:(CLLocationManager *)manager;

* 询问代理是否方位校准警告是否需要显示，当有新的Heading信息时调用该方法，如否有需要显示Heading校准信息，则返回YES

-(void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region ;

-(void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region ;

-(void)locationManager:(CLLocationManager *)manager
	didFailWithError:(NSError *)error;
	
-(void)locationManager:(CLLocationManager *)manager monitoringDidFailForRegion:(CLRegion *)region withError:(NSError *)error;                                             

-(void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status;

-(void)locationManager:(CLLocationManager *)manager didStartMonitoringForRegion:(CLRegion *)region ;

-(void)locationManagerDidPauseLocationUpdates:(CLLocationManager *)manager ;

-(void)locationManagerDidResumeLocationUpdates:(CLLocationManager *)manager ;

-(void)locationManager:(CLLocationManager *)manager didFinishDeferredUpdatesWithError:(NSError *)error ;

* 告知代理，以上事件有发生，以便做相应处理。


###使用LocationManager

{% highlight c %}
//Standard Location Service
//----------------------
Create a new location manager
locationManager = [[CLLocationManager alloc] init];

// Set Location Manager delegate
[locationManager setDelegate:self];

// Set location accuracy levels
[locationManager setDesiredAccuracy:kCLLocationAccuracyKilometer];

// Update again when a user moves distance in meters
[locationManager setDistanceFilter:500];

// Configure permission dialog
[locationManager setPurpose:@"My Custom Purpose Message..."];

// Start updating location
[locationManager startUpdatingLocation];

{% endhighlight %}

更多如何使用CLLocationManager指导，请参见[iOS 5 Core Frameworks: Core Location and Map Kit][core-location-and-map-kit],这里就不做详细介绍。



------

[tesla]: https://zh.wikipedia.org/wiki/特斯拉
[wgs84]: http://baike.baidu.cn/view/740401.htm
[location-class-ref]: http://developer.apple.com/library/ios/#documentation/CoreLocation/Reference/CLLocationManager_Class/CLLocationManager/CLLocationManager.html
[core-location-and-map-kit]: http://www.peachpit.com/articles/article.aspx?p=1830485