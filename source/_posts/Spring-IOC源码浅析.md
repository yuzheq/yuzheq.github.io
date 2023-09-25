---
title: Spring IOC源码浅析
top: false
cover: true
toc: true
mathjax: true
date: 2021-07-12 16:18:46
password:
summary:
tags: spring
categories: 后端
---

## 一点思考

在开始分析spring IOC的源码之前，我们有必要先理清大致的脉络，而有**三个问题**对脉络的构建至关重要——

1. **从哪里出发**：hello world~ 要想通这一点，我们需要把视角拉回`java web`，不可置否的是javaweb通过servlet组件可以实现一个完整的功能，但对于一个HTTP请求，我们处理的全过程包含`接收请求，处理请求，返回请求`，在这其中只有处理请求的`核心业务逻辑是需要我们去完成`的.

   而接收请求，返回请求是所有业务场景依托的通用部分，它们完成了报文的`解复用，解析，封装`，这里的任何一项留给程序员去做，都足以`让人头皮发麻`。而框架的实现者深知这一点，把最复杂，但也最通用的部分提取了出来。

   > 如果我们想运行任何一个项目，一句简单的helloworld就能像点睛之笔一般让项目枯木逢春。

   转念去想，我们学习任何知识似乎都只需要一句的hello world，看来所有的框架都认为我们不怎么聪明的亚子。。

2. **以什么为参考**：ioc容器的功能是创建并管理对象，所以对象什么时候被创建，什么时候被初始化是核心。

3. **解析的技巧**：让人诟病的框架通常差的千奇百怪，但往往一个优秀的框架在各个方面都无可挑剔，比如`注释，名称`。

   这些**路标**能够指引更高效的前行。

   ------

##  准备工作

我们首先在spring的配置文件中配置了**两个bean对象**：

![image-20210508101816973](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508101816973.png)

并通过简易的helloworld类中**打断点观察源码**的运行轨迹——

![image-20210508102002505](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508102002505.png)

如果我们直接运行项目，会在控制台观察到这些信息：

![image-20210508102357843](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508102357843.png)

接下来我们为创建IOC容器这一步打断点，看看会发生什么——

## 第一站

> 所有的故事都从这一句代码开始了：ioc = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

首先来看debug之后的版面分布：

![image-20210508102839427](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508102839427.png)

核心的逻辑都在创建IOC容器的过程中完成，我们F5进去瞅瞅：

![image-20210508103054925](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508103054925.png)

显然，这是在做类加载相关的事情，F6继续：

![image-20210508103210061](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508103210061.png)

在构造方法中又调用了另一个构造方法，有趣的是不论构造方法怎样重构，最终调用的都是这一句，看来它就是核心啦，贴出来看看——

![image-20210508103417772](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508103417772.png)

显然，配置文件的路径被作为一个**String数组**传入，refresh是个布尔值，它是之后的核心，parent就是老生常谈。

接下来解析xml——

![image-20210508103951949](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508103951949.png)

这一步无非是框架对于XML文件的解析，XML文件中的信息都是String类型的，显然需要将它们提取出来还原为真实的数据类型。但这不是我们的重点，先过咯：

![image-20210508104315170](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508104315170.png)

进入到refresh方法，映入眼帘的是复杂但有迹可循的方法，从这里开始，我们已经揭开了它的第一层面纱，开始第二站

## 第二站

refresh方法起手就是一个同步锁，这样能够保证IOC容器只会被创建一次，继续——

![image-20210508104811500](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508104811500.png)

放过这一句后，控制台打印出了**正在欲刷新ApplicationContext的信息**，不是我们核心关注点，next->

![image-20210508105131661](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508105131661.png)

这一步直观的看，**刷新内部bean工厂创建出子类ConfigurableListableBeanFactory**，来看下BeanFactory的顶层设计：

![image-20210508105727464](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508105727464.png)

看来ConfigurableListableBeanFactory的地位不高，它的祖爷爷**BeanFactory才是万恶之源**，可以看出，它独有的两个方法一个用于**返回遍历bean名称的迭代器**，一个是不知道有啥用的**冻结配置**(你只读的，我怎么改...)。

当我们放过这句后，控制台表示我有话说：

![image-20210508110312292](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508110312292.png)

也就是说，它实现的是**解析XML配置文件的Bean定义，并加载到一个容器中**那加载到哪里呢？ConfigurableListableBeanFactory？我们一看便知：

![image-20210508111122589](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508111122589.png)

IOC容器读取XML中的配置到Bean工厂中以待使用，更具体的说，它`将Bean对象的定义放入了旗下的两个Map`中，徐徐图之：

![image-20210508111658655](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508111658655.png)

见名知意，准备Bean工厂，放过后控制台，变量没有任何表示，next->

![image-20210508111908977](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508111908977.png)

接下来是这个后置处理器的专场，但控制台，变量并没有发生变化，显然并不核心，我们简要的说明他们的功能——

IOC容器的核心功能是创建并管理Bean对象，但它的功能却不仅包含这个，它有不同类型的bean，它还要支撑框架自身的功能，而后置处理器就是做这个事情的。下一步咯：

![image-20210508112239897](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508112239897.png)

初始化消息源？其实这一步是用于支持**国际化功能**的，但它在spring MVC才会一展身手，这里还不是它出场的时机，下一步——

![image-20210508112447315](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508112447315.png)

这里**初始化了事件的转化器**，看不懂，也不属于核心，过：

![image-20210508112616745](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508112616745.png)

留给子类实现？妙，起码暂时不用我关心了嘿嘿，下一个——

![image-20210508112735726](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508112735726.png)

初始化所有不是懒加载(饿汉模式，即**容器启动以后，对象就会被创建**)的**单例对象**。放过试试-》

![image-20210508120936682](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508120936682.png)

这里初始化了所有的单实例bean对象，终于到了最核心的部分，我们的第三步就从这里开始吧——

## 第三步

![image-20210508121322991](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508121322991.png)

值得一提的是，这个方式隶属于`AbstractApplicationContext`,它是`ConfigurableApplicationContext`的实现类，将`ConfigurableListableBeanFactory`(我们刚才创建的Bean工厂)作为参数传入。

在刚开始的部分**初始化类型转换服务**，不是核心部分，继续-》

![image-20210508122420754](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508122420754.png)

这一步就是在为解析做准备了，那下一步自然就是需要解析的部分：

![image-20210508122755207](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508122755207.png)

由于我们没有需要Autowire的部分，所以这一步可以直接过啦——

![image-20210508122955688](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508122955688.png)

继续：

![image-20210508123214114](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508123214114.png)

## 黎明时分

这里比较核心，我们不再截图，上代码：

```java
@Override
	public void preInstantiateSingletons() throws BeansException {
        	//日志记录
		if (this.logger.isDebugEnabled()) {
			this.logger.debug("Pre-instantiating singletons in " + this);
		}、
        	//从beanDefinitionNames中取得bean名称
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

        	//按照xml配置的bean名称创建bean对象
		for (String beanName : beanNames) {
            	   //获取bean的配置信息
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
            	    //确认过代码，是我们需要的bean对象
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                		//显然得到的bean名称是我们希望容器创建的普通bean，而不是factoryBean
				if (isFactoryBean(beanName)) {
					final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
					boolean isEagerInit;
					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
						isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>) () ->
								((SmartFactoryBean<?>) factory).isEagerInit(),
								getAccessControlContext());
					}
					else {
						isEagerInit = (factory instanceof SmartFactoryBean &&
								((SmartFactoryBean<?>) factory).isEagerInit());
					}
					if (isEagerInit) {
						getBean(beanName);
					}
				}
                		//我神经都蹦一块了，你就给我看这个~
				else {
					getBean(beanName);
				}
			}
		}

		// Trigger post-initialization callback for all applicable beans...
		for (String beanName : beanNames) {
            	   //通过名称开始创建对象啦
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
			}
		}
	}
```

我们来大致理理思绪：

![image-20210508124603963](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508124603963.png)

`根据bean名称获取到bean的配置信息-》从配置信息中判断是否是我们需要的bean对象-》如果是，再判断是工厂Bean还是普通Bean-》普通bean，调用getBean。`

看来最终创建bean对象的就是这个getbean方法咯，F6一手：

![image-20210508124353828](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508124353828.png)

一个bean被创建，我们来看看怎样实现的：

![image-20210508124938259](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508124938259.png)

进去看看：

![image-20210508125251963](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508125251963.png)

也就是查询缓存咯，因为我们是第一次注册，显然不存在这个Bean，继续：

![image-20210508130650268](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508130650268.png)

继续，来到了这：

![image-20210508130939660](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508130939660.png)

如果存在依赖的bean，就需要首先先把这个bean创建出来，当前没有，继续：

![image-20210508131336008](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508131336008.png)

终于创建bean对象了，拼夕夕的套路也没你深^-^  

![image-20210508131922757](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508131922757.png)

创建完成啦，但是流程还没完~

![image-20210508133621893](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508133621893.png)

这一步**将创建的singtonObject加入到singletonObjects**这个集合中去，而这个集合又隶属于我们刚才创建的bean工厂`ConfigurableListableBeanFactory`，也就是说我们创建的这个IOC容器是一个多集合的容器，而单例对象容器不过是它的众多集合之一。

![image-20210508133336593](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508133336593.png)

到此，容器终于创建了，那我们来获取bean对象试试——

![image-20210508134719909](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508134719909.png)

先获取ioc容器，然后调用getbean方法从单实例集合中获取指定名称的bean：

![image-20210508134941817](https://zyz-img.oss-cn-chengdu.aliyuncs.com/img/image-20210508134941817.png)

## 终曲

> 简述ApplicationContext与BeanFactory的区别？

从框架设计来看，ApplicationContext是BeanFactory的子接口，BeanFactory是spring框架最底层的接口，它基于最基础，最通用，**最核心的功能创建bean对象**，而ApplicationContext它在BeanFactory的基础上，**更多的关注容器功能的实现**，也就是我们熟识的**IOC,DI,AOP**，再进一步说，**BeanFactory是给ApplicationContext等子接口用的**，而**ApplicationContext直接对接开发人员**，

另外值得一提的是，spring中应用最多的模式即为工厂模式，可以认为我们通过**配置文件，注解来整合成一个制造说明说书**，而底层的工厂帮我们去制造！

------

> 山高路远，静水深流