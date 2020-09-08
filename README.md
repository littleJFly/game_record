# game_record
针对实际工作，主要记录一些游戏开发过程的经验，遇到的一些问题，用到相关技术等。
自己开发中踩过的坑：
1：使用新缓存模型，由于玩家缓存数据在不同的服务器，比如客户端请求更新引导状态，消息是发给大厅，
  实际上玩家正在捕鱼游戏中，这时候在大厅更新缓存DO数据，导致玩家退出游戏缓存更新之后不正确。
  原因：缓存不一致，应该发往对应的服务器，大厅发送消息给游戏服，在游戏服更新玩家缓存数据
2：游戏内配置表，guide_module_control_conf在游戏服配置初始化，由于先启动大厅服，导致报错解决方法:如果是大厅和游戏服公用的，放在大厅
3: 协议handler，按照现有初始化逻辑，world和game服分别注册各自目录下handler下的所有mapping方法，将Message注册到缓存，否者gate无法转发消息。
4：由于原有的新缓存模型生成器，每次生成都会替换最新代码，自定义代码需要手动备份，否则丢失。
 
review别人代码，自己的一些想法
1：一些逻辑稍微复杂的情况，if-else语句过多，可以考虑简化，便于他人理解，提高代码质量
2：一些通用的或者是重复的代码，可以考虑抽取出来，如捕鱼中原来的一些通用协议，同步玩家信息，发射子弹等。
3：增加合理的代码注释，便于之后维护
4：代码中方法的名称应当尽量和实现的逻辑保持一致，达到见名知意
5：容易出现异常的代码块应该增加try-catch

针对代码设计等规范方面
1：可以参考阿里开发手册借鉴一些好的编码方式
2：在代码设计上可以考虑使用一些设计模式来简化编程

开发效率方面
1：在开发功能时，数值给的太晚，导致前期由程序直接配置数值，可能数值不正确，导致测试时结果有误，增加测试时间
2: 前后端开发联调时，对于有些功能比较费时，表现为客户端资源出来比较慢，部分功能无法直接与客户端对接
3：需求文档过于简单，导致程序在开发过程中经常需要确认需求，有些文档功能比较模糊，表达不清楚，导致了最后开发效率比较低，频繁修改影响代码稳定性。

优化
1：简化繁琐的代码逻辑，减少代码分支（if-else语句）
2：对于共性的代码可以抽取出抽象方法或者抽取到上层类
3：考虑有些功能模块是否可以使用设计模式，来简化编程，提升代码的可维护性
4：对于不需要的方法参数不要传进来，不需要的注释直接删除
5：方法的参数不要太长，不便于维护和阅读，可以考虑增加一个实体类
6：代码中对于有些模块，枚举类和常量类都有，没必要，用一个就行
7: 针对相关协议中的参数，可以适当简化命名
8：去除代码中过多的switch-case语法，可以采用注解优化


玩家断线(离线处理):
1:gateServer -> sessionClosed()  -> NotifyPlayerLeftMessage
  检测到某个玩家session关闭，此时通知worldServer
2:worldServer -> playerLeft() 玩家离线处理  -> updatePlayerLeft2Game() 通知gameServer 玩家离线了 UpdatePlayerLeft2GameMessage
3:gameServer -> UpdatePlayerLeft2GameHandler ->  UpdatePlayerLeft2GameMessage 玩家离线处理


***心跳机制***:
HeartBeatHandler -> HeartBeatMessage
客户端每隔一定时间向GateServer发起请求，HeartBeatMessage,GateServer处理心跳回包(HeartBeatResultMessage)
同时GateServer向WorldServer发送心跳请求(HeartBeatMessage)，WorldServer更新玩家在线时间：在赋值之前触发
-> PlayerManager.getInstance().onlineTime(player, true) -> player.setHeartBeat(now); // 更新玩家的心跳时间

心跳时间，客户端每30秒向大厅发送一次心跳，部分游戏有自己的心跳，3秒
服务端处理的最大心跳叠加时间为35秒


离线监听:
worldServer -> GameTask -> run() -> PlayerManager.getInstance().checkDeleteCache(curMinute);
每隔3分钟清理一次离线半小时的玩家数据
PlayerManager -> clearPlayer() // 清理玩家缓存数据
