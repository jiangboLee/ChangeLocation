# ChangeLocation
改变自己的定位地址

![迟到.jpeg](http://upload-images.jianshu.io/upload_images/2868618-af39967557889e2e.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

大冬天的起床多不容易。现在公司都是钉钉打卡，微信打卡。老婆一直要我搞一个可以随时打卡的钉钉，我说那要越狱啊，现在咱手机都是11.0以上系统，没有完美越狱啊！！！然后今天发给我一个链接[iOS 模拟定位，让你在家钉钉打卡！](https://www.jianshu.com/p/38782cb3f688),然后照着大神自己写了下demo，回家装吧~
## 地图坐标系
在地图开发中：我们会接触到3中类型的地图坐标系：
1. WGS－84原始坐标系，一般用国际GPS纪录仪记录下来的经纬度，通过GPS定位拿到的原始经纬度，Google和高德地图定位的的经纬度（国外）都是基于WGS－84坐标系的；但是在国内是不允许直接用WGS84坐标系标注的，必须经过加密后才能使用；
2. GCJ－02坐标系，又名“火星坐标系”，是我国国测局独创的坐标体系，由WGS－84加密而成，在国内，必须至少使用GCJ－02坐标系，或者使用在GCJ－02加密后再进行加密的坐标系，如百度坐标系。高德和Google在国内都是使用GCJ－02坐标系，可以说，GCJ－02是国内最广泛使用的坐标系；
3. 百度坐标系:bd-09，百度坐标系是在GCJ－02坐标系的基础上再次加密偏移后形成的坐标系，只适用于百度地图。(目前百度API提供了从其它坐标系转换为百度坐标系的API，但却没有从百度坐标系转为其他坐标系的API)

而我们iOS端采用的定位坐标系是WGS-84.
- 我们可以直接通过定位代码拿到当前位置的经纬度。
```objective-c
 - (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    
    _manager = [[CLLocationManager alloc] init];
    _manager.delegate = self;
//    [manager requestAlwaysAuthorization];
    [_manager requestWhenInUseAuthorization];
    _manager.desiredAccuracy = kCLLocationAccuracyBest;
    _manager.distanceFilter = 1.0;
    [_manager startUpdatingLocation];
}

- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray<CLLocation *> *)locations {
    
    CLLocation *currentLocation = [locations lastObject];
    //1.如果打卡地点是这里，当前的经纬度。
    //2.直接copy下面的经纬度，无需转换
    //3.去Location1.gpx修改经纬度
    //当前的经纬度
    NSLog(@"当前的经纬度 %f,%f",currentLocation.coordinate.latitude,currentLocation.coordinate.longitude);
    self.location2DLable.text = [NSString stringWithFormat:@"纬度:%f,经度:%f",currentLocation.coordinate.latitude,currentLocation.coordinate.longitude];
    
    //地理反编码 可以根据坐标(经纬度)确定位置信息(街道 门牌等)
    CLGeocoder *geoCoder = [[CLGeocoder alloc]init];
    [geoCoder reverseGeocodeLocation:currentLocation completionHandler:^(NSArray<CLPlacemark *> * _Nullable placemarks, NSError * _Nullable error) {
        if (placemarks.count >0) {
            CLPlacemark *placeMark = placemarks[0];
            NSString *currentCity = placeMark.locality;
            if (!currentCity) {
                currentCity = @"无法定位当前城市";
            }
            //看需求定义一个全局变量来接收赋值
            NSLog(@"当前国家 - %@",placeMark.country);//当前国家
            NSLog(@"当前城市 - %@",currentCity);//当前城市
            NSLog(@"当前位置 - %@",placeMark.subLocality);//当前位置
            NSLog(@"当前街道 - %@",placeMark.thoroughfare);//当前街道
            NSLog(@"具体地址 - %@",placeMark.name);//具体地址
            self.locationLable.text = [NSString stringWithFormat:@"%@,%@,%@,%@,%@",placeMark.country,currentCity,placeMark.subLocality,placeMark.thoroughfare,placeMark.name];
        }
    }];
}
```
- 我们要拿到其他位置的坐标点，需要定位到其他位置。
这个时候我们可以先通过[高德地图](http://lbs.amap.com/console/show/picker)得到高德的坐标系，然后通过代码转换成WGS 坐标系。
```objective-c
//如果要想知道任意位置的坐标
    //1.去高德地图http://lbs.amap.com/console/show/picker，选中自己坐标
    //楼主在上海哦~
    //杭州  120.211963,30.274602 
    //2.在进行坐标转换
    CLLocationCoordinate2D location2D = CLLocationCoordinate2DMake(30.274602, 120.211963);
    CLLocationCoordinate2D WGSlocation2D = [ChangeLoction gcj02ToWgs84:location2D];
    NSLog(@"纬度：%f,经度：%f",WGSlocation2D.latitude , WGSlocation2D.longitude);
    //纬度：30.277029,经度：120.207428
    //3.去Location1.gpx修改经纬度
```
其中一大堆坐标系转来转去的方法在项目代码中，有兴趣可以下载看看
## 新建项目
现在我们已经能获得到定位的坐标系了。现在开始调试。可以新建一个项目，新建一个gpx文件。gpx是一种用于存储坐标数据的 XML 文件格式。它可以储存在一条路上的路点，轨迹，路线，且易于处理和转换到其他格式。OpenStreetMap 使用的所有 GPS 数据要转换为 GPX 格式才能上传。
![gpx.png](http://upload-images.jianshu.io/upload_images/2868618-b4069da23ccbb720.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后将自己要定位的经纬度写入文件中。
![修改经纬度.png](http://upload-images.jianshu.io/upload_images/2868618-5235f249332aea00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好的。项目完成了！就这么简单
## 开始调试
##### 必须连接上真机。然后运行。![Snip20180126_44.png](http://upload-images.jianshu.io/upload_images/2868618-6dc6e6576ac8e424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
运行后，点击Debug--->Simulate Location --> Location1（这个是你创建gpx的文件名哦）
这时你会发现手机当前的定位已经切换到你的经纬度。
**这个时候，你打开手机的其他软件，包括自带的地图，微信，钉钉。。。你会发现所有的当前定位都是你设置的位置了。然后，你想怎么弄就怎么弄咯**
你要修改另一个位置，只需要重新修改gpx中的参数，然后重新运行，发现还是上次的位置。这个时候只需要再点Location1，就可以更新到最新位置。
## 结束语
现在我的定位是自定义的。如果要恢复原来的怎么办？第一感觉，我选了Don't Simulate Location,发现当时没反应（原来只需要重新运行一遍就好（iphone6 10.3版本）），但后面发现来回操作多次，原生地图的定位一直在刷新（干脆也重启手机吧）。你以为结束了吗？发现iPhone X（11.2）竟然按上面操作没反应，改不回来了，应用杀掉也不行（没办法，只好重新手机OK了）。

#### iOS程序猿又有人要啦~ 赶快找你们的iOS程序猿给你们装一个公司的定位吧~哈哈。。。。妈妈再也不怕我上班迟到了！
