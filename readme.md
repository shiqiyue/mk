# 企业
## 企业录入
### 企业录入时序图 
```mermaid
sequenceDiagram
autonumber
participant api as  企业录入接口
participant verify as  请求校验器
participant entSrv as  EnterprisentService(企业 Service)
participant entAudSrv as EnterpriseAuditService(企业审核 Service)
participant loginId as loginIdService(企业登录ID Service)
participant ukey as UkeyService(企业Ukey Service)
participant entStateHisSrv as EnterpriseStateHisService(企业状态历史 Service)
participant mq as 消息队列
activate api
api->>verify: 请求校验参数
activate verify
verify->>verify: 校验参数格式
opt 参数格式错误
verify-->>api: 返回参数格式错误
end
verify->>entSrv: 请求返回企业名称是否存在
activate entSrv
entSrv->>entSrv: 通过企业名称查询数据库
entSrv-->>verify: 返回企业名称是否存在
deactivate entSrv
opt 企业名称存在
verify-->>api: 返回企业名称存在
end
verify-->>api: 返回校验通过
deactivate verify

opt 校验失败
api->>api:返回错误信息，退出
end

api->>entSrv: 请求录入企业信息
activate entSrv
entSrv->>entSrv: 插入企业信息到数据库
entSrv->>entAudSrv: 请求录入企业审核信息
activate entAudSrv
entAudSrv->>entAudSrv: 根据企业类型选择不同的审核方式(无需审核，单次审核，并行审核等)

alt 单次审核
entAudSrv->>entAudSrv: 插入一条企业待审核记录到数据库
entAudSrv-->>entSrv: 返回待审核状态
entAudSrv->mq: 发送存在待审核记录的消息到mq
else 并行审核
entAudSrv->>entAudSrv: 插入多条企业待审核记录到数据库，并指定审核部门
entAudSrv-->>entSrv: 返回待审核状态
entAudSrv->mq: 发送存在待审核记录的消息到mq
else 无需审核
entAudSrv-->>entSrv: 返回审核通过状态
end

deactivate entAudSrv
entSrv->>entSrv: 修改企业审核状态


entSrv->>entStateHisSrv: 请求添加企业状态记录
activate entStateHisSrv
entStateHisSrv->>entStateHisSrv:添加企业状态记录
entStateHisSrv-->>entSrv:返回是否添加成功
deactivate entStateHisSrv


entSrv->>loginId: 请求为企业创建登陆ID
activate loginId
loginId->>loginId:创建登陆ID
loginId-->>entSrv:返回登录ID
deactivate loginId
entSrv->>ukey: 请求为企业绑定ukey
activate ukey
ukey->>ukey:绑定ukey
ukey-->>entSrv:返回绑定结果
deactivate ukey
entSrv-->>api: 返回录入结果
deactivate entSrv
deactivate api
```
### 企业录入审核通过时序图
```mermaid
sequenceDiagram
autonumber
participant api as 企业审核通过接口
participant verify as 请求校验器
participant entAuditSrv as EnterpriseAuditService(企业审核 Service)
participant entSrv as EnterpriseService(企业 Service)
activate api
api->>verify: 请求校验参数
activate verify
verify->>verify: 校验参数格式
opt 参数格式错误
verify-->>api: 返回参数格式错误
end

verify-->>api: 返回校验成功
deactivate verify

opt 校验失败
api->>api:返回错误信息，退出
end
activate entAuditSrv
api->>entAuditSrv: 请求审核通过
entAuditSrv->>entAuditSrv: 修改审核记录状态为审核通过
entAuditSrv->>entAuditSrv: 查询当前审核企业是否还有其他未通过审核的记录
opt 存在未通过审核的记录
entAuditSrv-->>api: 返回审核成功
end
entAuditSrv->>entSrv: 请求修改企业状态
activate entSrv
opt 企业状态不为待审核
entSrv-->>entAuditSrv: 返回无需修改状态
end
entSrv->>entSrv: 修改企业状态为审核通过
entSrv-->>entAuditSrv: 修改状态成功
deactivate entSrv
entAuditSrv-->>api: 返回审核成功
deactivate entAuditSrv
deactivate api

```

### 企业录入审核驳回时序图

```mermaid
sequenceDiagram
autonumber
participant api as 企业审核通过接口
participant verify as 请求校验器
participant entAuditSrv as EnterpriseAuditService(企业审核 Service)
participant entSrv as EnterpriseService(企业 Service)
activate api
api->>verify: 请求校验参数
activate verify
verify->>verify: 校验参数格式
opt 参数格式错误
verify-->>api: 返回参数格式错误
end
verify-->>api: 返回校验结果
deactivate verify

opt 校验失败
api->>api:返回错误信息，退出
end
activate entAuditSrv
api->>entAuditSrv: 请求审核驳回
entAuditSrv->>entAuditSrv: 修改审核记录状态为审核驳回
entAuditSrv->>entSrv: 请求修改企业状态
activate entSrv
opt 企业状态不为待审核
entSrv-->>entAuditSrv: 返回无需修改状态
end
entSrv->>entSrv: 修改企业状态为审核驳回
entSrv-->>entAuditSrv: 修改状态成功
deactivate entSrv
entAuditSrv-->>api: 返回审核成功
deactivate entAuditSrv
deactivate api

```

### 企业录入-上传承诺书时序图

```mermaid
sequenceDiagram
autonumber
participant api as 企业审核通过接口
participant verify as 请求校验器
participant entSrv as EnterpriseService(企业 Service)
participant entStateHisSrv as EnterpriseStateHisService(企业状态历史 service)
participant codeSrv as PreviewCodeService(预编号 service)
activate api
api->>verify: 请求校验参数
activate verify
verify->>verify: 校验参数格式
opt 参数格式错误
verify-->>api: 返回参数格式错误
end
verify->>entSrv: 请求查询企业状态
activate entSrv
entSrv->>entSrv: 查询数据库，获取企业状态
entSrv-->>verify: 返回企业状态
deactivate entSrv
opt 企业状态不是已通过审核
verify-->>api: 返回错误【企业还未通过审核】
end
verify-->>api: 返回校验结果
deactivate verify

opt 校验失败
api->>api:返回错误信息，退出
end

api->>entSrv: 请求上传承诺书
activate entSrv
entSrv->>codeSrv: 请求获取预编号
activate codeSrv
codeSrv->>codeSrv: 根据规则生成预编号(使用redis等工具)
codeSrv-->>entSrv: 返回预编号
deactivate codeSrv
entSrv->>entSrv: 修改企业状态为正常,并填入预编号
entSrv->>entStateHisSrv: 请求添加企业状态历史
activate entStateHisSrv
entStateHisSrv->>entStateHisSrv: 添加企业状态历史记录
entStateHisSrv-->>entSrv: 返回添加成功
deactivate entStateHisSrv
entSrv-->>api:返回操作成功
deactivate entSrv
deactivate api
```

### 企业驳回后-重新录入时序图

```mermaid
sequenceDiagram
autonumber
participant api as  企业录入接口
participant verify as  请求校验器
participant entSrv as  EnterprisentService(企业 Service)
participant entAudSrv as EnterpriseAuditService(企业审核 Service)
participant mq as 消息队列
activate api
api->>verify: 请求校验参数
activate verify
verify->>verify: 校验参数格式
opt 参数格式错误
verify-->>api: 返回参数格式错误
end
verify->>entSrv: 请求查询企业审核状态
activate entSrv
entSrv->>entSrv: 查询数据库，获取企业审核状态
entSrv-->>verify: 返回企业审核状态
deactivate entSrv
opt 企业审核状态不是审核驳回
verify-->>api: 返回错误【企业审核状态不是审核驳回】
end
verify-->>api: 返回校验通过
deactivate verify

opt 校验失败
api->>api:返回错误信息，退出
end

api->>entSrv: 请求重新录入企业信息
activate entSrv
entSrv->>entSrv: 修改企业信息
entSrv->>entAudSrv: 请求生成新的审核记录
activate entAudSrv
entAudSrv->>entAudSrv: 根据企业类型选择不同的审核方式(无需审核，单次审核，并行审核等)

alt 单次审核
entAudSrv->>entAudSrv: 插入一条企业待审核记录到数据库
entAudSrv-->>entSrv: 返回待审核状态
entAudSrv->mq: 发送存在待审核记录的消息到mq
else 并行审核
entAudSrv->>entAudSrv: 插入多条企业待审核记录到数据库，并指定审核部门
entAudSrv-->>entSrv: 返回待审核状态
entAudSrv->mq: 发送存在待审核记录的消息到mq
end
deactivate entAudSrv
entSrv->>entSrv: 修改企业审核状态为待审核
entSrv-->>api: 返回重新录入结果
deactivate entSrv
deactivate api
```



## 企业清退

### 企业清退时序图
```mermaid
sequenceDiagram
autonumber
participant api as 企业退出接口
participant verify as 请求校验器
participant entSrv as EnterpriseService
participant vehSrv as VehicleService
participant drvSrv as DriverService
participant entStateHisSrv as EnterpriseStateHisService
activate api
api->>verify: 请求校验参数
activate verify
verify->>verify: 校验参数格式
verify->>vehSrv: 请求企业车辆数量
activate vehSrv
vehSrv->>vehSrv: 查询数据库获取企业车辆数量
vehSrv-->>verify: 返回企业车辆数量
deactivate vehSrv

opt 企业车辆大于0
verify-->>api: 返回错误[企业车辆大于0]
end

verify->>drvSrv: 请求企业驾驶员数量
activate drvSrv
drvSrv->>drvSrv: 查询数据库获取企业驾驶员数量
drvSrv-->>verify: 返回企业驾驶员数量
deactivate drvSrv

opt 企业驾驶员大于0
verify-->>api: 返回错误[企业驾驶员大于0]
end

verify-->>api: 返回校验通过
deactivate verify

opt 校验不通过
api-->> api: 返回错误信息，退出 
end
api->>entSrv: 请求退出企业
activate entSrv
entSrv->>entSrv: 修改企业状态为清退
entSrv->>entStateHisSrv: 请求添加企业状态记录
activate entStateHisSrv
entStateHisSrv->>entStateHisSrv: 添加企业状态记录
entStateHisSrv-->>entSrv: 返回是否添加成功
deactivate entStateHisSrv
entSrv-->>api:返回企业退出结果
deactivate entSrv
deactivate api
```

## 企业审核状态-状态图

```mermaid
stateDiagram-v2
state "审核通过" as pass
state "审核不通过" as reject
state "待审核" as pending
[*]--> pending:创建企业后，默认为待审核
pending-->pass:无需审核，或者审核通过后
pending-->reject:审核驳回后
reject-->pending:审核驳回后，重新录入信息
pass--> [*]
```





## 企业状态-状态图

```mermaid
stateDiagram-v2
state "正常" as normal
state "暂停受理" as pending
state "清退" as cancel
[*]-->pending: 添加企业后，默认为暂停受理
pending-->normal:审核通过后，并且补充必要的信息后
normal-->cancel:企业申请清退
cancel-->[*]
```

# 工程项目

## 工程项目报备

### 工程项目报备录入时序图

```mermaid
sequenceDiagram
autonumber
participant api as 工程项目录入接口
participant verify as 请求校验方法
participant ecdFileSrv as 工程报备Service
participant ecdFileAuditSrv as 工程报备审核Service
participant vehSrv as 车辆service
participant limitRouterSrv as 限速区域Service
participant controlRouterSrv as 管控区域Service
participant mq as 消息队列
activate api
api->>+verify:校验请求信息+
verify->>verify:校验参数格式
opt 参数格式错误
verify-->>api:返回【参数格式错误】
end
loop 循环校验车辆信息
verify->>+vehSrv: 查询车辆信息
vehSrv-->>-verify:返回车辆信息
opt 车辆不符合条件
verify-->>api:返回【车辆信息不符合条件】
end
end
loop 循环路线信息
verify->>+limitRouterSrv: 请求判断路线是否经过限行区，并且时间段是否符合要求
limitRouterSrv->>limitRouterSrv:查询数据库，返回包含路线的限行区
loop 循环查询出的限行区
opt 路线的时间段不符合限行区规定
limitRouterSrv-->>verify: 返回【路线经过限行区，且时间段不符合要求】
end
end
limitRouterSrv-->>-verify: 返回【符合限行区要求】
opt 不符合限行区要求
verify-->>api:返回【路线经过限行区，且时间段不符合要求】
end

verify->>+controlRouterSrv: 请求判断路线是否经过管控区，并且时间段是否符合要求
controlRouterSrv->>controlRouterSrv:查询数据库，返回包含路线的管控区
loop 循环查询出的管控区
opt 路线的时间段不符合管控区规定
controlRouterSrv-->>verify: 返回【路线经过管控区，且时间段不符合要求】
end
end
controlRouterSrv-->>-verify: 返回【符合管控区要求】
opt 不符合管控区要求
verify-->>api:返回【路线经过管控区，且时间段不符合要求】
end
end
verify-->>-api: 返回【校验通过】
api->>+ecdFileSrv: 请求录入工程报备信息
ecdFileSrv->>ecdFileSrv: 录入工程报备信息
ecdFileSrv->>+ecdFileAuditSrv: 按规则录入多条工程报备审核记录
ecdFileAuditSrv->>ecdFileAuditSrv: 往数据库查询工程报备审核记录
ecdFileAuditSrv->mq: 发送工程报备审核信息到mq
ecdFileAuditSrv-->>-ecdFileSrv: 返回录入审核记录成功
ecdFileSrv-->>-api: 返回录入工程报备信息成功
deactivate api
```

### 工程项目报备审核通过时序图

整体流程跟企业审核通过一样

### 工程项目报备审核驳回时序图

整体流程跟企业审核驳回一样



# 路线

## 路线报备
### 路线报备时序图

```mermaid
sequenceDiagram
autonumber
participant api as 路线报备录入接口
participant verify as 请求校验方法
participant ecdFileSrv as 路线报备Service
participant ecdFileAuditSrv as 路线报备审核Service
participant vehSrv as 车辆service
participant limitRouterSrv as 限速区域Service
participant controlRouterSrv as 管控区域Service
participant mq as 消息队列
activate api
api->>+verify:校验请求信息
verify->>verify:校验参数格式
opt 参数格式错误
verify-->>api:返回【参数格式错误】
end
loop 循环校验车辆信息
verify->>+vehSrv: 查询车辆信息
vehSrv-->>-verify:返回车辆信息
opt 车辆不符合条件
verify-->>api:返回【车辆信息不符合条件】
end
end
loop 循环路线信息
verify->>+limitRouterSrv: 请求判断路线是否经过限行区，并且时间段是否符合要求
limitRouterSrv->>limitRouterSrv:查询数据库，返回包含路线的限行区
loop 循环查询出的限行区
opt 路线的时间段不符合限行区规定
limitRouterSrv-->>verify: 返回【路线经过限行区，且时间段不符合要求】
end
end
limitRouterSrv-->>-verify: 返回【符合限行区要求】
opt 不符合限行区要求
verify-->>api:返回【路线经过限行区，且时间段不符合要求】
end
verify->>+controlRouterSrv: 请求判断路线是否经过管控区，并且时间段是否符合要求
controlRouterSrv->>controlRouterSrv:查询数据库，返回包含路线的管控区
loop 循环查询出的管控区
opt 路线的时间段不符合管控区规定
controlRouterSrv-->>verify: 返回【路线经过管控区，且时间段不符合要求】
end
end
controlRouterSrv-->>-verify: 返回【符合管控区要求】
opt 不符合管控区要求
verify-->>api:返回【路线经过管控区，且时间段不符合要求】
end
end
verify-->>-api: 返回【校验通过】
api->>+ecdFileSrv: 请求录入路线报备信息
ecdFileSrv->>ecdFileSrv: 录入路线报备信息
ecdFileSrv->>+ecdFileAuditSrv: 按规则录入多条路线报备审核记录
ecdFileAuditSrv->>ecdFileAuditSrv: 往数据库查询路线报备审核记录
ecdFileAuditSrv->mq: 发送路线报备审核信息到mq
ecdFileAuditSrv-->>-ecdFileSrv: 返回录入审核记录成功
ecdFileSrv-->>-api: 返回录入路线报备信息成功
deactivate api
```

### 路线报备审核通过时序图

整体流程跟企业审核通过一样

### 路线报备审核驳回时序图

整体流程跟企业审核驳回一样



# 线路牌

## 线路牌发放

### 线路牌发放时序图

```mermaid
sequenceDiagram
autonumber
participant api as 线路牌发放接口
participant seasonMethod as  季度线路牌发放方法
participant monthMethod as 月度线路牌发放方法
participant entSrv as 企业Service
participant entRouteCardSrv as 企业线路牌发放Service
participant trafficSrv as 交通违法Service
activate api
api->>seasonMethod:请求发放季度线路牌
activate seasonMethod
seasonMethod->>entSrv: 请求有效的企业列表
activate entSrv
entSrv-->>seasonMethod: 返回企业列表
deactivate entSrv
loop 循环企业列表
seasonMethod->>seasonMethod:查询本季度企业是否生成线路牌
opt 未生成
seasonMethod->>trafficSrv:查询前一季度的交通致死事故统计
activate trafficSrv
trafficSrv-->>seasonMethod:返回交通致死事故统计
deactivate trafficSrv
seasonMethod->>seasonMethod:根据统计信息生成发放量
seasonMethod->>entRouteCardSrv:请求保存本季度路线牌发放量
activate entRouteCardSrv
entRouteCardSrv->>entRouteCardSrv: 保存企业季度路线牌发放量
entRouteCardSrv-->>seasonMethod: 保存成功
deactivate entRouteCardSrv
end
seasonMethod->>monthMethod: 请求生成企业月度线路牌发放量(传入本季度发放量)
activate monthMethod
monthMethod->>trafficSrv: 查询前一个月企业违法数
activate trafficSrv
trafficSrv-->>monthMethod: 返回企业违法数
deactivate trafficSrv
monthMethod->>monthMethod:根据本季度发放量和违法数，生成本月线路牌发放量
monthMethod->>entRouteCardSrv:保存企业月度路线牌发放量
activate entRouteCardSrv
entRouteCardSrv-->>monthMethod:保存成功
deactivate entRouteCardSrv
opt 本月线路牌发放量小于前一个月的线路牌发放量
monthMethod->>entRouteCardSrv: 清空企业已发放的线路牌
activate entRouteCardSrv
entRouteCardSrv-->>monthMethod: 清空成功
deactivate entRouteCardSrv
end
monthMethod-->>seasonMethod: 生成月度路线牌成功
end
deactivate monthMethod
seasonMethod-->>api: 生成季度路线牌成功
deactivate seasonMethod
deactivate api
```

# 电子围栏

## 电子围栏报警

### 电子围栏报警时序图

```mermaid
sequenceDiagram
autonumber
participant consumer as 围栏报警Consumer(订阅围栏进出消息)
participant fenceHisSrv as 电子围栏过车记录Service
participant vehSrv as 车辆Service
participant endSrv as 工程Service
participant fenceAlarmSrv as 电子围栏报警Service
participant mq as 消息队列
activate consumer
consumer->>fenceHisSrv: 请求创建电子围栏过车记录
activate fenceHisSrv
fenceHisSrv->>fenceHisSrv: 插入电子围栏过车记录到数据库
fenceHisSrv-->>consumer: 返回插入成功
deactivate fenceHisSrv
consumer->>vehSrv: 查询车辆是否在目录库
activate vehSrv
vehSrv->>vehSrv: 查询数据库获取车辆信息
vehSrv-->>consumer: 返回车辆是否在目录库
deactivate vehSrv
opt 车辆在目录库
consumer->>endSrv: 查询车辆是否有工程报备，并且在运输时段
activate endSrv
endSrv->>endSrv: 根据车辆ID查询车辆是否有关联的工程
opt 车辆没有关联的工程
endSrv-->>consumer: 返回没有关联的工程
end
endSrv->>endSrv:查询工程的运输时间段
opt 当前时间不在运输时间段里面
endSrv-->>consumer: 返回没有在运输时间段里面
end
endSrv-->>consumer: 返回有工程报备，并且在运输时段里面
deactivate endSrv
opt 有工程报备，并且在运输时段里面
consumer->>consumer:结束
end
end
consumer->>fenceAlarmSrv: 请求添加电子围栏报警记录
activate fenceAlarmSrv
fenceAlarmSrv->>fenceAlarmSrv: 添加报警记录
fenceAlarmSrv-> mq: 发送一条报警消息到消息队列
fenceAlarmSrv-->>consumer: 添加报警记录成功
deactivate fenceAlarmSrv
deactivate consumer
```

