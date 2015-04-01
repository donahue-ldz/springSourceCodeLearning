* propertyEditor  属性编辑器是什么？我们在xml里面定义Bean的时候对Bean的属性的值采取字符串等的方式指定，但是如何将这些最常见的字符串属性转化成相关的类的属性的值的设置呢～比如就是int value = Integer.valueOf("1");属性编辑器自动将字符创格式的值转化成整形类型  但是有些时候我们不能转换或者说固定的那些转换是不能实现我们想要的东西的，才是我们可以指定自己的属性编辑器
我们可以自己定义自己的属性编辑器实现将比如字符串的属性值转换成自己所需要的对象等

---
例如：
```
class Address{
    private String street;
    private String doorNum;
    private String postNum;
    //Getter And Setter
}
```
此时我们要是指定自己的字符串属性配置的话，讲不能正确的被解析，毕竟谁知道这是什么玩意，如何解析对吧
我们此时可以定义自己的属性编辑器去解析
我们没有必要去直接继承propertyEditor去实现很多方法，我们可以直接继承PropertyEditorSupport去重写自己感兴趣方法就好了

```
public class AddressPropertyEditor extends PropertyEditorSupport{  
    //支持的格式为 streeValue,doorNumValue,postCode  
    public void setAsText(String text)  
    {  
        System.out.println("使用自己的编辑器。");  
        if (text == null || !StringUtils.hasText(text)) {  
            throw new IllegalArgumentException("老大，不能为空啊！");  
        }  
        else  
        {  
            String[] strArr = StringUtils.tokenizeToStringArray(text,",");  
            Address add = new Address();  
            add.setStreet(strArr[0]);  
            add.setDoorNum(strArr[1]);  
            add.setPostCode(strArr[2]);  
            /**将值对象赋值给需要被赋值的属性的对象
            setValue(add);  
        }  
    }  
      
    public String getAsText()  
    {  
        Address add = (Address)getValue();  
        return ""+add;  
    }  
}  

```


```
public class Person {  
    private String name;  
  
    private Address address;  
  //Getter And Setter
 }   

``

接下来就是我们的配置文件了

```
  <bean id="customEditorConfigurer"  class="org.springframework.beans.factory.config.CustomEditorConfigurer">  
  <property name="customEditors">  
    <map>  
      <entry key="Address"> <!-- 属性类型类 -->  
        <bean class="AddressPropertyEditor"/> <!--对应Address的编辑器 -->  
      </entry>  
    </map>  
  </property>  
</bean>  
  
 <bean id="person" class="com.stamen.propedit.Person">  
    
    <property name="address" value="朝阳区,Soho 1601,010101"/>  
 </bean>  
```

这样他就能将addr这个字符串解析构造成我们所期待的类了
