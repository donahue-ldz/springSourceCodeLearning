# java 类加载器

---


###ClassLoader作用
>ClassLoader负责载入系统的所有Resources（Class，文件，来自网络的字节流等），通过ClassLoader从而将资源载入JVM。
ClassLoader主要对类的请求提供服务，当JVM需要某类时，它根据名称向ClassLoader要求这个类，然后由ClassLoader返回这个类的class对象。 

`每个class都有一个reference，指向自己的ClassLoader。`
array的ClassLoader就是其元素的ClassLoader，若是基本数据类型，则这个array没有ClassLoader 。 

---

###主要方法
Java1.1及从前版本中，ClassLoader主要方法：  
Class loadClass( String name, boolean resolve ) ClassLoader.loadClass() 是 ClassLoader 的入口点 。 
`defineClass` 方法是ClassLoader的主要诀窍。该方法接受由原始字节组成的数组并把它转换成 Class对象。原始数组包含如从文件系统或网络装入的数据。  
`findSystemClass` 方法从本地文件系统装入文件。它在本地文件系统中寻找类文件，如果存在，就使用 defineClass 将原始字节转换成 Class 对象，以将该文件转换成类。当运行 Java 应用程序时，这是 JVM 正常装入类的缺省机制。  

`resolveClass`可以不完全地（不带解析）装入类，也可以完全地（带解析）装入类。当编写我们自己的 loadClass 时，可以调用 resolveClass，这取决于 loadClass 的 resolve 参数的值 

`findLoadedClass` 充当一个缓存：当请求loadClass装入类时，它调用该方法来查看 ClassLoader是否已装入这个类，这样可以避免重新装入已存在类所造成的麻烦。应首先调用该方法  。

###工作过程
一般load方法过程如下：

* 调用 findLoadedClass 来查看是否存在已装入的类。  
* 如果没有，那么采用某种特殊的神奇方式来获取原始字节。（通过IO从文件系统，来自网络的字节流等）  
* 如果已有原始字节，调用 defineClass 将它们转换成 Class 对象。  
如果没有原始字节，然后调用 findSystemClass从本地文件系统获取类。  

* 如果 resolve 参数是 true，那么调用 resolveClass 解析 Class 对象。  
如果还没有类，返回 ClassNotFoundException。  
* 否则，将类返回给调用程序。  

###委托模型
自从JDK1.2以后，ClassLoader做了改进，使用了委托模型，所有系统中的ClassLoader组成一棵树，<font color="red">ClassLoader在载入类库时先让Parent寻找，Parent找不到才自己找</font>。  
JVM在运行时会产生三个
`ClassLoader，Bootstrap ClassLoader、Extension ClassLoader和App ClassLoader`。
>* Bootstrap ClassLoader是用C++编写的，在Java中看不到它，是null。它用来加载核心类库，就是在lib下的类库
* Extension ClassLoader加载lib/ext下的类库
* App ClassLoader加载Classpath里的类库

<font color="red">三者的关系为:App ClassLoader的Parent是Extension ClassLoader，而Extension ClassLoader的Parent为Bootstrap ClassLoader。
加载一个类时，首先BootStrap进行寻找，找不到再由Extension ClassLoader寻找，最后才是App ClassLoader。</font>  

###为什么要委托模型
将ClassLoader设计成委托模型的一个重要原因是出于`安全考虑`，比如在Applet中，如果编写了一个java.lang.String类并具有破坏性。假如不采用这种委托机制，就会将这个具有破坏性的String加载到了用户机器上，导致破坏用户安全。但采用这种委托机制则不会出现这种情况。因为要加载java.lang.String类时，系统最终会由Bootstrap进行加载，这个具有破坏性的String永远没有机会加载。  

---

委托模型还带来了一些问题，在某些情况下会产生混淆，如下是Tomcat的ClassLoader结构图:  

                Bootstrap 
                  | 
                System 
                  | 
                Common 
                  |    
            Catalina  Shared 
                      |     
                   Webapp1  Webapp2 ... 

由 Common 类装入器装入的类决不能（根据名称）直接访问由 Web 应用程序装入的类。使这些类联系在一起的唯一方法是通过使用这两个类集都可见的接口。在这个例子中，就是包含由 Java servlet 实现的 javax.servlet.Servlet。  

如果在lib或者lib/ext等类库有与应用中同样的类，那么应用中的类将无法被载入。通常在jdk新版本出现有类库移动时会出现问题，例如最初我们使用自己的xml解析器，而在jdk1.4中xml解析器变成标准类库，load的优先级也高于我们自己的xml解析器，我们自己的xml解析器永远无法找到，将可能导致我们的应用无法运行。  

相同的类，不同的ClassLoader，将导致ClassCastException异常  

---
###线程中的ClassLoader
 
 每个运行中的线程都有一个成员`contextClassLoader`，用来在运行时动态地载入其它类，可以使用方法Thread.currentThread().setContextClassLoader(...);更改当前线程的contextClassLoader，来改变其载入类的行为；也可以通过方法Thread.currentThread().getContextClassLoader()来获得当前线程的ClassLoader。  
>实际上，在Java应用中所有程序都运行在线程里，如果在程序中没有手工设置过ClassLoader，对于一般的java类如下两种方法获得的ClassLoader通常都是同一个。

`this.getClass.getClassLoader()`  
`Thread.currentThread().getContextClassLoader()`

方法一得到的Classloader是静态的，表明类的载入者是谁；
方法二得到的Classloader是动态的，谁执行（某个线程），就是那个执行者的Classloader。
**对于单例模式的类，静态类等，载入一次后，这个实例会被很多程序（线程）调用，对于这些类，载入的Classloader和执行线程的Classloader通常都不同。**



###Web应用中的ClassLoader
 
回到上面的例子，在Tomcat里，WebApp的ClassLoader的工作原理有点不同，它先试图自己载入类（在ContextPath/WEB-INF/...中载入类），如果无法载入，再请求父ClassLoader完成。  
由此可得：  
对于WEB APP线程，它的contextClassLoader是WebAppClassLoader  
对于Tomcat Server线程，它的contextClassLoader是CatalinaClassLoader  

获得ClassLoader的几种方法
可以通过如下3种方法得到ClassLoader：  
this.getClass.getClassLoader(); // 使用当前类的ClassLoader  
Thread.currentThread().getContextClassLoader(); // 使用当前线程的ClassLoader  
ClassLoader.getSystemClassLoader(); // 使用系统ClassLoader，即系统的入口点所使用的ClassLoader。
>注意，system ClassLoader与根ClassLoader并不一样。JVM下system ClassLoader通常为App ClassLoader  

###自定义ClassLoader

* 安全性
类进入JVM之前先经过ClassLoader，所以可以在这边检查是否有正确的数字签名等.  
* 加密
java字节码很容易被反编译，通过定制ClassLoader使得字节码先加密防止别人下载后反编译，这里的ClassLoader相当于一个动态的解码器  
* 归档
可能为了节省网络资源，对自己的代码做一些特殊的归档，然后用定制的ClassLoader来解档  
* 自展开程序
把java应用程序编译成单个可执行类文件，这个文件包含压缩的和加密的类文件数据，同时有一个固定的ClassLoader，当程序运行时它在内存中完全自行解开，无需先安装.  
* 动态生成
可以生成应用其他还未生成类的类，实时创建整个类并可在任何时刻引入JVM  

###资源载入 
所有资源都通过ClassLoader载入到JVM里，那么在载入资源时当然可以使用ClassLoader，只是对于不同的资源还可以使用一些别的方式载入，例如对于类可以直接new，对于文件可以直接做IO等。

####载入类
假设有类A和类B，A在方法method里需要实例化B，可能的方法有3种。
对于载入类的情况，用户需要知道B类的完整名字（包括包名，例如"com.rain.B")

* 使用Class静态方法 Class.forName  
```
    Class cls = Class.forName("com.rain.B"); 
    B b = (B)cls.newInstance();
```
* 使用ClassLoader  
```    
/* Step 1. Get ClassLoader */ 
ClassLoader cl; //具体获得ClassLoader 
/* Step 2. Load the class */ 
Class cls = cl.loadClass("com.rain.B"); // 使用第一步得到的ClassLoader来载入B 
/* Step 3. new instance */ 
B b = (B)cls.newInstance(); // 有B的类得到一个B的实例 
```
* 直接new  
    `B b = new B();` 

####文件载入
（例如配置文件等）假设在com.rain.A类里想读取文件夹 /com/rain/config 里的文件sys.properties，读取文件可以通过绝对路径或相对路径，绝对路径很简单，在Windows下以盘号开始，在Unix下以"/"开始  
对于相对路径，`其相对值是相对于ClassLoader的`，**因为ClassLoader是一棵树，所以这个相对路径和ClassLoader树上的任何一个ClassLoader相对比较后可以找到文件，那么文件就可以找到，**<font color = "red">当然，读取文件也使用委托模型。</font>  

* 直接IO  
```
/** 
 * 假设当前位置是 "C:/test"，通过执行如下命令来运行A "java com.rain.A" 
 * 1. 在程序里可以使用绝对路径，Windows下的绝对路径以盘号开始，Unix下以"/"开始 
 * 2. 也可以使用相对路径，相对路径前面没有"/" 
 * 因为我们在 "C:/test" 目录下执行程序，程序入口点是"C:/test"，相对路径就 
 * 是 "com/rain/config/sys.properties" 
 * （例子中，当前程序的ClassLoader是App ClassLoader，system ClassLoader = 当前的 
 * 程序的ClassLoader，入口点是"C:/test"） 
 * 对于ClassLoader树，如果文件在jdk lib下，如果文件在jdk lib/ext下，如果文件在环境变量里， 
 * 都可以通过相对路径"sys.properties"找到，lib下的文件最先被找到 
 */ 
File f = new File("C:/test/com/rain/config/sys.properties"); // 使用绝对路径 
//File f = new File("com/rain/config/sys.properties"); // 使用相对路径 
InputStream is = new FileInputStream(f); 
```

如果是配置文件，可以通过java.util.Properties.load(is)将内容读到Properties里，Properties默认认为is的编码是ISO-8859-1，如果配置文件是非英文的，可能出现乱码问题。 

* 使用ClassLoader  

```
/** 
 * 因为有3种方法得到ClassLoader，对应有如下3种方法读取文件 
 * 使用的路径是相对于这个ClassLoader的那个点的相对路径，此处只能使用相对路径 
 */ 
InputStream is = null; 
is = this.getClass().getClassLoader().getResourceAsStream( 
       "com/rain/config/sys.properties"); //方法1 
//is = Thread.currentThread().getContextClassLoader().getResourceAsStream( 
       "com/rain/config/sys.properties"); //方法2 
//is = ClassLoader.getSystemResourceAsStream("com/rain/config/sys.properties"); //方法3 

```
如果是配置文件，可以通过java.util.Properties.load(is)将内容读到Properties里，这里要注意编码问题。  

* 使用ResourceBundle  
```
ResourceBundle bundle = ResourceBundle.getBoundle("com.rain.config.sys"); 
```

这种用法通常用来载入用户的配置文件，关于ResourceBunlde更详细的用法请参考其他文档  
总结：有如下3种途径来载入文件  

    1. 绝对路径 ---> IO 
    2. 相对路径 ---> IO 
                ---> ClassLoader 
    3. 资源文件 ---> ResourceBundle 

###web应用里载入资源
在web应用里当然也可以使用ClassLoader来载入资源，但更常用的情况是使用ServletContex
用户程序通常在classes目录下，如果想读取classes目录里的文件，可以使用ClassLoader，如果想读取其他的文件，一般使用ServletContext.getResource()  
如果使用ServletContext.getResource(path)方法，路径必须以"/"开始，路径被解释成相对于`ContextRoot`(**web的根路径**)的路径，此处载入文件的方法和ClassLoader不同


---
###参考资料

ClassLoader 详解及用途(写的不错) ,感谢作者，使得我第一次如此清楚的理解了类加载器机制！
[原文地址](http://blog.chinaunix.net/uid-21227800-id-65885.html)



