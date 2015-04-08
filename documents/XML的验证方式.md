
spring 读取xml之前进行相关验证
XML的验证保证了Xml的正确，主要的验证方法有两种，DTD和XSD 

###DTD 是 document type definition 一个DTD文档包含元素的定义规则，元素之间关系的定义规则，元素可以使用的属性，可以使用的实体或者符号规则

###XSD  xml schema definition  可以指定一个xml Schema来验证某个xml文档，检查是否复合要求 使用xsd文档对xml实例文档进行检查，除了要申明命名空间xmlns 之外还必须指定命名空间所对应的xml schema文档的存放位置，它包含两个部分，一个是名称空间uri ，另一个是名称空间所标志的xml schema 文件位置或者URL


验证方式要在文件的头部进行申明

spring 对相关模式的检测这是看文旦中是否包含“DocType” 如果包含就是DTD，否则就是XSD

---

查看更加详细的文档  关于xml
