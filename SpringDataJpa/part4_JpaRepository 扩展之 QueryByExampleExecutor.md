## JpaRepository 扩展之 QueryByExampleExecutor
###JpaRepository 介绍

从 JpaRepository 开始的子类，都是 Spring Data 项目对 JPA 实现的封装与扩展。JpaRepository 本身继承 PagingAndSortingRepository 接口，是针对 JPA 技术的接口，提供 flush()、saveAndFlush()、deleteInBatch()、deleteAllInBatch() 等方法。我们来看一下 UML 来对 JpaRespository 有个整体的认识。
![在这里插入图片描述](http://images.gitbook.cn/af50bef0-2eab-11e8-8e27-3f3d79ceb212)

- 从图中其实可以发现，JPA 的实现类最关键是：SimpleJpaRepository，我们多次提到，还有一个最关键的实现类是 QuerydslJpaRepository，会在后面继续介绍。
- 从图中还可以看出来，最关键的几个接口 QueryByExampleExecutor、JpaSpecificationExecutor。
- 从图中还可以好好体会一些接口的用意（暴露那些该暴露的操作方法，而不是一股脑的把所有的方法都暴露给使用的人，因为不是每个场景下面都会用到所有方法。作者感悟在实际工作中，当我们去设计公共方法或者架构的时候，要充分考虑清楚抽象类和接口的区别及其应用场景。）