---
layout: post
title: "TestNG+Groovy进行接口测试"
tags: [code]
---
在测试开发组里待了已经有一个多月了，在完成了辉哥布置的几个接口测试的任务以后，在熟悉了测试开发项目的框架以后，大概了解了测试开发工作的主要流程。


### 开发之前的准备工作

* 了解TestNG
* 了解Groovy
* 我们的测试开发框架已经为我们做了什么

#### TestNG
**TestNG**是一个开源的自动化测试框架，关于TestNG的网上教程有很多，也有很多的入门书籍，一个较为全面的网上入门教程的链接：[TestNG教程](https://www.yiibai.com/testng/)。TestNG被设计出来的目的就是优于JUnit，尤其是在集成测试很多类的时候。TestNG一个优于JUnit的地方就是注解。因为我们的测试框架已经帮我们配好了TestNG的开发环境，所以我们不需要去了解TestNG一些底层的配置，我们只需要在我们的测试代码中使用TestNG的一些语法就可以了。下面是我们项目里面主要用到的几个注解。  

1. @Test注解  
@Test注解是TestNG提供的最基本的注解之一，作用在方法或者类上。顾名思义，添加了这个注解的方法会被TestNG自动识别为测试方法，在项目启动时会自动执行所有带有@Test注解的方法。@Test注解支持很多属性，在我们的项目中主要用到的有：  
dataProvider：此属性的值为测试方法提供一个数据驱动的数据源  
description：测试方法的描述信息  
groups：指定测试方法属于哪个组  
enabled：测试方法是否需要被执行，默认为true  
如下是使用@Test注解的一个测试用例：  
``` groovy
/**
 * 无存管  
 * @param args  
 * @param dp  
 */  
@Test(dataProvider='dpNoDepositeBank', description='无存管', groups=['bstransfer', 'T2'], enabled = true)
void noDepositeBank(Map args, List dp) {
	assert dp.size() == 1
	def m = Bstransfer.newBstransferDepositebanksQuery()
			.set_branch_no(BRANCH_NO)
			.set_op_station(OP_STATION)
			.set_password(PASSWORD)
			.bind(dp[0])
			.send()
	assert m.code == 2370001
}
```
其中dataProvider属性的值是一个带有@DataProvider注解的方法的name属性的值，本例中的数据源的配置如下所示：  
``` groovy
@DataProvider(name='dpNoDepositeBank')
Object[][] dpNoDepositeBank() {
	// 普通、信用
	def args = [
		[asset_prop:'0'],
		[asset_prop:'7'],
	].collect { x -> x + [sqlId: SqlBstransferDPEnum.DepositoryNone]}
	return Ds.lst2dp(args)
}
```  
name属性的值和@Test注解中的dataProvider属性的值保持一致。  

2. @DataProvider注解  
@DataProvider注解主要的用途是为测试方法（带有@Test注解的方法）提供测试驱动的数据，@DataProvider的用法参考上文给出的例子。带有@DataProvider注解的方法必须返回一个Object[][]，即一个Object类的二维数组，二维数组中的每个一维数组中的值，在测试方法的入参中可以分配。简单举个例子：首先有一个带有@DataProvider注解的数据源simpleData，示例代码如下：  
``` groovy
@DataProvider(name='simpledata')
Object[][] simpleData() {
    Object[][] obj = [
                      [1,'31231',['fund_code':'000540','fund_account':'40040072']],
                      [2,'3213213',['fund_code':'001621','fund_account':'13213132']]
                      ]
    return obj
}
```
这个数据源返回一个二维数组obj对象，二维数组里有两个一维数组对象，每个一维数组对象里面都有三个值，如第一个一维数组里面保存了1，‘312321’，还有一个Map，Map里面key为fund_code的value值为000540，key为fund_account的value值为40040072。在测试方法中使用这个数据源，如下：  
``` groovy
@Test(dataProvider = 'simpledata')
void test(int i,String s,Map args) {
    println('第一个参数：'+i +'\n第二个参数：' + s + '\n第三个参数：'+args)
}
```
该测试方法的入参对应了数据源中每个一维数组里面的值，运行这个测试方法，控制台输出结果如下图所示：
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/testNG-example.png "IDEA截图")
从输出结果很容易可以看出，在为测试方法配置好数据源以后，数据源的每个一维数组中的值会根据数据类型分配给测试方法的入参（如果测试方法中定义了入参的话）。  

3. @BeforeMethod和@AfterMethod，@BeforeClass和@AfterClass  
这两个注解在项目中有用到，主要的作用是日志记录，就是在执行每个测试类、方法前后记录将执行的类、方法记录到日志文件中，这几个注解在接口自动化测试的时候用不到。  

#### Groovy ####
**Groovy**是一种基于JVM的敏捷开发语言，在语法上面结合了Python，Ruby和SmallTalk的许多强大特性，由于Groovy运行在JVM上，所以Groovy可以自由使用Java的类库，与Java无缝集成。要运行Groovy需要先在本地配置一套Groovy环境。由于Groovy运行在JVM上，所以安装Groovy之前我们还必须在本地安装一套Java环境。  

1. [Java环境配置教程](http://www.runoob.com/java/java-environment-setup.html)   
 
2. [Groovy开发环境配置教程](https://www.jianshu.com/p/777cc61a6202)  

3. [Groovy语法，官方文档地址](http://www.groovy-lang.org/documentation.html)  

4. [W3CSchool的Groovy教程](https://www.w3cschool.cn/groovy/)  
#### 我们的测试开发框架已经为我们做了什么 ####
打开测试开发的项目之前首先确保你的本地有一个IDE开发工具，如InteliJ IDEA，并且已经配置好了Java和Groovy开发环境，由于我们的项目是基于Maven构建的，所以还需要在本地安装Maven环境。  
Maven环境的安装可以参考如下链接：[Maven环境安装](https://www.yiibai.com/maven/maven_environment_setup.html)  
下载并配置完毕Maven以后，我们需要在开发工具中将我们已经配好的Maven环境集成进去，下面是IDEA配置Maven的教程：[IDEA中配置Maven](https://blog.csdn.net/qq_32588349/article/details/51461182)  
史上最简单IDEA使用教程总结，可看可不看，内容太多可能会引起不适：[IDEA教程总结](https://blog.csdn.net/qq_35246620/article/details/61191375)  
如果你本地已经有了Maven环境，因为测试开发的Maven仓库地址和你原先的远程仓库地址不一样，所以需要替换settings.xml文件。确保你替换了IDEA里面的settings文件，控制台（也就是你本地Maven环境的settings文件），如果你IDEA里面配置的Maven的settings.xml文件和本地Maven运行的settings.xml文件不一致，那么需要全部更改，否则使用控制台运行本项目的时候会报错。  
在本地的IDE环境中打开项目，比如用InteliJ IDEA打开项目，可以看到整个项目的目录结构如图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84%E5%9B%BE.png)  
打开src文件夹，可以看到main和test两个子文件夹，main文件夹里存放的是我们定义的接口类，这个稍后介绍，test文件夹就是我们接口测试的主要战场了。test文件夹下面有两个文件夹分别是java和resources，java文件夹下面保存的就是接口测试的代码了，而resources文件夹下面是接口测试需要用到的一些配置文件。
继续展开test文件夹下面的java文件夹，可以看到如图所示的目录结构：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/test%E7%BB%93%E6%9E%84%E5%9B%BE.png)  
qa.t.x.auto包下面有4个不同的包，分别是api、bat、tool、util，其中api包里是我们接口测试的代码，抛开接口测试的代码不谈，我们需要知道我们的测试开发框架已经为我们做了哪些事情：  
1. bat包  
bat包里主要用来提供需要进行一系列查询更改不同数据库，调用不同接口才能完成的操作的整合，比如修复某个账户条件检查的不同条件使其能进行某些业务上的操作，重新开一个资金账号并且激活这个资金账号，重置用户的pid等。可以认为bat包为我们提供了一些工具帮助我们快速生成一些我们需要的数据。bat包下面的工具类已经在jekins上面集成，可以通过访问jekins来使用这些工具。如图是jekins的页面：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/jekins.png)  
如果我想重建某个资金账号的pid，点击T-重建pid，进入重建pid的工作页面，如图：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/rebuild-pid.png)  
点击Build With Paramenters，可以看到右侧页面如图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/pid.png)  
在client_id输入框里根据提示输入client_id，选择对应的环境，点击开始构建即可。  
2. tool包  
tool包下面的类如图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/tool.png)  
需要了解tool包下面两个类的用法，分别是ApiClassGenerator类和SqlEnumGenerator类。  
① ApiClassGenerator类  
ApiClassGenerator类的主要作用是自动生成接口类，也就是main文件夹下面保存的各个接口类。假如开发在wiki上面更新或者新增了一系列新的接口，首先我们需要去test文件夹下面的resources文件夹下面的conf/wadl文件夹下更新或者新建一个.wadl的文件，如图是wadl文件夹下的文件：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/wadl.png)  
打开wadl文件夹下面任意一个.wadl文件，可以看到这是一个xml类型的文件，resources元素里面的base属性值是接口所在机器的地址，每个resources的子元素resource定义了接口的路径，传参以及调用方法。根据wiki里面关于接口的定义，我们需要手动创建或者更新.wadl文件，wadl文件的命名是根据接口所在的大类来的，比如现金理财的接口在trade类下面，所以我们新建现金理财的wadl文件时的命名方式是trade.cashmoney.wadl。  
更新好wadl文件以后，只需要运行ApiClassGenerator类的main方法，就可以在main文件夹下面看到我们在trade.cashmoney.wadl文件里面定义好的接口类了。  
② SqlEnumGenerator类  
SqlEnumGenerator类主要被用来生成查询数据库的接口。对于一些我们经常需要从数据库去查询的数据，我们不需要每次都自己写sql代码去数据库里面把数据捞出来。我们可以在国金证券-质量平台网页上点击SQL模板按钮，如下图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/sqlmodel.png)  
右侧页面显示我们自己定义的SQL模板，选择其中一个点击可以查看SQL模板的SQL代码和需要的参数，如下图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/sqlcode.png)  
输入模板需要的参数，选择一个执行环境，点击执行按钮，将会执行这段SQL代码并且返回执行结果，如下图是执行返回的结果：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/sqlresult.png)  
默认只返回一条结果，如果需要多条数据可以在入参里面加入size参数，值就是你需要的结果条数就可以了，如在入参中加入了size参数以后返回的结果：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/sqlmultiresult.png)  
点击SQL模板按钮，在右侧页面点击"+"按钮，可以新增一个SQL模板，新增SQL模板的页面如下图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/newsql.png)  
在ID栏填入新增SQL模板的ID，可以根据SQL模板的用途来命名，如这个SQL模板是用来查询客户client_id的，ID可以命名为Query_ClientId,ID不能重复。名称一栏根据情况写，业务模块和模板根据SQL模板所在的模块选择，如现金理财查询有基金户待确认的客户的SQL模板的业务模块和模板分别是现金理财和dp。  
运行SqlEnumGenerator类的main方法，就可以把刚刚生成的SQL模板转化成代码里面的一个接口，这样每次取数据的时候只需要直接调用这个接口就可以了。生成的SQL接口是一个枚举类，枚举类的类名是根据添加SQL模板时填写的业务模块和模板生成的，类所在的包是src/main/sql/业务模块，类名是"Sql业务木块模板Enum"，这个枚举类有两个属性，分别是sqlId和sqlName，sqlId是根据添加SQL模板时填写的ID生成的，具体的规则是"fetch_ufdb_业务模块名称_模板名称_ID",sqlName就是填写的名称。每个枚举对象的名称是填写的ID。以现金理财业务的dp模板下的有基金户未确认的用户查询为例，该SQL模板的定义如下图:  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/TAUnchecked.png)    
生成的SqlEnum如下代码：  
``` groovy
TAUnChecked('fetch_ufdb_CashMoney_DP_TAUnChecked','有基金户待确认的用户')
```
每个枚举类里面还有一个类方法用来调用SQL模板，具体的调用方法是：  
```groovy
SqlCashMoneyDPEnum.TAUnChecked.dbCall(args)
```  
args就是传给SQL模板的入参。返回的数据类型是一个List<Map> 
3. util包  
util包里面的类如下图所示：  
![](https://raw.githubusercontent.com/FruitSamurai/MyResources/master/img/code/util.png)  
和大多数的util包一样，这些类主要封装了一些在代码中需要用到的服务方法。