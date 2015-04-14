spring的bean在加载的时候经历了相当复杂的过程
BeanClass beanName = (BeanClass)bf.getBean("beanName")
具体顺序如下：
### 转化对应的beanName
为什么要进行这个步骤呢，在查找bean的时候传入的不一定是BeanName 参数可能是别名，也可能是FactoryBean，所以要进行一系列的解析，直到BeanName确定
### 尝试在缓存中加载单例
### bean的实例化
在缓存中获取的bean是原始状态的bean，需要对bean进行实例化，我们需要初始化到我们需要的状态
###原型模式的依赖检查
为了解决循环依赖的过程
###parentBeanFactory
如果在缓存中不能找到对应的bean，就到其父类中去寻找相关的bean 一直递归到最顶端
###将存储xml配置属性的genericBeanDefinition 转换成rootBeanDefinition
原因是所有的配置文件的读取都是到 genericBeanDefinition中的，但是bean加载的时候是针对rootBeanDefinition
所以需要进行一次转化
###寻找依赖
在初始化一个bean的时候，如果这个bean的初始化依赖与其他bean的话，就优先初始化所以依赖的bean
###针对scope进行不同bean的创建
这一步是最重要的也是spring的特性体现
###类型转化
在获取bean的时候我们还可以指定类型，这样将获取的bean的类型转化成我们指定的类型，也可以自己定义转化器


具体展开对每个步骤的###原型模式的依赖检查
为了解决循环依赖的过程
###parentBeanFactory
如果在缓存中不能找到对应的bean，就到其父类中去寻找相关的bean 一直递归到最顶端
###将存储xml配置属性的genericBeanDefinition 转换成rootBeanDefinition
原因是所有的配置文件的读取都是到 genericBeanDefinition中的，但是bean加载的时候是针对rootBeanDefinition
所以需要进行一次转化
###寻找依赖
在初始化一个bean的时候，如果这个bean的初始化依赖与其他bean的话，就优先初始化所以依赖的bean
###针对scope进行不同bean的创建
这一步是最重要的也是spring的特性体现
###类型转化
在获取bean的时候我们还可以指定类型，这样将获取的bean的类型转化成我们指定的类型，也可以自己定义转化器


具体展开对每个步骤的###原型模式的依赖检查
为了解决循环依赖的过程
###parentBeanFactory
如果在缓存中不能找到对应的bean，就到其父类中去寻找相关的bean 一直递归到最顶端
###将存储xml配置属性的genericBeanDefinition 转换成rootBeanDefinition
原因是所有的配置文件的读取都是到 genericBeanDefinition中的，但是bean加载的时候是针对rootBeanDefinition
所以需要进行一次转化
###寻找依赖
在初始化一个bean的时候，如果这个bean的初始化依赖与其他bean的话，就优先初始化所以依赖的bean
###针对scope进行不同bean的创建
这一步是最重要的也是spring的特性体现
###类型转化
在获取bean的时候我们还可以指定类型，这样将获取的bean的类型转化成我们指定的类型，也可以自己定义转化器

