在loadDocument方法中涉及一个参数EntityResolver，这个类是什么作用呢？
sax官网的解释是，如果sax应用程序需要实现自定义处理外部实体，必须实现此接口并使用setEntitResolver方法向sax驱动器注册
也就是对于sax解析xml文档的时候，sax首先读取该xml文档的声明，根据申明去找相关的DTD定义在网络上去下载
但是如果没有网络或者网络出问题的时候这个体验就比较糟糕了，EntityResolver 的目的就是项目本身可以提供一个如何寻找dtd的声明的方法，由程序本身来实现寻找，自己实现控制这样就避免了在网络上下载的尴尬
spring提供了两个实现去本地找，一个就是dtd查找BeanDtdResolver（默认的是在当前路径下面去找） 和加载xsd的pluggableSchemaResolver（是在META-INF/Spring.schemas文件中找）
这些类的方法resolveEntity（String publicId,String systemId）根据systemId查找

