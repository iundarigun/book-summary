# Effective Java

## Create and destroy objects

### Item 1: Consider static factory methods instead of constructors
A class can give us a public `static factory method`, that returns an instance of the class. Don't confuse with design pattern `Factory Method`.

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

Pros:
- That methods have a name, different than construtors. Sometimes the parameters of the constructor does not describe the return object. 
- An other problem with constructor is we can have only one constructor with the same signature, but we don't have the same restriction with `static factory method`, because we can give different names describing the behaivor.
- It is not necessary create a new object for every invocation, different than constructor. This allows `immutable classes`, that uses pre-construct objects or caches. Performance increases if these objects are frequently used. 
- It allows the class be a `singleton` too.
- They can return any subtype of the return, returning different objects for different input parameters. An API can return objects without become public classes. 

Contras:
- A limitation of use classes without public constructors is not allow subclasses. Perhaps is a good think, to encourage programmers to use composition instead of inheritance.
- May be dificult to find, to discover how instance a class without public constructors

Common names for `static factory method`:
- from: Conversion from a single parameter.
- of: Agregation method of a couple of values.
- valueOf: More verbose for `from` or `of`
- instance/getInstance: Instance describe for the parameters.
- create/newInstance: Like previous, but garanting new instance for each invocation.
- getType (Type is the class name)
- newType (Type is the class name)
- type (Type is the class name)

### Item2: Use a builder when have too many parameters in the constructor

Static factories and constructor have a limitation when we have a huge number of parameters, mainly if most are optional, because we need to create a good number of constructors or static factory methods.

We have some alternatives:
- Telescoping constructor: One constructor with the mandatory, and other constructors for mandatory and one/two or more of optionals. It is not a good decision for the number of constructor sometimes we will need to pass values for parameters that we are not interested
- JavaBeans: A constructor without parameters, and a setter for each parameter. The problem is the object may be in an inconsistent state, and not allow immutable classes.

The solution is a builder. Client invokes a constructor with mandatory parameters and receive a builder. This object allows set the optional parameters. At last, the client invokes build method to receive an instance.

### Item 4:  Implements non-instantiation using private constructors

Sometimes, your class is only a group of static fields and methods. Don't use `abstract` modifier for this, because users can be thing that you design for inheritance. Collateral efect, this forbiden subclasses.

### Item 6: Avoid creating unnecessary objects

You can avoid new objects using static factory methods, because constructor must to create a new object. 

For this motive, prefer primitive type over boxing primitives, and beware of involuntary autoboxing.

### Item 7: Eliminate obsolete object references

The best way to elminate a obsolete object reference is leave it out of scope. This happens naturally defining it on the most limited scope as possible.

Be careful with cache object. It is easy to forget them.

### Item 9: Prefer try-with-resources to try-finally

In the past, `try-finally` was the best way to garantee that a resource will be closed correctly. But if we have nested closable items, the code is a mess.

Java 7 gives to us `try-with-resources`, that we can used to close objects that implement the interface `AutoCloseable`. Not only is more readable, but give to us better ways to identify bugs.

## Methods Common to All Objects
Object is not an abstract class, but was projected to inheritance

### Item 10: Obey the general contract when overriding equals

The easyest way to avoid problems is not overriding `equals` method. This is the wright thing to do when:
- Each instance of the class is exclusive.
- It is not necessary a test of "logical equals"
- When a superclass already overrides `equals` with the wright behaivor
- When the class is private or package-private and you are sure that never will be invoke

When a class has a "logical equals" different for the object identity, for example **value classes**, like Integer or String. 

The contract will be follow:
- Reflexive: For any value non-null of `x`, `x.equals(x)` must to be true
- Symmetrical: For non-null values of `x` and `y`, if `x.equals(y)` is true only if `y.equals(x)` is true. 
- Transitive: Fior non-null values of  `x`, `y` and `z`, if `x.equals(y)` is true and `y.equals(z)` is true, then `x.equals(z)`
- Consistent: For non-null values of `x` and `y`, for countless invocations of `x.equals(y)` must to return the same value.
- For any value non-null of `x`, `x.equals(null)` must to be false.

We haven't a way to extends an instanciable class adding a value component and keep the equals contract. This point can break the transitive rule. 

For the primitive atributes, except `float` and `double`, use the operator `==` to compare. For `float` and `double`, use `Float.Compare(float, float)` and `Double.compare`. For objects atributes, use equals. To avoid `NullPointerException` for nullable atributes, use `Objects.equals(Object, Object)`.


### Item 11: Always override hashCode when you override equals

If you don't it, you will break the `hashCode` contract.
- The countless invocations of hashCode during the execution of an application, must to return the same value.
- If two objects are equals with `equals` method, the return of the two hashCode must to be the same. 

You must to exclude each atribute to the hashCode calculation that you exluded from equals.

Use the format:
```java
@Override
public int hashCode() {
    int result = field1.hashCode();
    result = 31 * result + field2.hashCode();
    result = 31 * result + field3.hashCode();
    // etc

    return result;    
}
```

### Item 12: Always override toString
The general contract of `toString` tell us that the returned string must to be a concise but informative representation, readable to a person.

Giving a good implementation makes more useful and systems are easier to debug. If is possible, `toString` must to return all of informations. If the class is big, this may be impossible, but we can return a summary.

Exceptions:
- Static classes
- Enums

### Item 14: Consider implementing Comparable

The `compareTo` method is not declared in Object, but is the only method of the `Comparable` interfaace. It is similar of `equals`, except for admit comparations of order, not only equality. Implementing `Comparable` indicates that instances have _natural order_.

Returns negative integer, zero or positive integer if the object is less than, equals, or great than the parameter object. Rules:
- For non-null objects, `x.compareTo(y) == -1 * y.compareTo(x)`.
- For non-null objects, if `x.compareTo(y) > 0` and `y.compareTo(z) > 0`, so `x.compareTo(z) > 0`.
- For non-null objects, if `x.compareTo(y) == 0`, so `y.compareTo(x) == 0`.
- It is recommended that if `x.compareTo(y) == 0`, `x.equals(y)` returns true.

## Classes and Interfaces

### Item 15: Minimize the accessibility of classes and members

A good-design component hides details of implementation, separating the API and the implementation. This concept is know as encapsulations, and is an elemantary principle of software design.

The basic principle is simple: make each class or or member as inaccessible as possible. Use the less access level.

- Atributes of public classes must to be not public. If you let public, you can not control values.
- Classes with mutable atributes are not thread-safe.


### Item 16: In public classes, use accessor methods, not public fields
This allow you keep flexibility to change internally without afect public contracts.

### Item 17: Minimize mutability
An inmutable class is a class that instances can not be modified. Theses classes are easy to design, implement and use, with less errors and more safes:
- Don't create methods to change the object status (ex. _setters_)
- Let the class as _final_ to guarantee that anyone can not extend it
- Make all fields _final_
- Make all fields _private_. Technically, you can let _public_ and _final_ but is not recommended (see item 15 and 16)
- Grant exclusive access to any mutable object. Never initialize these components using an object from client, never return on _getter_. Use defensive copies instead.

Inmutable objects are simple. An inmutable object has exactly one state, the creation state. They are the easyest way to guarantee thread-safe. We can share inmutable objects and it is a good practice. We can incentive it creating common final static public instances, like `BigDecimal.ZERO`. 

If for any reason, you can not create your class inmutable, restrict mutability all you can.

### Item 18: Favor composition over inheritance

Inheritance is not always the best way to reuse code. It can be safe if use under the same package, where the programmer has all control. But remember that: Inheritance not allow encapsulation.

Instead of inheritance, create a new class with a private field with a reference to the "main" class. When you want invoke a method of this class, you can create a method on the new class and make it invoke the method of the reference of the new class, faking a inheritance. Sometimes this estrategy can be called as _delegation_ 


### Item 19: Design and document for inheritance or else prohibit it

### Item 20: Prefer interfaces to abstract classes
We have to mechanisms to define type allowing different implementations: Interfaces and abstract classes. After `default methods` on intrefaces, both mechanisms allow you let some implementation methods.

The main reason is we can extend one unique class, but we can implement more than one interfaces. So we can add new behavior to our class, implementing a new interface and let the current implementations.

### Item 21: Design interfaces for posterity
Default methods is the key of this topic. Avoid adding it on existing interfaces, but you can add it when you are creating new interfaces to give a default implementation.

### Item 22: Use interfaces only to define types

When a class implements an interface, this interface works as a type, that we can use to refer instances of the class. 

Don't use an interface to define constants, building only with final static fields. Instead of, use an enum or an util class.

### Item 23: Prefer class hierarchies to tagged classes

Tagged class is a class that have a field (for example an enum value) to describe the real type/function of this class. This kind of classes have some problems, like boilerplates, null fields and switches.

For this purpose, inheritance can be a better option.

### Item 24: Favor static member classes over nonstatic
A nested class should exist only to serve its enclosing class. 4 types of classes:
#### 1. static member classes
The simplest kind of nested class. It is an ordinary class that it is declared inside another class and has access to all of the enclosing class’s members.

#### 2. nonstatic member classes
Syntactically, the only difference between static and nonstatic member classes is that static member classes have the modifier static in their declarations. Each instance of a nonstatic member class is implicitly associated with an enclosing instance of its containing class: It is impossible to create an instance of a nonstatic member class without an enclosing instance

If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration.

#### 3. anonymous classes
An anonymous class has no name. It is declared and instancied at the same time at the point of use. There are many limitations on the applicability of anonymous classes, like can not execute **instanceof** test. We can not implement many interfaces at the same time too. And the clients can not invoke members, except those it inherits from its supertype. Must be short classes to keep readability.

Today, it is recomended use lambda instead of this kind of classes.

#### 4. local classes
Local classes are the least frequently used.

### Item 25: Limit source files to a single top-level class
While the Java compiler lets you define multiple top-level classes in a single source file, there are no benefits associated with doing so, and there are significant risks.

If you are temted to put multiple top-level classes into a single source file, consider using static member classes (Item 24)

The leason is clear: Never put multiple top-level classes or interfaces in a single source file. Following this rule guarantees that you can’t have multiple definitions for a single class at compile time.

---

## Generics
### Item 26: Don’t use raw types 
A class or interface whose declaration has one or more type parameters is a generic class or interface (for example `List<E>`). Each generic type defines a raw type, which is the name of the generic type used without any accompanying type parameters. In the example, `List` is the raw type.

It pays to discover errors as soon as possible after they are made, ideally at compile time. Using raw types, you don’t discover potential errors until runtime.

It is legal to use raw types, but you should never do it. If you use raw types, you lose all the safety and expressiveness benefits of generics.

_note_: You lose type safety if you use a raw type such as `List`, but not if you use a parameterized type such as `List<Object>`.

If you want to use a generic type but you don’t know or care what the actual type parameter is, you can use a question mark instead. For example, the unbounded wildcard type for the generic type Set<E> is Set<?> (read “set of some type”). I

#### Exceptions
You must use raw types in class literals. The specification does not permit the use of parameterized types. In other words, `List.class` and `String[].class` are all legal, but `List<String>.class` and `List<?>.class` are not.

Another exception is using `instanceof`. After validate as raw type, we must to cast to willcard type. 


### Item 27: Eliminate unchecked warnings
Eliminate every unchecked warning that you can. If you can’t eliminate a warning, but you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an `@SuppressWarnings("unchecked")` annotation.

Every time you use a @SuppressWarnings("unchecked") annotation, add a comment saying why it is safe to do so.

Every unchecked warning represents the potential for a `ClassCastException` at runtime. Do your best to eliminate these warnings. 

### Item 28: Prefer lists to arrays 
Arrays are covariant, that means simply that if `Sub` is a subtype of `Super`, then the array type `Sub[]` is a subtype of the array type `Super[]`. Generics, by contrast, are invariant;

```java
    // Fails at runtime!
    Object[] objectArray = new Long[1];
    objectArray[0] = "I don't fit in"; // Throws ArrayStoreException

    // Won't compile!
    List<Object> ol = new ArrayList<Long>(); // Incompatible types
    ol.add("I don't fit in");
```

Arrays know and enforce their element type at runtime. We can't create a generic array because it isn't `typesafe`.

### Item 29: Favor generic types

Writing your own generic types is a bit more difficult, but it’s worth the effort to learn how.

### Item 30: Favor generic methods 
### Item 31: Use bounded wildcards to increase API flexibility 
### Item 32: Combine generics and varargs judiciously
### Item 33: Consider typesafe heterogeneous containers 

151 6 Enums and Annotations 
### Item 34: Use enums instead of int constants
### Item 35: Use instance fields instead of ordinals 
### Item 36: Use EnumSet instead of bit fields
### Item 37: Use EnumMap instead of ordinal indexing
### Item 38: Emulate extensible enums with interfaces 
### Item 39: Prefer annotations to naming patterns 
### Item 40: Consistently use the Override annotation
### Item 41: Use marker interfaces to define types 

191 7 Lambdas and Streams 
### Item 42: Prefer lambdas to anonymous classes 
### Item 43: Prefer method references to lambdas 
### Item 44: Favor the use of standard functional interfaces 
### Item 45: Use streams judiciously 
### Item 46: Prefer side-effect-free functions in streams 
### Item 47: Prefer Collection to Stream as a return type
### Item 48: Use caution when making streams parallel 

8 Methods 
### Item 49: Check parameters for validity 
### Item 50: Make defensive copies when needed 
### Item 51: Design method signatures carefully 
### Item 52: Use overloading judiciously 
### Item 53: Use varargs judiciously 
### Item 54: Return empty collections or arrays, not nulls 
### Item 55: Return optionals judiciously 
### Item 56: Write doc comments for all exposed API elements 

9 General Programming 
### Item 57: Minimize the scope of local variables
### Item 58: Prefer for-each loops to traditional for loops
### Item 59: Know and use the libraries 
### Item 60: Avoid float and double if exact answers are required 
### Item 61: Prefer primitive types to boxed primitives 
### Item 62: Avoid strings where other types are more appropriate 
### Item 63: Beware the performance of string concatenation 
### Item 64: Refer to objects by their interfaces 
### Item 65: Prefer interfaces to reflection 
### Item 66: Use native methods judiciously
### Item 67: Optimize judiciously 
### Item 68: Adhere to generally accepted naming conventions

10 Exceptions 
### Item 69: Use exceptions only for exceptional conditions 
### Item 70: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors .
### Item 71: Avoid unnecessary use of checked exceptions 
### Item 72: Favor the use of standard exceptions
### Item 73: Throw exceptions appropriate to the abstraction
### Item 74: Document all exceptions thrown by each method
### Item 75: Include failure-capture information in detail messages
### Item 76: Strive for failure atomicity 
### Item 77: Don’t ignore exceptions 

11 Concurrency 
### Item 78: Synchronize access to shared mutable data 
### Item 79: Avoid excessive synchronization 
### Item 80: Prefer executors, tasks, and streams to threads 
### Item 81: Prefer concurrency utilities to wait and notify 
### Item 82: Document thread safety 
### Item 83: Use lazy initialization judiciously 
### Item 84: Don’t depend on the thread scheduler 

12 Serialization 
### Item 85: Prefer alternatives to Java serialization 
### Item 86: Implement Serializable with great caution 
### Item 87: Consider using a custom serialized form 
### Item 88: Write readObject methods defensively 
### Item 89: For instance control, prefer enum types to readResolve .
### Item 90: Consider serialization



