[//]: # (title: Intermediate Properties)

In the beginner's tour, you learned how properties are used to declare characteristics of class instances and how to access
them. This chapter digs deeper into how properties work in Kotlin and explores other ways that you can use them in your code.

## Backing fields

Under the hood, properties are set using a `set()` function and retrieved using a `get()` function. These functions work
with the actual value of the property, known as a **backing field**. In Kotlin, the backing field is automatically hidden
in Kotlin, but you can access it by using the keyword `field`. For example, this code:

```kotlin
class Contact(val id: Int, var email: String) {
    val category: String = ""
}
```

Is the same as this code:

```kotlin
class Contact(val id: Int, var email: String) {
    val category: String = ""
        get() = field
        set(value) {
            field = value
        }
}
```

The `get()` function retrieves the property value from the field: `""`.
The `set()` function accepts `value` as a parameter and assigns it to the field, where `value` is `""`. 

It's important to note that you can't explicitly call the `get()` and `set()` functions on properties in Kotlin. By default,
Kotlin provides `get()` and `set()` functions for properties, but you can create your own custom `get()` and `set()` functions.

> `get()` and `set()` functions are also called getters and setters. 
> 
{type="tip"}

Access to the underlying property value is especially useful when you want to perform additional logic
in your `get()` or `set()` functions without causing an infinite loop. For example, you have a 
`Person` class with a property called `name`:

```kotlin
class Person {
    var name: String = ""
}

```

You want to ensure that the first letter of the `name` property is capitalized, so you create a custom `set()` function
that uses the [`.replaceFirstChar()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/replace-first-char.html) 
and [`.uppercase()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/uppercase-char.html) extension functions. 
However, if you refer to the property directly in your `set()` function, you create an infinite loop and see a `StackOverflowError`
at runtime:

```kotlin
class Person {
    var name: String = ""
        set(value) {
            // This causes a runtime error
            name = value.replaceFirstChar { firstChar -> firstChar.uppercase() }
        }
}

fun main() {
    val person = Person()
    person.name = "kodee"
    println(person.name)
}
```
{validate="false"}

To fix this, you can use the backing field in your `set()` function instead:

```kotlin
class Person {
    var name: String = ""
        set(value) {
            field = value.replaceFirstChar { firstChar -> firstChar.uppercase() }
        }
}

fun main() {
    val person = Person()
    person.name = "kodee"
    println(person.name)
    // Kodee
}
```

Backing fields are also useful when you want to add logging, send notifications when a property value changes,
or use additional logic that compares the old and new property values.

<!-- Mention backing properties? Needs intro on visibility modifiers -->

For more information about backing fields, see [Backing fields](properties.md#backing-fields).

## Extension properties

Just like extension functions, there are also extension properties. Extension properties allow you to add new properties
to existing classes without modifying their source code. However, extension properties in Kotlin do **not** have backing
fields. This means that Kotlin doesn't provide default `get()` and `set()` functions automatically. You have to write them
yourself. Additionally, the lack of a backing field means that they can't hold any state.

To declare an extension property, write the name of the class that you want to extend followed by a `.` and the name of
your property. Just like with normal class properties, you need to declare a receiver type for your property. 
For example:

```kotlin
val String.lastChar: Char
```

Extension properties are most useful when you want a property to contain a computed value without using inheritance.
You can think of extension properties working like a function that has only a single parameter: the receiver object.
For example, let's say that you have a data class called `Person` that has two properties: `firstName`, `lastName`.

```kotlin
data class Person(val firstName: String, val lastName: String)
```

You want to be able to access the person's full name without modifying the `Person` data class or inheriting from it.
You can do this by creating an extension property with a custom `get()` function:

```kotlin
data class Person(val firstName: String, val lastName: String)

// Extension property to get the full name
val Person.fullName: String
    get() = "$firstName $lastName"

fun main() {
    val person = Person(firstName = "John", lastName = "Doe")
    
    // Use the extension property
    println(person.fullName)
    // John Doe
}
```

> It's important to note that extension properties can't override existing properties of a class.
> 
{type="note"}

Just like with extension functions, we use extension properties widely in the Kotlin standard library. For example,
consider the [`lastIndex` property](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/last-index.html) for a `CharSequence`.

## Delegated properties

You already learned about delegation in the [Classes](kotlin-tour-intermediate-classes.md#delegation) chapter. You can
also use delegation with properties to delegate their `get()` and `set()` functions to another object. This is useful
when you have more complex requirements for storing properties that a simple backing field can't handle, such as storing
values in a database table, browser session, or map. Using delegated properties also reduces boilerplate code because the
logic for getting and setting your properties is contained only in the object that you delegate to.

The syntax is similar to using delegation with classes but operates on a lower level. Declare your property, followed by
the `by` keyword and the object you want to delegate to. For example:

```kotlin
val displayName: String by Delegate
```

Suppose you want to have a computed property, like a user's display name, that is calculated only once because
the operation is expensive and your application is performance sensitive. You can use a delegated property to cache the
display name so that it is only computed once but can be accessed anytime without performance impact.

First, you need to create the object to delegate to that contains the cached value and defines its `get()` function.
In this case, the object will be an instance of the `CachedStringDelegate` class:

```kotlin
class CachedStringDelegate {
    var cachedValue: String? = null
}
```

The `cachedValue` is the delegated property. Every delegated property **must** have a `getValue()` operator function. 
If the property is mutable, you must also have a `setValue()` operator function.

By default, the `getValue()` and `setValue()` functions have the following construction:

```kotlin
operator fun getValue(thisRef: Any?, property: KProperty<*>): String {}

operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {}
```

In these functions:

* The `thisRef` parameter refers to the object containing the property. In this example, it's an instance of 
the `CachedStringDelegate` class. By default, the type is set to `Any?` but you may need to declare a more specific type.
* The `property` parameter refers to the property that is get or set. You can use this parameter to access information
like the property's name or type. By default, the type is set to `KProperty<*>`, which is a reflection object. You don't
need to worry about changing this in your code.

> We won't discuss reflection in this tour because it's not necessary for you to learn how to use delegated properties. 
> However, if you'd like to learn more about reflection, see [Reflection](reflection.md).
> 
{type="tip"}

The `getValue()` function has a return type of `String` by default, but you can adjust this if you want.

The `setValue()` function has an additional parameter `value`, which is used to hold the new value that's assigned to the
property.

Within the `CachedStringDelegate` class, add the behavior that you want from the `get()` function of the `cachedValue` property
to the `getValue()` operator function body:

```kotlin
class CachedStringDelegate {
    var cachedValue: String? = null

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        if (cachedValue == null) {
            cachedValue = "Default Value"
            println("Computed and cached: $cachedValue")
        } else {
            println("Accessed from cache: $cachedValue")
        }
        return cachedValue ?: "Unknown"
    }
}
```

The `getValue()` function checks whether the `cachedValue` property is `null`. If it is, the function assigns the value of
`"Default value"` as well as printing a string for logging purposes. If the `cachedValue` property isn't `null`, so it's already been computed,
another string is printed for logging purposes. Finally, the function uses a safe call to return either the cached value
or `"Unknown"` if the value is `null`.

Now you can delegate the property that you want to cache (`val displayName`) to an instance of the `CachedStringDelegate` class:

```kotlin
import kotlin.reflect.KProperty

class CachedStringDelegate {
    var cachedValue: String? = null

    operator fun getValue(thisRef: User, property: KProperty<*>): String {
        if (cachedValue == null) {
            cachedValue = "${thisRef.firstName} ${thisRef.lastName}"
            println("Computed and cached: $cachedValue")
        } else {
            println("Accessed from cache: $cachedValue")
        }
        return cachedValue ?: "Unknown"
    }
}

//sampleStart
class User(val firstName: String, val lastName: String) {
    val displayName: String by CachedStringDelegate()
}

fun main() {
    val user = User("John", "Doe")
    
    // First access computes and caches the value
    println(user.displayName)  
    // Computed and cached: John Doe

    // Subsequent accesses retrieve the value from cache
    println(user.displayName)  
    // Accessed from cache: John Doe
}
//sampleEnd
```
{runnable="true"}

<!-- Figure out why "John Doe" is also printed twice in code sample -->

This example:
* Creates a `User` class that has two properties in the header: `firstName`, and `lastName`, and one property in the
class body `displayName`.
* Delegates the `displayName` property to an instance of the `CachedStringDelegate` class.
* Creates an instance of the `User` class called `user`.
* Prints the result of accessing the `displayName` property on the `user` instance.

Note that in the `getValue()` function, the type for the `thisRef` parameter is narrowed from `Any?` type to the object
type: `User`. This is so that the compiler can access the `firstName` and `lastName` properties of the `User` class.

### Standard delegates

The Kotlin standard library provides some useful delegates for you so you don't have to always create yours from scratch.
If you use one of these delegates, you don't have to define a `getValue()` and `setValue()` function because these are
automatically provided by the standard library.

#### Lazy properties

To initialize a property only when it's first accessed, use a lazy property. The standard library provides the `Lazy`
interface to use as a delegate. To create an instance of the `Lazy` interface, use the `lazy()` function by providing it
with a lambda expression to execute when the `get()` function is called for the first time. Any further calls of the `get()`
function return the same result that was provided on the first call.

For example:

```kotlin
class Database {
    fun connect() {
        println("Connecting to the database...")
    }

    fun query(sql: String): List<String> {
        return listOf("Data1", "Data2", "Data3")
    }
}

val databaseConnection: Database by lazy {
    val db = Database()
    db.connect()
    db
}

fun fetchData() {
    val data = databaseConnection.query("SELECT * FROM data")
    println("Data: $data")
}

fun main() {
    // First time accessing databaseConnection
    fetchData()
    // Connecting to the database...
    // Data: [Data1, Data2, Data3]

    // Subsequent access uses the existing connection
    fetchData()
    // Data: [Data1, Data2, Data3]
}
```
{runnable="true"}

In this example:

* There is a `Database` class with `connect()` and `query()` member functions. 
* The `connect()` function prints a string to console, and the `query()` function accepts an SQL query and returns a list.
* There is a `dataBaseConnection` property that is a top-level lazy property.
* The lambda expression provided to the `lazy()` function:
  * Creates an instance of the `Database` class.
  * Calls the `connect()` member function on this instance (`db`).
  * Returns the instance.
* There is a `fetchData()` function that:
  * Creates an SQL query by calling the `query()` function on the `dataBaseConnection` property.
  * Assigns the SQL query to the `data` variable.
  * Prints the `data` variable to console.
* The `main()` function calls the `fetchData()` function. The first time, it is called, the lazy property is initialized.
The second time, the same result is returned as the first call.

Lazy properties are useful not only when initialization is resource-intensive but also when a property might not be used
in your code. Additionally, lazy properties are thread-safe by default, which is particularly beneficial if you are working
in a concurrent environment.

To learn more about lazy properties, see [Lazy properties](delegated-properties.md#lazy-properties).

#### Observable properties

To monitor whether the value of a property changes, use an observable property. An observable property is useful when
you want to detect a change in the property value and use this knowledge to trigger a reaction. The standard library provides
the `Delegates` object to use as a delegate.

To create an observable property, you must first import `kotlin.properties.Delegates`. Then, use the `Delegates.observable()` function
and provide it with a lambda expression to execute whenever the property changes.

For example:

```kotlin
import kotlin.properties.Delegates

class Thermostat {
    var temperature: Double by Delegates.observable(20.0) { _, old, new ->
        if (new > 25) {
            println("Warning: Temperature is too high! ($old°C -> $new°C)")
        } else {
            println("Temperature updated: $old°C -> $new°C")
        }
    }
}

fun main() {
    val thermostat = Thermostat()
    thermostat.temperature = 22.5  
    // Temperature updated: 20.0°C -> 22.5°C
  
    thermostat.temperature = 27.0  
    // Warning: Temperature is too high! (22.5°C -> 27.0°C)
}
```

In this example:

* There is a `Thermostat` class that contains an observable property: `temperature`.
* The `Delegates.observable()` function accepts `20.0` as a parameter and uses it to initialize the property.
* The lambda expression provided to the `Delegates.observable()` function:
  * Has three parameters:
    * `_`, which refers to the property itself.
    * `old`, which is the old value of the property.
    * `new`, which is the new value of the property.
  * Checks if the `new` parameter is greater than `25`, and depending on the result, prints a string to console.
* The `main()` function:
  * Creates an instance of the `Thermostat` class called `thermostat`.
  * Updates the value of the `temperature` property of the instance to `22.5`, which triggers a print statement with a temperature update.
  * Updates the value of the `temperature` property of the instance to `27.0`, which triggers a print statement with a warning.

Observable properties are useful not only for logging and debugging purposes. You can also use them for use cases like
updating a UI or to perform additional checks, like verifying the validity of data.

To learn more about observable properties, see [Observable properties](delegated-properties.md#observable-properties).

## Next step