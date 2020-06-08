---
layout: post
title:  Java的类型加载
date:   2016-12-07 10:00:00 +0800
categories: 技术
tag: Java
---

* content
{:toc}


**本文是对Java的类型加载的汇总和个人理解**

***

# Java的三种类加载器

Java类加载用于动态的把.class二进制流读入JVM并生成相应的Class对象

Java默认提供了3种类加载器，其中一个JVM内置的，用本地方法实现的叫 启动类加载器(BootStrapClassloader)，另外两个类加载器是用Java实现的；其中启动类加载器是不能被Java代码访问到的；用户可以自定义类加载器，但都需要继承ClassLoader

```
import java.util.List;

public class Test {

	public static void main(String... args) throws NoSuchFieldException, SecurityException {

		System.out.println("Test's Classloader: " + Test.class.getClassLoader().getClass().getName());
		System.out.println("System's Classloader: " + System.class.getClassLoader());
		System.out.println("List's Classloader: " + List.class.getClassLoader());

		System.out.print("Classloader Link: ");
		ClassLoader cl = Test.class.getClassLoader();
		do {
			System.out.print(cl.getClass().getName() + " -> ");
			cl = cl.getParent();
		} while (cl != null);
		System.out.println(cl);
	}

}

output：
Test's Classloader: sun.misc.Launcher$AppClassLoader
System's Classloader: null
List's Classloader: null
Classloader Link: sun.misc.Launcher$AppClassLoader -> sun.misc.Launcher$ExtClassLoader -> null

```

由输出可见，Java代码可被三种类加载器加载，但启动类加载器不能被访问到，所以输出为null；并且这3个类加载器有父子关系，组成一条链，它们的关系是专门设计的，这叫“双亲委派模型”

![各种类加载器的关系](../../../../styles/images/classloaders.png)


这3个类加载器分别负责加载不同位置的.class文件:

Class Loader | .class Position
:----------- | -----------: 
BootStrapClassloader         | jre/lib/rt.jar        
ExtClassLoader         | jre/ext/lib/\*.jar、*.class        
AppClassLoader         | CLASSPATH

由此可见，rt.jar中的System和List的类加载器都是BootStrapClassloader，但Java代码不可访问所以输出null；Test则在CLASSPATH路径中，所以由AppClassLoader加载

# 双亲委派模型

有两种方式可以触发类型的加载，一种是使用Class.forName、Classloader.loadClass等API来加载类型；另一种是程序代码引用一种未加载的类型时，JVM解析符号引用前使用AppClassLoader加载类

无论哪种方式触发类型加载，都需要加载器进行加载；当一个加载器加载类型时，双亲委派模型就起作用了，一个双亲委派模型的类加载流程：

1. 当一个加载器接收到加载请求时，首先检查这个类加载器已加载的类型列表里是否已经存在该类型，有则返回该类型
2. 否则委托parent加载器加载类型，这个委托流程会一直下去直到把请求传递到 启动类型加载器，当parent加载器加载成功则结束
3. 若parent加载器没办法加载类型(它会抛异常)，则由本加载器加载该类型，若加载不了则抛异常
4. 若某一个加载成功，则类型会有一个指向该加载器的引用，并且从*开始收到加载请求的加载器*到*实际加载类的加载器*都会在其加载的类型列表里增加一个新类型；*实际加载类的加载器*叫做*定义类型加载器*，加载器链上从*定义类型加载器*到*开始收到加载请求的加载器*都被称为*初始类型加载器*
5. 当一个类型加载了以后，虚拟机会检查其类型信息并使用本类型的*定义类型加载器*加载其父类型和所实现的接口类型(自定义的类型加载器在调用defineClass的时候进行)

这一流程可参考Classloader的源码：

```
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
	synchronized (getClassLoadingLock(name)) {
		// First, check if the class has already been loaded
		Class<?> c = findLoadedClass(name);
		if (c == null) {
		
			...
			
			if (parent != null) { // load class by parent first
				c = parent.loadClass(name, false);
			} else { // use bootstrapclassloader to load the class
				c = findBootstrapClassOrNull(name);
			}
			
			...

			if (c == null) { // parent can't load the class
				// then use self's API to find and load the class
				c = findClass(name);

				// this is the defining class loader; record the stats
				sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
				sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
				sun.misc.PerfCounter.getFindClasses().increment();
			}
		}
		
		...
		
		return c;
	}
}
    
```

双亲委派模型的好处是：使得值得信赖的类必须通过值得信赖的类加载器加载，并供其他类使用

**加载约束**

当两个由不同 定义类型加载器 加载的对象引用同一个限定名(包名)的类型A时，在JVM解析这个A符号引用时，这两个定义类型加载器加载该类型A的过程中，该类型A必须由同一个 加载器加载

# 自定义类加载器

自定义的类加载器也得遵守双亲委派模型，自定义的类加载器的父亲都是AppClassLoader；为了遵守双亲委派模型，自定义的类加载器只需要覆盖findClass方法就行了，当所有祖先类加载器找不到类时，就会使用自定义类加载器的findClass来定位.class文件并使用defineClass创建Class对象

自定义加载器例子

```

class MyClassLoader extends ClassLoader {

	private String basePath;

	MyClassLoader(String basePath) {
		this.basePath = basePath;
	}

	MyClassLoader(ClassLoader parent, String basePath) {
		super(parent);
		this.basePath = basePath;
	}

	@Override
	protected Class<?> findClass(String className) throws ClassNotFoundException {
		byte[] classData = getTypeFromBasePath(className);

		// Try to load the class from base path directory
		if (classData == null) {
			throw new ClassNotFoundException();
		}
		
		// Parse the class binary data to create a Class instance 
		return defineClass(className, classData, 0, classData.length);
	}

	private byte[] getTypeFromBasePath(String typeName) {
		String fileName = basePath + File.separator + typeName.replace(".", File.pathSeparator) + ".class";

		try {
			BufferedInputStream bis = new BufferedInputStream(new FileInputStream(fileName));
			ByteArrayOutputStream out = new ByteArrayOutputStream();

			int c = bis.read();
			while (c != -1) {
				out.write(c);
				c = bis.read();
			}
			return out.toByteArray();
		} catch (IOException e) {
			return null;
		}
	}
	
	public static void main(String[] args) throws Exception{  
        new MyClassLoader("/test").loadClass("ClassName");
    }
}


```

loadClass的参数是类的全限定名(包含包名)

# 参考文献

《深入JAVA虚拟机》