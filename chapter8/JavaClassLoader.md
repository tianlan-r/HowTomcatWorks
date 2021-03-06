# Java Class Loader

每当创建一个Java类实例时，这个类必须首先被加载到内存中。JVM使用一个类加载器来加载类。通常，类加载器会在一些Java的核心类库，以及环境变量CLASSPATH中的所有目录里寻找要加载的类，如果没找到，就会抛出java.lang.ClassNotFoundException异常。

从J2SE 1.2开始，JVM用到了三个类加载器：bootstrap加载器、extension加载器以及system加载器。这三个加载器之间有父子关系，bootstrap在最顶层，system在最低层。

bootstrap类加载器用来引导启动JVM，当调用java.exe程序时它就开始工作了。它必须使用本地代码来实现，因为它是用来加载JVM运行所需要的类。同时，它还负责加载java的所有核心类，比如java.lang或者java.io包里的类。bootstrap会在核心类库，比如rt.jar、i18n.jar等中查找类，要从哪一个类库中查找依赖于JVM的版本和操作系统。

extension类加载器负责加载标准扩展目录中的类。它简化了程序猿的工作，使得只需要把JAR文件复制到扩展目录即可，jar文件会自动被搜寻到。不同的供应商有不同的扩展类库，SUN的JVM的标准扩展目录是/jdk/jre/lib/ext。

system类加载器是默认的类加载器，它会搜寻在环境变量CLASSPATH中指定的目录和jar包。

那么jvm使用的是哪个类加载器呢？答案是委派模式，这样就可以解决安全问题。没当要载入一个类时，system类加载器首先被调用，但是，它不会马上载入这个类，而是把这个任务委派给它的父亲——extension类加载器，extension类加载器同样会把任务委培给它的父亲——bootstrap加载器。因此，bootstrap加载器是最先尝试加载类的，如果bootstrap加载器没有找到需要的类，extension加载器就会去尝试加载，如果extension加载器也没找到，system加载器才去加载。如果system加载器仍然没有找到，就会抛出ClassNotFoundException异常。

对于安全性来说，这种委派机制是非常重要的。你可以使用安全管理器来限制对某一目录的访问。现在，某个恶意用户编写了一个名为java.lang.Object的类，这个类可以访问硬盘上的任意目录，因为JVM信任java.lang.Object类，它不会监视这个类的活动。如果这个自定义的java.lang.Object被载入，安全管理机制就被轻松地绕过了。幸运的是，由于有委派机制，这种情况不会发生。下面是它的工作原理。

当这个自定义的类在程序的某处被调用时，system加载器把请求委派给extension加载器，extension加载器又委派给bootstrap加载器。bootstrap加载器会在核心类库查找，就会找到标准的java.lang.Object并实例化它，这样，自定义的Object就不会载入了。

通过继承抽象类java.lang.ClassLoader，你还可以编写自己的类加载器。Tomcat需要一个自定义的类加载器有一下几个原因：

- 在加载类的时候可以指定特殊的规则
- 缓存已经加载过的类
- 为了实现类的预加载

