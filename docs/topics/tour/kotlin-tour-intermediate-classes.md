[//]: # (title: Intermediate Classes)

## Class inheritance

In the previous chapter, we covered how to extend classes without modifying the original source code. But what if you want
to have shared code between classes? In this case, class inheritance can help you with that.

In Kotlin, by default, classes can't be inherited. To make a class inheritable, use the `open` keyword before your class
declaration:

```kotlin
open class Vehicle
```

By having to explicitly configure your class for inheritance, this helps prevent unintended inheritance and makes 
your classes easier to maintain. By default, there aren't any subclasses that you need to worry about during your
development.

To create a class that inherits from another, add a colon after your class header followed by a call to the constructor 
of the parent class that you want to inherit from:

```kotlin
class Car : Vehicle
```
{validate="false"}

In this example, the `Car` class inherits from the `Vehicle` class:

```kotlin
open class Vehicle(val make: String, val model: String)

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model)

fun main() {
    // Creates an instance of the Car class
    val car = Car("Toyota", "Corolla", 4)
}
```

Just like when creating a normal class instance, if your class inherits from a parent class, then it must initialize
all the parameters declared in the parent class header. So in the example, the `car` instance of the `Car` class initializes
the parent class parameters: `make` and `model`.

Kotlin classes only support **single inheritance**, so it is only possible to inherit from **one class at a time**. Otherwise,
you will see an error:

```kotlin
open class Animal(val species: String)

open class Machine(val model: String)

// The RobotAnimal class tries to inherit from both the Animal class and the Machine class
class RobotAnimal(species: String, model: String) : Animal(species), Machine(model)
// Error: Only one class may appear in a supertype list
```
{validate="false"}

At the highest point in the Kotlin class hierarchy is the common parent class: `Any`. Ultimately, all classes inherit
from the `Any` class:

![An example of the class hierarchy with Any type](any-type-class.png){thumbnail="true" width="200" thumbnail-same-file="true"}

The `Any` class provides the `equals()` and `toString()` functions as member functions automatically. Therefore, you can
use these inherited functions in any of your classes. For example:

```kotlin
open class Vehicle(val make: String, val model: String)

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model)

fun main() {
    //sampleStart
    val car1 = Car("Toyota", "Corolla", 4)
    val car2 = Car("Honda", "Civic", 2)
    
    // Uses the .toString() function via string templates to print class properties
    println("Car1: make=${car1.make}, model=${car1.model}, numberOfDoors=${car1.numberOfDoors}")
    println("Car2: make=${car2.make}, model=${car2.model}, numberOfDoors=${car2.numberOfDoors}")
    // Car1: make=Toyota, model=Corolla, numberOfDoors=4
    // Car2: make=Honda, model=Civic, numberOfDoors=2
    
    // Uses the .equals() function via the == operator to check if the make and model are the same
    val areMakeEqual = car1.make == car2.make
    val areModelEqual = car1.model == car2.model
    
    println("Car make is the same: $areMakeEqual")
    println("Car model is the same: $areModelEqual")
    // Car make is the same: false
    // Car model is the same: false
    
    //sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-any-class"}

### Overriding inherited behavior

If you want to inherit from a class but change some of the behavior, you can do so by overriding the inherited behavior.

By default, it's not possible to override a member function or property of a parent class. Just like with classes, you need
to add special keywords.

#### Member functions

To allow a function in the parent class to be overridden, use the `open` keyword before it's declaration in the parent class:

```kotlin
open fun displayInfo() {}
```
{validate="false"}

To override an inherited member function, use the `override` keyword before the function declaration in the child class:

```kotlin
override fun displayInfo() {}
```
{validate="false"}

For example:

```kotlin
open class Vehicle(val make: String, val model: String) {
    open fun displayInfo() {
        println("Vehicle Info: Make - $make, Model - $model")
    }
}

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model) {
    override fun displayInfo() {
        println("Car Info: Make - $make, Model - $model, Number of Doors - $numberOfDoors")
    }
}

fun main() {
    val car1 = Car("Toyota", "Corolla", 4)
    val car2 = Car("Honda", "Civic", 2)

    // Uses the overridden displayInfo() function
    car1.displayInfo()
    // Car Info: Make - Toyota, Model - Corolla, Number of Doors - 4
    car2.displayInfo()
    // Car Info: Make - Honda, Model - Civic, Number of Doors - 2
}
```
{kotlin-runnable="true" id="kotlin-tour-class-override-function"}

This example:
* Creates two instances of the `Car` class that inherit from the `Vehicle` class: `car1` and `car2`.
* Overrides the `displayInfo()` function in the `Car` class to also print the number of doors.
* Calls the overridden `displayInfo()` function on `car1` and `car2` instances.

#### Properties

The syntax for overriding properties is exactly the same as for overriding member functions.

To allow a property in the parent class to be overridden, use the `open` keyword before it's declaration in the parent class:

```kotlin
open val transmissionType: String
```
{validate="false"}

To override an inherited member function, use the `override` keyword before the function declaration in the child class:

```kotlin
override val transmissionType: String
```
{validate="false"}

Properties can be overridden in the class header or in the class body.

<table header-style="top">
   <tr>
       <td>Class header</td>
       <td>Class body</td>
   </tr>
   <tr>
<td>

```kotlin
open class Vehicle(val make: String, val model: String, open val transmissionType: String)

class Car(make: String, model: String, val numberOfDoors: Int, override val transmissionType: String = "Automatic") : Vehicle(make, model)
```

</td>
<td>

```kotlin
open class Vehicle(val make: String, val model: String) {
    open val transmissionType: String = ""
}

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model) {
    override val transmissionType: String = "Automatic"
}
```

</td>
</tr>
</table>

> If the property you want to override in the parent class has no default value, you must initialize it in the child class.
>
{type="note"}

For example:

```kotlin
open class Vehicle(val make: String, val model: String) {
    open val transmissionType: String
    open fun displayInfo() {
        println("Vehicle Info: Make - $make, Model - $model, Transmission Type - $transmissionType")
    }
}

class Car(make: String, model: String, val numberOfDoors: Int) : Vehicle(make, model) {
    override val transmissionType: String = "Automatic"
    override fun displayInfo() {
        println("Car Info: Make - $make, Model - $model, Transmission Type - $transmissionType, Number of Doors - $numberOfDoors")
    }
}

fun main() {
    val car1 = Car("Toyota", "Corolla", 4)
    val car2 = Car("Honda", "Civic", 2)

    car1.displayInfo()
    // Car Info: Make - Toyota, Model - Corolla, Transmission Type - Automatic, Number of Doors - 4
    car2.displayInfo()
    // Car Info: Make - Honda, Model - Civic, Transmission Type - Automatic, Number of Doors - 2

}
```
{kotlin-runnable="true" id="kotlin-tour-class-override-property"}

This example:
* Creates two instances of the `Car` class that inherit from the `Vehicle` class: `car1` and `car2`.
* Overrides the `transmissionType` variable in the `Car` class to `"Automatic"`.
* Overrides the `displayInfo()` function in the `Car` class to also print the transmission type and the number of doors.
* Calls the overridden `displayInfo()` function on `car1` and `car2` instances.

For more information about class inheritance and overriding class behavior, see [Inheritance](inheritance.md).

## Classes practice

Exercises TBD.

## Interfaces

But what if you want to share some code across multiple classes? Since classes are limited by single inheritance, this could
be challenging to implement using classes alone. However, in Kotlin you can use interfaces to solve this problem.

Interfaces are similar to classes, but they have some differences:

* You can’t create an instance of an interface. Therefore, they don't have a constructor or header.
* Their functions and properties are implicitly `open` by default.

You use them when you want to define a set of functions and properties that classes can inherit and implement later.

To declare an interface, use the `interface` keyword:

```kotlin
interface PaymentMethod
```

### Interface inheritance

Interfaces support multiple inheritance so a class can inherit from multiple interfaces at once. First, let's consider
the scenario where a class inherits from one interface.

To create a class that inherits from an interface, add a colon after your class header followed by the name of the interface
that you want to inherit from. You don't use parentheses `()` after the interface name, because interfaces don't have a 
constructor:

```kotlin
class CreditCardPayment : PaymentMethod
```

For example:

```kotlin
interface PaymentMethod {
    // No open keyword needed because functions are inheritable by default
    fun initiatePayment(amount: Double): String
}

class CreditCardPayment(val cardNumber: String, val cardHolderName: String, val expiryDate: String) : PaymentMethod {
    override fun initiatePayment(amount: Double): String {
        // Simulate processing payment with credit card
        return "Payment of $$amount initiated using Credit Card ending in ${cardNumber.takeLast(4)}."
    }
}

fun main() {
    val paymentMethod = CreditCardPayment("1234 5678 9012 3456", "John Doe", "12/25")
    println(paymentMethod.initiatePayment(100.0))
    // Payment of $100.0 initiated using Credit Card ending in 3456.
}
```

In the example:
* `PaymentMethod` is an interface that has an `initiatePayment()` function without an implementation.
* `CreditCardPayment` is a class that inherits from the `PaymentMethod` interface.
* The `CreditCardPayment` class overrides the inherited `initiatePayment()` function.

* `paymentMethod` is an instance of the `CreditCardPayment` class.
* The overridden `initiatePayment()` function is called on the `paymentMethod` instance with a parameter of `100.0`.

To create a class that inherits from multiple interfaces, add a colon after your class header followed by the name of the interfaces
that you want to inherit from separated by a comma:

```kotlin
class CreditCardPayment : PaymentMethod, PaymentType
```

For example:

```kotlin
interface PaymentMethod {
    fun initiatePayment(amount: Double): String
}

interface PaymentType {
    val paymentType: String
}

class CreditCardPayment(val cardNumber: String, val cardHolderName: String, val expiryDate: String) : PaymentMethod, PaymentType {
    override fun initiatePayment(amount: Double): String {
        // Simulate processing payment with credit card
        return "Payment of $$amount initiated using Credit Card ending in ${cardNumber.takeLast(4)}."
    }
    
    override val paymentType: String = "Credit Card"
}

fun main() {
    val paymentMethod = CreditCardPayment("1234 5678 9012 3456", "John Doe", "12/25")
    println(paymentMethod.initiatePayment(100.0))
    // Payment of $100.0 initiated using Credit Card ending in 3456.

    println("Payment is by ${paymentMethod.paymentType}")
    // Payment is by Credit Card
}
```

In the example:
* `PaymentMethod` is an interface that has an `initiatePayment()` function without an implementation.
* `PaymentType` is an interface that has a `paymentType` function that isn't initialized.
* `CreditCardPayment` is a class that inherits from the `PaymentMethod` and `PaymentType` interfaces.
* The `CreditCardPayment` class overrides the inherited `initiatePayment()` function and the `paymentType` property.

* `paymentMethod` is an instance of the `CreditCardPayment` class.
* The overridden `initiatePayment()` function is called on the `paymentMethod` instance with a parameter of `100.0`.
* The overridden `paymentType` property is accessed on the `paymentMethod` instance.

For more information about interfaces and interface inheritance, see [Interfaces](interfaces.md).

## Delegation

Interface inheritance is useful but if your interface contains many functions, child classes may end up with a lot of 
boilerplate code. When you only want to override a small part of your parent class's behavior, this can feel like you are
having to repeat yourself a lot.

> Boilerplate code is a chunk of code that is reused with little or no alteration in multiple parts of a software project.
> 
{type="tip"}

For example, let's say that you have an interface called `Drawable` that contains a number of functions and one property
called `color`:

```kotlin
interface Drawable {
    fun draw()
    fun resize()
    val color: String?
}
```

You create a class called `Circle` which inherits from the `Drawable` interface and provides implementations for all of
its inherited member functions:

```kotlin
class Circle : Drawable {
    override fun draw() {
        TODO("Not yet implemented")
    }
    override fun resize() {
        TODO("Not yet implemented")
    }
}
```

If you wanted to create a child class of the `Circle` class which had the same behavior **except** for the value of the
`color` property, you still need to add implementations for each member function of the `Circle` class:

```kotlin
class RedCircle(val circle: Circle) : Circle {
    
    // Start of boilerplate code
    override fun draw() {
        circle.draw()
    }
    override fun resize() {
        circle.resize()
    }
    // End of boilerplate
    override val color = "red"
}
```

You can see here that if you have a large number of member functions in the `Drawable` interface, the amount of boilerplate
code in the `RedCircle` class can be very large. However, there is an alternative.

In Kotlin, you can use delegation to delegate the implementation of the interface to an instance of a class. For example,
you can create an instance of the `Circle` class and delegate the implementations of the member functions of the `Circle`
class to this instance. To do this, use the `by` keyword. For example:

```kotlin
class RedCircle(param : Circle) : Circle by param
```

Here, `param` is the name of the instance of the `Circle` class that the implementations of member functions are delegated to.

Now you don't have to add implementations for the inherited member functions in the `RedCircle` class. The compiler does
this for you automatically from the `Circle` class. This saves you from having to write a lot of boilerplate code. Instead,
you add code only for the behavior you want to change for your child class. 

For example, if you want to change the value of the `color` property:

```kotlin
class RedCircle(param : Circle) : Circle by param {
    // No boilerplate code!
    override val color = "red"
}
```

If you want to, you can also override the behavior of an inherited member function in the `RedCircle` class, but now
you don't have to add new lines of code for every inherited member function.

For more information about delegation, see [Delegation](delegation.md).

## Interface and delegation practice

## Objects

In Kotlin, you can use **objects** to declare a class with a single instance. In a sense, you declare the class and create
the single instance at the same time. You can use objects in two ways:

* Object declarations.
* Object expressions.

### Object declarations

Object declarations are useful when you want to create a class that you will only ever have one instance for. This class
may be used as a single reference point for your program or to coordinate behavior across a system.

> A class that has only one instance that is easily accessible is called a **singleton**.
>
{type="note"}

Objects in Kotlin are lazy. They are only created once they are accessed. Kotlin also ensures that all objects are created
in a thread-safe manner so that you don't have to check this manually yourself.

To create an object, use the `object` keyword:

```kotlin
object DoAuth {}
```

Following the name of your `object`, add any properties or member functions within the object body defined by curly braces `{}`.

> Objects can't have constructors so don't have a header like a class does.
>
{type="note"}

For example, let's say that you wanted to create an object called `DoAuth` that is responsible for authentication:

```kotlin
object DoAuth {
    fun takeParams(username: String, password: String) {
        println("input Auth parameters = $username:$password")
    }
}

fun main(){
    // The object is created when the takeParams() function is called
    DoAuth.takeParams("coding_ninja", "N1njaC0ding!")
    // input Auth parameters = coding_ninja:N1njaC0ding!
}
```

The object has a member function called `takeParams` that accepts `username` and `password` variables as parameters
and returns a string to console. The `DoAuth` object is only created when the function is called for the first time.

> Objects can inherit from classes and interfaces. For example:
> 
> ```kotlin
> interface Auth {
>     fun takeParams(username: String, password: String)
> }
>
> object DoAuth : Auth {
>     override fun takeParams(username: String, password: String) {
>         println("input Auth parameters = $username:$password")
>     }
> }
> ```
>
{type="note"}

#### Data objects

To make it easier to print the contents of an object declaration, Kotlin has **data** objects. Similar to data classes,
which you learned about in the beginners tour, data objects come automatically with additional member functions: 
`.toString()` and `.equals()`.

> Unlike data classes, data objects do not come automatically with the `.copy()` member function because they only have
> a single instance that can't be copied.
>
{type ="note"}

To create a data object, use the same syntax as for object declarations but prefix it with the `data` keyword:

```kotlin
data object AppConfig {}
```

For example:

```kotlin
data object AppConfig {
    var appName: String = "My Application"
    var version: String = "1.0.0"
}

fun main() {
    println(AppConfig)
    // App config
}
```

For more information about data objects, see [](object-declarations.md#data-objects).

#### Companion objects

In Kotlin, a class can also have an object: a **companion** object. You can only have **one** companion object per class.
A companion object is created only when its class is referenced for the first time.

Any properties or functions declared inside a companion object are shared across all class instances.

To create a companion object within a class, use the same syntax for an object declaration but prefix it with the `companion`
keyword:

```kotlin
companion object Bonger {}
```

> A companion object doesn't have to have a name.
> 
{type="note"}

To access any properties or functions of the companion object, reference the class name. For example:

```kotlin
class BigBen {
    companion object Bonger {
        fun getBongs(nTimes: Int) {
            repeat(nTimes) { print("BONG") }
            }
        }
    }
}

fun main() {
    // Companion object is created because the class is referenced for the first time.
    BigBen.getBongs(12)
    // BONG BONG BONG BONG BONG BONG BONG BONG BONG BONG BONG BONG 
}
```

This example creates a class called `BigBen` that contains a companion object called `Bonger`. The companion object
has a member function called `getBongs()` that accepts an integer and prints `"BONG"` to the console the same number of times
as the integer.

In the `main()` function, the `getBongs()` function is called by referring to the class name. The companion object is created
at this point. The `getBongs()` function is called with parameter 12.

For more information about companion objects, see [](object-declarations.md#companion-objects).

### Object expressions

Objects can also be created without a name, called anonymous objects. Object expressions are useful when you want to declare
a class and create an instance but not give it a name. Usually this is because you only need to access it within one function.
The most common use case for object expressions is to hold a structure of properties.

Unlike object declarations, object expressions aren't used to declare classes with a single instance. Instead, they are
commonly used for ad hoc creation of a class instance. Every time an object expression is evaluated, a new instance is created.

To create an object expression, the syntax is the same for an object declaration, but you don't give it a name:

```kotlin
object {}
```

For example, let's say that you wanted to create a function called `rentPrice()` that you can use to calculate the cost
of renting a venue depending on the days chosen. You can use an object expression to hold properties containing information
about the rates on different days:

```kotlin
fun rentPrice(standardDays: Int, festivityDays: Int, specialDays: Int): Unit {

    val dayRates = object {
        var standard: Int = 30 * standardDays
        var festivity: Int = 50 * festivityDays
        var special: Int = 100 * specialDays
    }

    val total = dayRates.standard + dayRates.festivity + dayRates.special

    print("Total price: $$total")

}

fun main() {
    // The object is created when the rentPrice() function is called
    rentPrice(10, 2, 1)
    // Total price: $500
}
```

The `rentPrice()` function accepts variables representing three different types of days: `standardDays`, `festivityDays`,
and `specialDays`, and returns a string containing the total price for the number of days selected.

Within the `rentPrice()` function body, there is an object expression that contains three properties: `standard`, `festivity`,
and `special`. These properties are initialized based on the provided function parameters.

The object expression is assigned to the `dayRates` variable so it's properties can be accessed later by the `total`
variable to finish the calculation.

For more information about objects, see [Object expressions and declarations](inheritance.md).

## Objects practice

## Special classes

In addition to normal classes and data classes, Kotlin has special types of classes designed for various purposes, such 
as restricting specific behavior or reducing the performance impact of creating small objects.

### Abstract classes

Abstract classes in Kotlin are classes that have their properties and member functions declared but not necessarily their
behavior. Similar to interfaces, the intention is that any abstract member functions and properties are overridden and 
defined by another class.

To create an abstract class, use the `abstract` keyword:

```kotlin
abstract class Animal
```

And similarly, for an abstract function:

```kotlin
abstract fun makeSound()
```

For example, let's say that you want to create an abstract class called `Product` that you can create child classes from
to define different product categories:

```kotlin
abstract class Product( val name: String, var price: Double ) {
    // Abstract function to get the product category
    abstract fun category(): String

    // A function that can be shared by all products
    fun productInfo(): String {
        return "Product: $name, Category: ${category()}, Price: $price"
    }
}
```

In the abstract class:
* The constructor has two parameters for the `name` and the `price` of the product.
* There is an abstract function that returns the product category as a string.
* There is a function that prints information about the product.

Let's create a child class for electronics:

```kotlin
class Electronic( name: String, price: Double, val warranty: Int ) : Product(name, price) {
    override fun category(): String {
        return "Electronic"
    }
}
```

The `Electronic` class:
* Inherits from the `Product` abstract class.
* Has an additional parameter in the constructor: `warranty`, which is specific to electronics.
* Overrides the `category()` function to return the string `"Electronic"`.

Now, you can use these classes like this:

```kotlin
fun main() {
    // Create instance of the Electronic class
    val laptop = Electronic(name = "Laptop", price = 1000.0, warranty = 2)

    println(laptop.productInfo())
    // Product: Laptop, Category: Electronic, Price: 1000.0
}
```

So if you have the choice between using an interface or an abstract class in your code, which should you use? Abstract
classes offer the following benefits:

* Abstract classes can have a constructor.
* Abstract classes can contain state.

Abstract classes have a very narrow use case. In most cases, it is better to use an interface.

For more information about abstract classes, see [Abstract classes](classes.md#abstract-classes).

### Sealed classes

There may be times when you want to restrict inheritance. You can do this with sealed classes. Once you declare that a class
is sealed, you can only create child classes from it within the same package. It's not possible to inherit from the sealed
class outside of this scope.

> A package is a collection of code with related classes and functions, typically within a directory. To learn more about
> packages in Kotlin, see [Packages and imports](packages.md).
> 
{type="note"}

To create a sealed class, use the `sealed` keyword:

```kotlin
sealed class Mammal
```

Sealed classes are particularly useful when combined with a `when` expression. By using a `when` expression, you can
define the behavior for all possible child classes. For example:

```kotlin
sealed class Mammal(val name: String)

class Cat(val catName: String) : Mammal(catName)
class Human(val humanName: String, val job: String) : Mammal(humanName)

fun greetMammal(mammal: Mammal): String {
    when (mammal) {
        is Human -> return "Hello ${mammal.name}; You're working as a ${mammal.job}"
        is Cat -> return "Hello ${mammal.name}"   
    }
}

fun main() {
    println(greetMammal(Cat("Snowy")))
    // Hello Snowy
}
```

In the example:
* There is a sealed class called `Mammal` that has the `name` parameter in the constructor.
* The `Cat` class inherits from the `Mammal` sealed class and uses the `name` parameter from the `Mammal` class as the
`catName` parameter in its own constructor.
* The `Human` class inherits from the `Mammal` sealed class and uses the `name` parameter from the `Mammal` class as the
`humanName` parameter in its own constructor. It also has the `job` parameter in its constructor.
* The `greetMammal()` function accepts an argument of `Mammal` type and returns a string.
* Within the `greetMammal()` function body, is a `when` expression that uses the [`is` operator](typecasts.md#is-and-is-operators) to check the type of `mammal` and decide which action to perform.
* The `main()` function calls the `greetMammal()` function with an instance of the `Cat` class and `name` parameter called `Snowy`.

For more information about sealed classes and their recommended use cases, see [Sealed classes and interfaces](sealed-classes.md).

### Enum classes

Enum classes are useful when you want to represent a finite set of distinct values in a class. An enum class contains enum
constants, which are themselves instances of the enum class.

To create an enum class, use the `enum` keyword:

```kotlin
enum class State
```

Let's say that you want to create an enum class that contains the different states of a process. Each enum constant must
be separated by a comma `,`:

```kotlin
enum class State {
    IDLE, RUNNING, FINISHED
}
```

The `State` enum class has enum constants: `IDLE`, `RUNNING`, and `FINISHED`. To access an enum constant, use the
class name followed by a `.` and the name of the enum constant:

```kotlin
val state = State.RUNNING
```

You can use this enum class with a `when` expression to define the action to take depending on the value of the enum constant:

```kotlin
enum class State {
    IDLE, RUNNING, FINISHED
}

fun main() {
    val state = State.RUNNING
    val message = when (state) {
        State.IDLE -> "It's idle"
        State.RUNNING -> "It's running"
        State.FINISHED -> "It's finished"
    }
    println(message)
    // It's running
}
```

Enum classes can have properties and member functions just like normal classes. 

For example, let's say that you are working with HTML and you want to create an enum class that contains some colors. 
You want these colors to each have a property, let's call it `rgb`, that contains their RGB value as a hexadecimal. 
When creating the enum constants, you must initialize it with this property:

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF),
    YELLOW(0xFFFF00)
}
```

> Kotlin stores hexadecimals as integers so the `rgb` property has `Int` type and not `String` type.
>
{type="note"}

To add a member function to this class, separate it from the enum constants with a semicolon `;`:

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF),
    YELLOW(0xFFFF00);

    fun containsRed() = (this.rgb and 0xFF0000 != 0)
}

fun main() {
    val red = Color.RED
    
    // Calls containsRed() function on enum constant
    println(red.containsRed())
    // true

    // Calls containsRed() function on enum constants via class names
    println(Color.BLUE.containsRed())
    // false
    println(Color.YELLOW.containsRed())
    // true
}
```

In this example, the `containsRed()` member function accesses the value of the enum constant's `rgb` property using the
`this` keyword, and checks if the hexadecimal value contains `FF` as its first bits to return a boolean value.

For more information about enum classes, see [Enum classes](enum-classes.md).

### Inline value classes

Sometimes in your code, you may want to create small objects from classes and use them only briefly. This approach can
have a performance impact. Inline value classes are a special type of class that avoid this performance impact. However,
they can only contain values.

To create an inline value class, use the `value` keyword and the `@JvmInline` annotation:

```kotlin
@JvmInline
value class Email
```

> The `@JvmInline` annotation is used by Kotlin to know to optimize the code when it is compiled. To learn more about
> annotations, see [Annotations](annotations.md).
> 
{type="note"}

An inline value class **must** have a single property initialized in the class header.

Let's say that you want to create a class that collects an email address:

```kotlin
// The value property is initialized in the class header.
@JvmInline
value class Email(val value: String)

fun sendEmail(email: Email) {
    println("Sending email to ${email.value}")
}

fun main() {
    val myEmail = Email("example@example.com")
    sendEmail(myEmail)
    // Sending email to example@example.com
}
```

In the example:
* `Email` is an inline value class that has one property in the class header: `value`.
* The `sendEmail()` function accepts objects with type `Email` and prints a string to standard output.
* The `main()` function:
    * Creates an instance of the `Email` class called `email`.
    * Calls the `sendEmail()` function on the `email` object.

By using an inline value class, the class is inlined and used directly in your code without creating an object. This can 
significantly reduce memory footprint and improve the runtime performance of your code.

For more information about inline value classes, see [Inline value classes](inline-classes.md).

## Special classes practice

## Next step

