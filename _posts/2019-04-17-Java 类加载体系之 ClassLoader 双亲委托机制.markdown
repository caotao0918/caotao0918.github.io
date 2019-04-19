### Java 类加载体系之 ClassLoader 双亲委托机制

java  是一种类型安全的语言，它有四类称为安全沙箱机制的安全机制来保证语言的安全性，这四类安全沙箱分别是：

- 类加载体系
- .class文件检验器
- 内置于java虚拟机的安全特性
- 安全管理器及javaAPI

#### 类的加载体系如下

java 程序中的  .java 文件编译完会生成  .class 文件，而  .class 文件就是通过被称为类加载器的 ClassLoader加载的，而 ClassLoder 在加载过程中会使用“双亲委派机制”来加载  .class 文件，先上图：  

![](https://img2018.cnblogs.com/blog/1537462/201904/1537462-20190419105227042-49221283.png)



**`BootStrapClassLoader`** ：启 动 类 加 载 器 ，该`ClassLoader` 是  jvm  在 启 动 时 创 建 的 ，用 于 加载`$JAVA_HOME$/jre/lib` 下面的类库（或者通过参数-`Xbootclasspath` 指定）。由于启动类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不能直接通过引用进行操作。  

**`ExtClassLoader`**：扩展类加载器，该 `ClassLoader` 是在 sun.misc.Launcher 里作为一个内部类 `ExtClassLoader`  定义的（即 `sun.misc.Launcher$ExtClassLoader`），`ExtClassLoader` 会加载 `$JAVA_HOME/jre/lib/ext` 下的类库（或者通过参数`-Djava.ext.dirs` 指定）。  

**`AppClassLoader`**：应用程序类加载器，该  `ClassLoader`  同样是在  `sun.misc.Launcher`  里作为一个内部`AppClassLoader` 定义的（即 `sun.misc.Launcher$AppClassLoader`），`AppClassLoader` 会加载 `java` 环境变量  `CLASSPATH`  所指定的路径 下的类库，而  `CLASSPATH`  所指定的   路   径   可   以   通   过`System.getProperty("java.class.path")`获取；当然，该变量也可以覆盖，可以使用参数-cp，例如：`java -cp` 路径  （可以指定要执行的 class 目录）。  

**`CustomClassLoader`**：自定义类加载器，该 `ClassLoader` 是指我们自定义的 `ClassLoader`，比如 `tomcat` 的`StandardClassLoader` 属于这一类；当然，大部分情况下使用 `AppClassLoader` 就足够了。

前面谈到了 ClassLoader 的几类加载器，而 ClassLoader 使用双亲委派机制来加载 class 文件的。ClassLoader的双亲委派机制是这样的（这里先忽略掉自定义类加载器 CustomClassLoader）：  

1）当 AppClassLoader 加载一个 class 时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器 ExtClassLoader 去完成。  

2）当 ExtClassLoader  加载一个  class 时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader 去完成。  

3）如果  BootStrapClassLoader  加载失败（例如在$JAVA_HOME$/jre/lib  里未查找到该  class），会使用ExtClassLoader 来尝试加载；  

4）若 ExtClassLoader 也加载失败，则会使用 AppClassLoader 来加载，如果 AppClassLoader 也加载失败，则会报出异常 ClassNotFoundException。  

下面贴下 ClassLoader 的 loadClass(String name, boolean resolve)的源码：  

```java
1．protected synchronized Class<?> loadClass(String name, boolean resolve)
2．  throws  ClassNotFoundException  {
3．	//  首先找缓存是否有 class
4．	Class  c  =  findLoadedClass(name);
5．	if  (c  ==  null)  {
6．	//没有判断有没有父类
7．	try  {
8．	if  (parent  !=  null)  {
9．	//有的话，用父类递归获取 class
10．	c = parent.loadClass(name, false);
11．	} else {
12．	//没有父类。通过这个方法来加载
13．	c = findBootstrapClassOrNull(name);
14．	}
15．	} catch (ClassNotFoundException e) {
16．	// ClassNotFoundException thrown if class not found
17．	// from the non-null parent class loader
18．	}
19．	if (c == null) {
20．	//  如果还是没有找到，调用 findClass(name)去找这个类
21．	c = findClass(name);
22．	}
23．	}
24．	if (resolve) {
25．	resolveClass(c);
26．	}
27．	return c;
28．  }
```

代码很明朗：首先找缓存（findLoadedClass），没有的话就判断有没有 parent，有的话就用 parent 来递归的 loadClass，然而 ExtClassLoader 并没有设置 parent，则会通过 findBootstrapClassOrNull 来加载 class，而findBootstrapClassOrNull 则会通过 JNI 方法”private native Class findBootstrapClass(String name)“来使用 BootStrapClassLoader 来加载 class。  

然后如果 parent 未找到 class,则会调用 findClass 来加载 class，findClass 是一个 protected 的空方法，可以覆盖它以便自定义 class 加载过程。另 外 ，虽 然   ClassLoader   加载类是使用  loadClass  方法 ，但 是 鼓 励 用  ClassLoader   的 子 类 重 写  findClass(String)，而不是重写 loadClass，这样就不会覆盖了类加载默认的双亲委派机制。  

**双亲委派托机制为什么安全**

举个例子，ClassLoader 加载的 class 文件来源很多，比如编译器编译生成的 class、或者网络下载的字节码。而一些来源的 class  文件是不可靠的，比如我可以自定义一个 java.lang.Integer 类来覆盖 jdk  中默认的 Integer  类，例如下面这样：

```java
1. package java.lang;
2. public class Integer {
3.   public Integer(int value) {
4. 		System.exit(0);
5.	}
6. }
```

初始化这个 Integer 的构造器是会退出 JVM，破坏应用程序的正常进行，如果使用双亲委派机制的话该  Integer 类永远不会被调用，以为委托 BootStrapClassLoader 加载后会加载 JDK 中的 Integer 类而不会加载自定义的这个，可以看下下面这测试个用例：  

```java
1．public static void main(String...  args) {
2．	Integer i = new Integer(1);
3．	System.err.println(i);
4．}
```

*执行时 JVM 并未在 new Integer(1)时退出，说明未使用自定义的 Integer，于是就保证了安全性。*
