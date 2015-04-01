spring提供了一种叫做BeanFactoryPostProcessor的容器扩展机制。该机制允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做相应的修改。这就相当于在容器实现的第一阶段最后加入一道工序，让我们对最终的BeanDefinition做一些额外的操作，比如修改其中bean定义的某些属性，为bean定义增加其他信息等。

BeanFactoryPostProcessor 接口是对Bean 工厂的后处理操作。 


在Spring 的PropertyPlaceholderConfigurer 类是实现BeanFactoryProcessor 接口中非常有用的类。它用于Spring 从外部属性文件中载入属性，并使用这些属性值替换Spring 配置文件中的占位符变量（${varible}）。 


Spring 的ApplicationContext 容器可以非常方便的使用PropertyPlaceholderConfigurer，只需通过简单的配置即可使用。 

示例： 
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"> 
<property name="location" value="jdbc.properties" /> 
</bean>  
