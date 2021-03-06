# php-game-server

技术架构：
PHP + MYSQL + MEMCACHE结构，JSON通信，心跳推送，NOSQL作为主数据格式，支持热更新
在高并发性能、服务器扩展性、版本更新维护、开发效率上都有不错的表现
如需要处理实时双向通信，可方便集成C++端
NOSQL和MEMCACHE在单台服务器无法支撑时，可以考虑分布式部署
同样应用层、服务层、数据层可以考虑剥离到不同服务器进行部署

背景介绍：
使用PHP作为主架构的游戏已经有不少了，比如《我叫MT》，从架构成熟度上来说完全可以负担大型游戏。理论上，我认为，不需要实时连接的游戏都应该使用PHP架构以提升开发效率，降低开发成本和风险。这套精简的服务端框架，前后共经历了三款游戏的完整开发，迭代更新后已经趋于稳定，基本解决了所有网游中会碰到的性能、并发、推送、数据更新、安全性等问题。

文件目录结构说明：
daemon 功能守护模块，静态类，APP层用于与客户端连接，MOD层为模块底层，indexcontrol为重要的入口处理文件
enums 静态配置，包括公式以及文字
includes 公共类
object 可被实例化的类，比如玩家及佣兵对象
debug 测试用例
webservice 服务接口
crontab 定时服务
doc 说明文档

类命名规则：
daemon enums object的类名均按照目录结构命名  例如 daemon/a.php 文件里面的 类名应该为 Daemon_A
所有用到的类均可自动加载  不用手动include文件

通信规则：
JSON格式通信
用户只能访问App层，增加访问安全性
服务端数据大部分缓存到客户端,然后更新最新的数据到客户端
只刷新客户端需要的数据，不要什么都打包过去。都用最小输出。

RETURN规则：
1 正确
0 错误
负数 错误类型
其他格式

编程风格：
全部用驼峰命名，写清注释
全部大写的话就用_分割

数据库说明：
配置文件都以cfg_开头
日志文件都以log_开头
缓存文件以为m_开头

服务端向客户端推送机制：
个人推送，写在用户数据里，心跳去读
全服推送，写在系统变量，用户心跳去读(比如弄个FLAG，用户也弄个，系统的FLAG30秒后清除)
用户定时去取

缓存机制：
玩家对象以及佣兵对象做缓存对象
系统对象做缓存对象
缓存中的数据不以完整形态进入，比如ITEM，只保留ID及变化属性，基本属性实时去配置文件取
尽量更新缓存数据，而非DB
每10分钟进行数据库倾倒

流程规则：
输出时要判断出错的话就取消输出variable和command
save应该在APP层，要考虑并发情况
setvariable和command可以同时存在于APP和MOD层
心跳建议设置在10S一次，仅为刷新聊天和通告
写的时候，先往APP写，然后抽到MOD里

并发问题：
1. 单个缓存文件，只能由一个对象访问
2. 1的基础上加内存锁
3. 对其他用户的推送信息采用心跳读取beat表方式

死锁问题：
系统用的强制解锁(等10秒循环)或者暂缓(忽略,下次再调用)
个人用的报错

多处登陆问题：
存一个SESSIONID，每次比对，不同的话就是他处登陆
不过这样就不能在线修改别人数据了

PHP注意点：
addslashes 用于用户输入的数据库查询地方
数值处理，都要转换成整数，用到小数的话，比较大小时会出问题
json解析后，array格式的数据必须从小往大赋值，否则会在JSON转换后成为OBJECT

MYSQL注意点：
select count(*) from 不加WHERE
myisam datetime varchar

服务端配置文件：
1. conf.ini
2. includes/global.php => line30 conf地址
3. crontab目录下$globleFile地址