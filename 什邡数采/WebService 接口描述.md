# 什邡数据采集系统 WebService 接口描述
>## 下载基础信息接口
```c#
/// <summary>
/// 下载基础信息
/// <summary>
/// <param name="equipmentType">设备类型: 0: 不限, 1: RT, 2: MTS, 3: 雪茄吸阻</param>
/// <param name="lastSyncServerTime">最近同步时间, 该参数存在时将只会同步在该时间之后产生了修改的牌号</param>
/// <param name="recentDays">预留工单号相关参数, 获取多少天内的数据, -1 表示全部获取</param>
/// <param name="responseXml">基础信息xml字符串返回</param>
/// <param name="failReason">错误信息返回</param>
/// <return>成功返回True, 失败返回False</return>
bool listBaseInfomation(int equipmentType, string lastSyncServerTime, int recentDays, out string responseXml, out string failReason)
```
#### 接口返回 xml 文件中的除牌号中的测量指标不同外,其他属性都是通用的
#### 牌号中的指标是根据数采系统牌号中的指标生成的
#### 数采系统中添加指标时会要求填写指标别名, 如: 重量( ```weight``` ), 下载牌号时就会根据指标别名生成 ```xml```
#### ```均值```, ```SD``` 值判定标准也可以从牌号中获取
#### ```<lastSyncServerTime />``` 返回当前获取基础信息的时间
```xml
<?xml version="1.0" encoding="GB2312" ?>
<info>
  <specification id="1" name="牌号1" equipmentType="设备类型名称">
    <weight mid="" high="" low="" />
    <circle mid="" high="" low="" position="" />
    <oval mid="" high="" low="" />
    <length mid="" high="" low=""  />
    <resistance mid="" high="" low="" />
    <hardness mid="" high="" low="" position="" />
  </specification>
  <turn id="1" name="班次1" />
  <turn id="2" name="班次2" />
  <turn id="3" name="班次3" />
  <machine id="1" name="机台1" />
  <machine id="2" name="机台2" />
  <machine id="3" name="机台3" />
  <type id="1" name="测量类型1" />
  <type id="2" name="测量类型2" />
  <type id="3" name="测量类型3" />
  <lastSyncServerTime>2022-9-14 13:09:09</lastSyncServerTime>
</info>
```

>## 上传测量数据接口
```c#
/// <summary>
/// 上传测量数据
/// <summary>
/// <param name="equipmentType">设备类型</param>
/// <param name="xmlData">上传xml文件字符串</param>
/// <param name="failReason">错误信息返回</param>
/// <return>成功返回True, 失败返回False</return>
bool uploadMeasureData(int equipmentType, string xmlData, out string failReason)
```
#### 解析 ```<info />``` 节点可以根据 ```machineType``` 匹配不同的解析方法, 方便扩展
#### ```<data />``` 节点中 ```time``` 属性是必须的, 其他属性根据测试台测量项目来定义, 数采系统会解析剩下的属性, 根据 ```属性名称``` 查询 ```指标别名``` 获取 ```指标Id``` 生成 ```json字符串``` 保存到数据库中
#### ```beginTime```、```endTime``` 节点不能为空, ```productionTime```、```deliverTime``` 节点可以为空
#### 当设备类型为 ```雪茄吸阻``` 时, ```turnId```、```machineId``` 可为空
#### ```userData``` 作为冗余属性, ```雪茄吸阻``` 类型数据可将 ```工号``` 使用 ```userData``` 传递
```xml
<?xml version="1.0" encoding="GB2312" ?>
<upload>
  <info beginTime="" endTime="" productionTime="" deliverTime="" specificationId="" turnId="" machineId="" typeId="" instance="" pickupWay="" userData="" temperature="" humidity="">
    <data time="" weight="" circle="" oval="" length="" resistance="" hardness="" />
    <data time="" weight="" circle="" oval="" length="" resistance="" hardness="" />
    <data time="" weight="" circle="" oval="" length="" resistance="" hardness="" />
  </info>
</upload>
```

> ## 上传标定数据接口
```c#
/// <summary>
/// 上传标定数据
/// <summary>
/// <param name="equipmentType">设备类型</param>
/// <param name="xmlData">上传标定数据xml字符串</param>
/// <param name="failReason">返回错误消息</param>
/// <return>返回标定验证记录id集合</return>
List<int> uploadCalibrationData(int equipmentType, string xmlData, out string failReason)
```
#### ```operation``` 包括 ```标定```、```验证```、```内部标定```、```内部验证``` 类型
#### ```unit``` 为单元, ```unitType``` 为单位, 没有单位时可不填写
```xml
<?xml version="1.0" encoding="GB2312" ?>
<upload>
  <info instance="">
    <record id="" time="" operation="" unit="" unitType="" resultCode="" description="" temperature="" humidity="" />
    <record id="" time="" operation="" unit="" unitType="" resultCode="" description="" temperature="" humidity="" />
    <record id="" time="" operation="" unit="" unitType="" resultCode="" description="" temperature="" humidity="" />
  </info>
</upload>
```

> ## 接口调用方式
#### 添加服务引用
```c#
BasicHttpBinding binding = new BasicHttpBinding();
EndpointAddress endpoint = new EndpointAddress("WebService 地址");
// 初始化服务
var service = new DataServiceSoapClient(binding, endpoint);
// 调用接口, 其他接口都可以使用该方式
// 定义请求参数
listBaseInformationRequest request = new listBaseInformationRequest(1, "", 0);
var response = service.listBaseInformationAsync(request).Result;
// 获取响应数据
var result = response.listBaseInfomationResult;
var responseXml = response.responseXml;
var failReason = response.failReason;
```