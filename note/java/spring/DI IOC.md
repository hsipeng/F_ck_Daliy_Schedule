## 依赖注入和控制反转
>Dependency Injection和| nversion of control其实就是一个东西的两种不同的说法而已。本质上是一回事。
Dependency Injection是一个程序设计模式和架构模型,一些时候也称作 Invers| on of control,尽管在技术上来讲,
Dependency Injection是一个 Inversion of control的特殊实现, Dependency Injection是指一个对象应用另外一个对象
来提供一个特殊的能力,例如:把一个数据库连接以参数的形式传到一个对象的结构方法里面而不是在那个对象内部自
行创建一个连接。 Inversion of control|和 Dependency Injection的基本思想就是把类的依赖从类内部转化到外部以减少
依赖。应用 d Inversion of Control,对象在被创建的时候,由一个调控系统内所有对象的外界实体,将其所依赖的对象的
引用,传递给它。也可以说,依赖被注入到对象中。所以, Inversion of control|是,关于一个对象如何获取他所依赖的
对象的引|用,这个责任的反转。


