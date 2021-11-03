# ISOP

#### 1.数据接入

路径：ISOP/apps/dataconfig/java

​	postgresql,kafka中的数据

#### 2.任务管理

路径：ISOP/apps/task_manager/java/TaskManager

​	调用SDK 将资源信息写入postgresql

#### 3.集群管理

路径：ISOP/bcm/agent

| 路径                                                    | 功能描述             |
| ------------------------------------------------------- | :------------------- |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/action   | 执行集群管理中的任务 |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/daemon   | 指定执行的类         |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/entity   | 进程信息             |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/firewall | 防火墙配置和更新监控 |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/kafka    | kafka异常监控        |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/ntp      | ntp服务器更新监控    |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/server   | 数据采集服务         |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/service  | 存放service服务类    |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/sigar    |                      |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/ssh      | 开启ssh监听          |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/tools    | 清理用户生成的log    |
| /agent/src/main/java/com/nsfocus/bsa/bcm/agent/utils    | 存放util             |

------

路径：ISOP/bcm/server

| 路径                                                         | 功能描述           |
| ------------------------------------------------------------ | :----------------- |
| server\src\main\java\com\nsfocus\bsa\bcm\server\agent        | 存放代理相关配置   |
| server\src\main\java\com\nsfocus\bsa\bcm\server\backup       | 数据转储收集器配置 |
| server\src\main\java\com\nsfocus\bsa\bcm\server\common       |                    |
| server\src\main\java\com\nsfocus\bsa\bcm\server\componentServiceAdd | 组件服务添加       |
| server\src\main\java\com\nsfocus\bsa\bcm\server\componentServiceAdd\kafkaAdd | 重新启动job管理    |
| server\src\main\java\com\nsfocus\bsa\bcm\server\componet     | 组件配置           |
| server\src\main\java\com\nsfocus\bsa\bcm\server\dao          | 存放中间对象实体   |
| server\src\main\java\com\nsfocus\bsa\bcm\server\db           | 存储集群管理       |
| server\src\main\java\com\nsfocus\bsa\bcm\server\firewallcontroller | 防火墙管理         |
| server\src\main\java\com\nsfocus\bsa\bcm\server\framework    | 框架               |
| server\src\main\java\com\nsfocus\bsa\bcm\server\monitor      | 监听               |
| server\src\main\java\com\nsfocus\bsa\bcm\server\rest         | 存放接口定义       |
| server\src\main\java\com\nsfocus\bsa\bcm\server\upgrade      | 组件更版           |

#### 4.认证中心、心跳检测等服务

路径：\ISOP\bsa\bsa-common

| 路径                                             | 功能描述       |
| ------------------------------------------------ | -------------- |
| src/main/java/com/nsfocus/common/annotation      | 存放自定义注解 |
| src/main/java/com/nsfocus/common/constant        | 存放常量       |
| src/main/java/com/nsfocus/common/core/controller | 存放接口定义   |
| src/main/java/com/nsfocus/common/enums           | 存放枚举类     |
| src/main/java/com/nsfocus/common/exception       | 存放自定义异常 |
| src/main/java/com/nsfocus/common/response        | 存放响应模板   |
| src/main/java/com/nsfocus/common/utils           | 存放util       |

------

路径：\ISOP\bsa\bsa-framework

| 路径                                              | 功能描述            |
| ------------------------------------------------- | ------------------- |
| src/main/java/com/nsfocus/framework/aspectj       | 存放切入方法        |
| src/main/java/com/nsfocus/framework/audit         | 存放日志审计操作    |
| src/main/java/com/nsfocus/framework/config        | 存放配置类          |
| src/main/java/com/nsfocus/framework/interceptor   | 存放git上传忽略配置 |
| src/main/java/com/nsfocus/framework/util          | 存放util            |
| src/main/java/com/nsfocus/framework/web/exception | 全局异常处理        |
| src/main/java/com/nsfocus/framework/web/spring    | 存放Spring工具封装  |

------

路径：\ISOP\bsa\bsa-generator

------

路径：\ISOP\bsa\bsa-system

------

ISOP\bsa\bsa-system\bsa-device-plugin  ----设备插件服务

```
bsa-device-plugin-active-task  --启动task
	

bsa-device-plugin-common
	--src/main/java/com/nsfocus/device/plugin/common/constant  存放常量
	--src/main/java/com/nsfocus/device/plugin/common/entity  存放数据库映射实体
	--src/main/java/com/nsfocus/device/plugin/common/mapper  存放Mybatis映射接口interface
	--src/main/java/com/nsfocus/device/plugin/common/service  存放service服务类
	--src/main/java/com/nsfocus/device/plugin/common/util    存放util
	--src/main/java/com/nsfocus/device/plugin/common/vo      存放中间对象实体
	--src/main/resources/mapper							存放Mybatis数据库XML文件	
	
bsa-device-plugin-sdk-huawei --设备管理huawei
bsa-device-plugin-sdk-passive --设备管理passive
```

ISOP\bsa\bsa-system\bsa-heartbeat -----心跳检测服务

```
bsa-heartbeat-sdk
	--src/main/java/com/nsfocus/heartbeat/sdk/client  存放心跳检测客户端
	--src/main/java/com/nsfocus/heartbeat/sdk/util    存放util
	--src/main/java/com/nsfocus/heartbeat/sdk/vo      存放中间对象实体

bsa-heartbeat-server
	--src/main/build/assembly/bin  --存放启动心跳检测命令
	--src/main/java/com/nsfocus/heartbeat/server/config --存放配置类
	--src/main/java/com/nsfocus/heartbeat/server/constant --存放常量
	--src/main/java/com/nsfocus/heartbeat/server/controller --存放接口定义
	--src/main/java/com/nsfocus/heartbeat/server/dto --存放接口
	--src/main/java/com/nsfocus/heartbeat/server/entity -- 存放数据库映射实体
	--src/main/java/com/nsfocus/heartbeat/server/mapper -- 存放Mybatis的Dao映射接口interface
	--src/main/java/com/nsfocus/heartbeat/server/netty  --存放netty服务端处理
	--src/main/java/com/nsfocus/heartbeat/server/service --存放service服务类
	--src/main/java/com/nsfocus/heartbeat/server/task --存放心跳检测相关任务
	--src/main/java/com/nsfocus/heartbeat/server/vo  --存放中间对象实体
```

ISOP\bsa\bsa-system\bsa-verify ----认证服务

```
src/main/build/assembly/bin  --存放启动心跳检测命令
src/main/java/com/nsfocus/verify/config 	--存放配置类
src/main/java/com/nsfocus/verify/constant	--存放常量
src/main/java/com/nsfocus/verify/controller	--存放接口定义
src/main/java/com/nsfocus/verify/echache	--存放缓存处理
src/main/java/com/nsfocus/verify/entity		--存放数据库映射实体	
src/main/java/com/nsfocus/verify/heartbeat   --存放心跳检测工具
src/main/java/com/nsfocus/verify/jobs        --存放执行job
src/main/java/com/nsfocus/verify/mapper      --存放Mybatis映射接口interface
src/main/java/com/nsfocus/verify/service     --存放service服务类
src/main/java/com/nsfocus/verify/util		--存放util
src/main/java/com/nsfocus/verify/vo			--存放中间对象实体
src/main/resources/mapper				--存放Mybatis数据库XML文件	
```

#### 5.etl相关服务

路径：ISOP\etl\java\common

ISOP\etl\java\common\common-lib   --公共依赖

```
	ISOP\etl\java\common\common-lib\de
	ISOP\etl\java\common\common-lib\dm
```

ISOP\etl\java\common\common-service   --公共服务

```
	src/main/java/com/nsfocus/bsa/common_service  ----存放util
```

ISOP\etl\java\data_load\BsaPersistent   --数据加载

```
src/main/java/com/nsfocus/bsa/alarm   --bsa组件警告配置
src/main/java/com/nsfocus/bsa/apps    --持久化配置
src/main/java/com/nsfocus/bsa/commons --公共服务
src/main/java/com/nsfocus/bsa/configuration --配置类
src/main/java/com/nsfocus/bsa/entity  --存放实体类
src/main/java/com/nsfocus/bsa/es      --es配置
src/main/java/com/nsfocus/bsa/fastmodeapps --hive配置
src/main/java/com/nsfocus/bsa/utils   --存放util
```

ISOP\etl\java\data_transformation  --数据转换

​	ISOP\etl\java\data_transformation\bsa-setl

```
src/main/java/com/nsfocus/bsa/setl/filter
src/main/java/com/nsfocus/bsa/setl/output
src/main/java/com/nsfocus/bsa/setl/parser
src/main/java/com/nsfocus/bsa/setl/utils
```

   ISOP\etl\java\data_transformation\bsa-setl-enhancer

```
src/main/java/com/nsfocus/bsa/setl/utils/parser/enhance 组件增强配置
```

   ISOP\etl\java\data_transformation\bsa-setl-large-flow

```
src/main/java/com/nsfocus/bsa/setl/output
src/main/java/com/nsfocus/bsa/setl/parser
src/main/java/com/nsfocus/bsa/setl/utils
```

   ISOP\etl\java\data_transformation\BsaSetlCommon

```
src/main/java/com/nsfocus/bsa/setl/bean
src/main/java/com/nsfocus/bsa/setl/filter
src/main/java/com/nsfocus/bsa/setl/input
src/main/java/com/nsfocus/bsa/setl/parser
src/main/java/com/nsfocus/bsa/setl/region
src/main/java/com/nsfocus/bsa/setl/utils
```

​    ISOP\etl\java\data_transformation\parsers

