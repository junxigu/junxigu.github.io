---
layout: post
title:  Junit 4功能的简单汇总和简述
date:   2016-09-21 10:00:00 +0800
categories: 技术
tag: Junit
---

* content
{:toc}


**本文是对Junit 4功能的简单汇总和简述，目标读者是想了解一下Junit4了哪些测试工具的人员，所有代码和技术细节都是参考[Junit Wiki](https://github.com/junit-team/junit4/wiki#junit-usage-and-idioms)**

***

# 快速入门和浅尝
此处不细说，直接参考简单明了的[Wiki](https://github.com/junit-team/junit4/wiki/Getting-started)吧，里面的内容就是Junit4 jar包的使用和运行简单测试实例，10分钟即可完成入门


# Junit4功能汇总和简述
Junit4几乎所有功能官方汇总列表，可从此了解Junit4的各种功能，[官方Wiki](https://github.com/junit-team/junit4/wiki)相当简洁明了

此处对较重要的功能稍微汇总，若有感兴趣的功能可到官方Wiki进一步了解：注意 所有代码出自Junit4 wiki

## Assertions和Matchers
[用法详情](https://github.com/junit-team/junit4/wiki/Assertions)
这两种功能都是单元测试的技术基石，用来开发单元测试的断言，例如：

##### 常用Assertion之一：断言text == text
`assertEquals("text", "text");`

注意：参数1是期望值，参数2是实际值

##### 常用Matchers：allOf, equalTo, startsWith
`assertThat("good", allOf(equalTo("good"), startsWith("good"))); `

Matchers都用在assertThat断言中，代码的意思是：此处断言”good” 同时满足所有条件，条件分别是：与“good”相同，以“good”开头

**Matchers好处是：更符合断言语序，Matchers表达的比较语义更丰富和灵活，也易于开发人员扩展**
[用法详情](https://github.com/junit-team/junit4/wiki/Matchers-and-assertthat)

*另：要测试 [异常情况](https://github.com/junit-team/junit4/wiki/Exception-testing) 是否符合预期行为;
要测试 [超时](https://github.com/junit-team/junit4/wiki/Timeout-for-tests) 是否符合预期行为*


## 测试用例的运行前初始化和运行后资源清理
[用法详情](https://github.com/junit-team/junit4/wiki/Test-fixtures)
主要使用**@BeforeClass，@AfterClass, @Before 和 @After**来标注方法，这些标注都很容易理解

* @BeforeClass: 标注static类方法，用于对单元测试类中所有case都是用的资源进行初始化（一般用于初始化高成本资源），每个单元测试类只会运行一次
* @AfterClass**：同上，功能是释放资源
* @Before**：标注实例方法（单元测试用例）,用于对在每个单元测试用例运行前初始化一些变量或资源
* @After**：同上，功能是释放资源

## 测试用例启动器
顾名思义，这些官方的启动器负责启动开发人员开发的测试用例，你可以根据用例所需要的不同功能来选择不同的启动器，官方的启动器有4个，不满足需求可以自己扩展；
此处稍微汇总
[用法详情](https://github.com/junit-team/junit4/wiki/Test-runners)


**IDE内置的单元测试启动器**：Eclipse，Netbeans等内置了

**通过命令行启动单元测试的启动器**：org.junit.runner.JUnitCore；使用的时候直接在命令行窗口跑单元测试
**** JUnit4TestAdapter**: 在Junit3的启动器中运行 用Junit4开发的单元测试 时使用的Junit4启动器adapter

**使用@RunWith来注解的class来作为单元测试启动器**：可以在注解里赋值来指定Junit内置的几个启动器，例如：

*1. 把一堆单元测试用例定义成一整套单元测试* [用法详情](https://github.com/junit-team/junit4/wiki/Aggregating-tests-in-suites)*
	
	@RunWith(Suite.class) // Sute.class是Junit的启动器
	@Suite.SuiteClasses({  //里面的class都是开发人员开发的一些单元测试
		TestFeatureLogin.class,
		TestFeatureLogout.class,
		TestFeatureNavigate.class,
		TestFeatureUpdate.class
	})
	public class FeatureTestSuite {
		// the class remains empty,
		// used only as a holder for the above annotations
	}
	
*2. 给一个测试用例提供不同的参数* [用法详情](https://github.com/junit-team/junit4/wiki/Parameterized-tests)

	@RunWith(Parameterized.class)
	public class FibonacciTest {
	    @Parameters
	    public static Collection<Object[]> data() {
	        return Arrays.asList(new Object[][] {     
	                 { 0, 0 }, { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5 }, { 6, 8 }  
	           });
	    }

	    private int fInput;

	    private int fExpected;

	    public FibonacciTest(int input, int expected) {
	        fInput= input;
	        fExpected= expected;
	    }

	    @Test
	    public void test() {
	        assertEquals(fExpected, Fibonacci.compute(fInput));
	    }
	}

*3. 把不同的单元测试用例弄成不同分类，然后使用 分类启动器 启动不同类别的单元测试* [用法详情](https://github.com/junit-team/junit4/wiki/Categories)

*4. 还有一些其他实验性和第三方启动器* [用法详情](https://github.com/junit-team/junit4/wiki/Test-runners#experimental-runners)

*5. 理论启动器：用于启动使用了“理论”技术开发单元测试的启动器，可参考下文对理论的简述* [用法详情](https://github.com/junit-team/junit4/wiki/Theories)

## 开发单元测试时，一些可重用的Junit功能组件的
[用法详情](https://github.com/junit-team/junit4/wiki/Rules)
通过@Rule注解来标注所使用的 组件，例如：

	@Rule
	public TemporaryFolder tempFolder = new TemporaryFolder();
	
Rule的原理是Junit在初始化一个单元测试类的时候会实例化一个@Rule标注的 rule域实例，并在每个case运行的时候都会调用 rule域实例的evaluate方法

Junit所提供的组件包括：

* TemporaryFolder：提供在测试时生成临时目录和文件的功能组件
* ExternalResource：提供在测试时 自定义一些外部资源（数据库连接，socket等） 的基类，实现它提供具体的资源
* ErrorCollector：测试过程中对 自认为的错误状态进行收集的工具
* Verifier：提供 自定义条件校验器 的基类，实现它提供具体的校验
* TestWatcher：使用它能在Junit运行每个单元测试用例的每个步骤时得到通知，从而收集用例的测试状态信息
* TestName：获取测试用例名称
* Timeout：测试 超时 是否符合预期行为
* ExpectedException：测试 异常情况 是否符合预期行为
* ClassRule：标注类的域，一般是定义一些对整个单元测试类的所有单元测试用例都有影响的域（例如外部资源等），这里不细述，请参考[用法详情](https://github.com/junit-team/junit4/wiki/Rules)
* RuleChain：声明Rule链，链里的Rule是有顺序的
* 另外，可以实现TestRule接口来定制自己的Rule

## 理论：一种针对给一个单元测试用例定义各种可能的输入和前置条件的技术
[用法详情](https://github.com/junit-team/junit4/wiki/Theories)
使用这种启动器，你可以定义测试用例的所有前置条件（不满足前置条件则忽略这个测试用例），以及不同的测试输入参数

***

# Junit的其他常用技术：
* [忽略（跳过）某些测试用例](https://github.com/junit-team/junit4/wiki/Ignoring-tests)
* [对测试用例定义运行的先决条件 assumptions（不满足会自动跳过测试用例](https://github.com/junit-team/junit4/wiki/Assumptions-with-assume)
* [多线程，并发测试](https://github.com/junit-team/junit4/wiki/Multithreaded-code-and-concurrency)
* [持续集成和测试](https://github.com/junit-team/junit4/wiki/Continuous-testing)
* [maven和grade](https://github.com/junit-team/junit4/wiki#junit-usage-and-idioms)


