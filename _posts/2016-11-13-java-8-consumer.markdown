---
layout: post
title:  "Java 8 Cosumer Functional Inteface."
date:   2016-11-13 20:52:29 -0700
categories: jekyll update
---
Java 8 offers a Functional interface called Consumer,A consumer allows you to take in a value and return no results.It is expected to have side effects.
{% highlight java linenos %}
public class ConsumerExample {
  public static void main(String[] args)
  {
    Arrays.asList("a","b","c","d").stream().forEach(testConsumer);
  }
  static Consumer<String> testConsumer=(x)->{System.out.println(x);};
}
{% endhighlight %}
More information can be found here `https://docs.oracle.com/javase/8/docs/api/java/util/function/Consumer.html`



