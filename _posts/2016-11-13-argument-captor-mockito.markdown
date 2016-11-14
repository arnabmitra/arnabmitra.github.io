---
layout: post
title:  "ArgumentCaptor usage in Mockito."
date:   2016-11-13 20:52:29 -0700
categories: jekyll update
---

-- From Mockito Documentation 
Use it to capture argument values for further assertions.
Mockito verifies argument values in natural java style: by using an equals() method. This is also the recommended way of matching arguments because it makes tests clean & simple. In some situations though, it is helpful to assert on certain arguments after the actual verification. For example:
   ArgumentCaptor argument = ArgumentCaptor.forClass(Person.class);
   verify(mock).doSomething(argument.capture());
   assertEquals("John", argument.getValue().getName());
Here is a simple example where we test that the JMS producer was called with a 
particular person object.


Test for testing out the InviteToConference#joinConference method.

{% highlight java linenos %}
import org.junit.Before;  
 import org.junit.Test;  
 import org.mockito.ArgumentCaptor;  
 import org.mockito.Captor;  
 import org.mockito.InjectMocks;  
 import org.mockito.Mock;  
 import org.mockito.Mockito;  
 import org.mockito.MockitoAnnotations;  
 import static org.junit.Assert.*;  
 public class InviteConferenceTest {  
 @Mock  
 JmsProducer mockJmsProducer;  
 @Captor  
 private ArgumentCaptor personCaptor;  
 @InjectMocks  
 InviteToConference test=new InviteToConference();  
 @Before  
 public void initMocks()  
 {  
 MockitoAnnotations.initMocks(this);  
 }  
 @Test  
 public void testForArgumentCaptor()  
 {  
 Person newPerson=new Person("Arnab","Mitra");  
 test.joinConference(newPerson);  
 Mockito.verify(mockJmsProducer).sendMessage(personCaptor.capture());  
 Person p=personCaptor.getValue();  
 assertEquals("Arnab", p.getFirstName());  
 assertEquals("Mitra", p.getLastName());  
 }  
 }  
{% endhighlight %}

{% highlight java linenos %}
public class InviteToConference {  
 private Person person;  
 private JmsProducer jmsProducer;  
 public void joinConference(Person p)  
 {  
 jmsProducer(p);  
 }  
 public void jmsProducer(Person p)  
 {  
 jmsProducer.sendMessage(p);  
 //send a jms message maybe  
 }  
 }  
{% endhighlight %}
{% highlight java linenos %}
 public class Person {  
 private String firstName;  
 private String lastName;  
 public String getFirstName() {  
 return firstName;  
 }  
 public void setFirstName(String firstName) {  
 this.firstName = firstName;  
 }  
 public String getLastName() {  
 return lastName;  
 }  
 public void setLastName(String lastName) {  
 this.lastName = lastName;  
 }  
 public Person(String firstName, String lastName) {  
 super();  
 this.firstName = firstName;  
 this.lastName = lastName;  
 }  
 }  
{% endhighlight %}
{% highlight java linenos %}
 public interface JmsProducer {  
 public Person sendMessage(Person p);  
 }  
{% endhighlight %}