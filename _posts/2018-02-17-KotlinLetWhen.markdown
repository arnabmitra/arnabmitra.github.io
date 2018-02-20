---
layout: post
title:  "Notes on Kotlin standard library functions (let(),run(),apply(),also(),use(),wiht()) "
date:   2018-01-18 20:52:29 -0700
categories: jekyll update
---

This is my attempt to explore more about some function in the [kotlin standard library](https://kotlinlang.org/api/latest/jvm/stdlib/index.html)
By no means is this very complete just some notes.

+------------------+--------------------+--------------------+---------------------+------------------+
| Name of Function | Extension Function |    Return value    |  Argument in block  | block definition |
+------------------+--------------------+--------------------+---------------------+------------------+
| also()           | yes                | T(this)            | explicit it         | (T) -> Unit      |
| apply()          | yes                | T(this)            | implicit this       | (T) -> Unit      |
| let()            | yes                | R(from block body) | explicit it         | (T)->R           |
| run()            | yes                | R(from block body) | implicit this       | T.()->R          |
| with()           | no                 | R(from block body) | implicit this       | T.()->R          |
+------------------+--------------------+--------------------+---------------------+------------------+

## let()
let is a scoping function, it let's you define variables which are only needed only within the scope of the
let function.

```
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R 
```

```
var product = findProduct()
product.let{
    //the variable it, can be used to refer to the product to which the function is being applied to
}

```
Use it when you want the variable not to be used outside the scope.

```
DataConnection.getConnection().let
{
    connection -> //do something with the connection
}
 
 //connection not available outside scope

```
let() can also be made to execute only when the var or val on which it is applied is not null.
```
//only apply the function if the DataConnection.getConnection() return a non null value.
DataConnection.getConnection()?.let
{
    connection -> //do something with the connection
}
 
 //connection not available outside scope
``` 

```
data class Dog(var name: String, var breed: String, var age: Int)

class SomeClass {
    fun addDogs() {

        var dogs = mutableListOf<Dog>()
        val favouriteDog = "Liam"
        dogs.add(
            favouriteDog.let {
            print(this)
            print(it)
            Dog(it, "Boxer", 42)
            )
        }
    }
}

fun main(args: Array<String>) {
    println("Hello, world!")
    SomeClass().addDogs()
}

```

##run()
```
fun <T, R> T.run(block: T.() -> R): R
```


```
 favouriteDog.run {
      print("some thing")
 }

```

####let() vs run()
Comparison:

 - (T) -> R in let(), but defined as T.() -> R in run()
 - their return value is R from block function.
 
##apply()

```
fun <T> T.apply(block: T.() -> Unit): T

```
apply() defines an extension function on all types. When you invoke it, it calls the closure passed in parameter and then returns the receiver object that closure ran on

```
  val favouriteDog = "Liam"
        
  var res=favouriteDog
  .apply { println(dogs) }
  
  println(res)//prints Liam

```

Another example of how it returns this is 
```
 print(LocalDate.now().apply { println(plusDays(1)) })

```
This will print tomorrows date but will still return(and in this case print) today's date.

##also()
Returns this as seen in apply(), but the Argument in block is implied it
```
var dogs = mutableListOf<Dog>()
        val favouriteDog = "Liam"
        favouriteDog.let {
            print(this)
            print(it)
            Dog(it, "Boxer", 42)
        }
        .also { println(it.name) }
        .also { dogs.add(it) }

```
##with()
```
fun <T, R> with(receiver: T, block: T.() -> R): R
```
With is not an extension function but instead it takes T as it's given receiver and returns 
block is defined as T.() -> R so you can return what you wan't from the function block.
for e.g
```
 var resOfWith=with(resultDog)
        {
            resultDog.age =62
            resultDog.name="Liam_1"
            34
        }


        println(resOfWith)//will print 34
```

## use()
Another immensely useful function is use(), since it maps to java try with resources(https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)
```
fun <T : AutoCloseable?, R> T.use(block: (T) -> R): R
```
Executes the given block function on this resource and then closes it down
correctly whether an exception is thrown or not.

```
fun readProperties() = Properties().apply {
    FileInputStream("application.properties").use { fis ->
        load(fis)
    }
}


```

