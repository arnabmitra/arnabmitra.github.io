---
layout: post
title:  "Decorators using Kotlin."
date:   2017-07-11 20:52:29 -0700
categories: jekyll update
---
# Implementing the decorator pattern in Kotlin.

Kotlin makes it easier to implement the decorator pattern by using the "by" keyword 

Lets take my favourite example from the book ``Head First Design Pattern`` of the decorator pattern
![starbuz coffee](/assets/decorator.png)

Here the author describes a fictitious coffee shop where the Base beverage is decorated by various kinds of
Condiments to make the final beverage.

To make a perfect mocha

1 cup of brewed coffee(Dark Roast)
2 servings of your favorite chocolate bar shavings or cocoa powder (roughly 1/4 cup)
a serving of whipped cream
(I know this is a terrible mocha)

Now some people may want 4 servings of Cocoa powder, some people don't like whipped cream :(

Now Starbuzz coffee should be able to accomodate such changes and calculate the cost and description 
accurately.

The Decorator pattern is a perfect use case here, make your base beverage and then decorate it with 
your favorite condiments, perfect ...



The essence of the pattern is that a new class is created, implementing the same interface as the original class
 and storing the instance of the original class as a field.
  Methods in which the behavior of the original class doesn’t need
   to be modified are forwarded to the original class instance."

Excerpt From: Dmitry Jemerov
Svetlana Isakova. “Kotlin in Action.

```
class DelegatingCollection<T> : Collection<T> {
    private val innerList = arrayListOf<T>()

    override val size: Int get() = innerList.size
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun iterator(): Iterator<T> = innerList.iterator()
    override fun containsAll(elements: Collection<T>): Boolean =
            innerList.containsAll(elements)
}
```


Kotlin eliminates all these additional methods because you can just write

```
class DelegatingCollection<T>(
        innerList: Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {}

```

Coming back to Starbuzz coffee example here is some code that implements this 
problem using the decorator pattern in Kotlin

```

interface Beverage {

    var desription: String

    fun getDescription(): String = "Unknown beverage"

    fun cost(): Double
}


interface CondimentDecorator : Beverage {
    override fun getDescription(): String
}

class Espresso : Beverage {
    override var desription: String
        get() = "Espresso"
        set(value) {}

    override fun cost(): Double {
        return 1.99
    }


}

class HouseBlend : Beverage {
    override var desription: String
        get() = "HouseBlend"
        set(value) {}

    override fun cost(): Double {
        return 0.89
    }

}

class DarkRoast : Beverage {
    override fun getDescription(): String {
        return desription
    }

    override var desription: String
        get() = "DarkRoast"
        set(value) {}

    override fun cost(): Double {
        return 0.69
    }

}

class CocoaPowder(val beverage: Beverage):CondimentDecorator,Beverage by beverage
{
    override fun cost(): Double {
        return beverage.cost() +  0.20
    }

    override fun getDescription(): String {
        return beverage.getDescription()+" and 1 serving of Cocoa Powder"
    }

}

class Whip(val beverage: Beverage):CondimentDecorator,Beverage by beverage
{
    override fun cost(): Double {
        return beverage.cost() +  0.10
    }

    override fun getDescription(): String {
        return beverage.getDescription()+" and serving of whipped cream"
    }

}

fun main(args: Array<String>) {
    println("Starbuzzz Coffee")
    var dr:Beverage = DarkRoast()
    dr= CocoaPowder(dr)
    dr= CocoaPowder(dr)
    dr=Whip(dr)
    println("""The cost of the beverage with ingredients:
        |${dr.getDescription()} has cost of: $${dr.cost()}""".trimMargin())
}



```


Running this will give the following output


```
Starbuzzz Coffee
The cost of the beverage with ingredients:
DarkRoast and 1 serving of Cocoa Powder and 1 serving of Cocoa Powder and serving of whipped cream has cost of: $1.19
```
 
