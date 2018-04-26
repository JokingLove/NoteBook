---
title: spring容器初始化的几种方式
categories: IT
tags: spring
---

spring 通常有五种初始化的方式，代码如下：
<!-- more -->

``` java
package ssh.spring;
 
import java.io.IOException;
 
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
 
public class Test {
   @org.junit.Test
   public void test1(){
      ApplicationContext ac=new ClassPathXmlApplicationContext("ssh/spring/applicationContext.xml");
      Person p1=(Person)ac.getBean("person");
      System.out.println("test1 "+p1);
   }
   @org.junit.Test
   public void test2(){
      ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml",this.getClass());
      Person p1=(Person)ac.getBean("person");
      System.out.println("test2 "+p1);
   }
   @org.junit.Test
   public void test3(){
      Resource resource=new ClassPathResource("ssh/spring/applicationContext.xml");
      BeanFactory beanFactory=new XmlBeanFactory(resource);
      Person p1=(Person)beanFactory.getBean("person");
      System.out.println("test3 "+p1);
   }
   @org.junit.Test
   public void test4() throws IOException{
      Resource resource=new ClassPathResource("applicationContext.xml",this.getClass());
      BeanFactory beanFactory=new XmlBeanFactory(resource);
      Person p1=(Person)beanFactory.getBean("person");
      System.out.println("test4 "+p1);
   }
   @org.junit.Test
   public void test5(){
      Resource resource=new FileSystemResource("E:/Java/study/WebRoot/WEB-INF/classes/ssh/spring/applicationContext.xml");
      BeanFactory beanFactory=new XmlBeanFactory(resource);
      Person p1=(Person)beanFactory.getBean("person");
      System.out.println("test5 "+p1);
   }
}
```