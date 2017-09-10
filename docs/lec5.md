# Lecture 5: Numbers, Strings, Collections

## Learning Outcomes

- Familiar with wrapper classes with primitives and autoboxing / unboxing; when to use primitive types and when to use wrapper classes
- Understand the differences between mutable and immutable objects, using String and StringBuffer as example
- Familiar with the comparator and iterator interfaces
- Understand more about generics: type erasure, generic methods, wildcard types, bounded wild card types.
- Familiar with Java collection frameworks: `Set`, `List`, `Map` and their concrete class `HashSet`, `LinkedList`, `ArrayList`, and `HashMap`.
- Aware of the other classes in Java Collection and is comfortable to look them up by reading the Java documentation.
- Understand the need to override `hashCode` every time `equals` is overriden.
- Understand there are differences between the collection classes and know when to use which one

## Wrapper Classes

Let's consider how to swap two integers:

```Java
class Swapper {
	public static void swap(int x, int y) {
		int temp = x;
		x = y;
		y = temp;
	}
}
```

The above code is a classic example of the `swap` method.  It does not work in Java.  Recall that Java uses pass-by-value for primitive types -- changing x and y only changes the local copies of the variables on the call stack of `swap`, not the variables belonging to the caller.

We should use pass-by-reference.  In Java, since all objects are passed by reference, we need to pass in an object that represents an integer.  Java provides a set of wrapper class:  one for each primitive type: `Boolean`, `Byte`, `Character`, `Integer`, `Double`, `Long`, `Float`, and `Short`.  So, we could write the `swap` method as:

```Java
class Swapper {
	public static void swap(Integer x, Integer y) {
		Integer temp = x;
		x = y;
		y = temp;
	}
}
```

The above would successfully swap the two integers.

An old fashion way (before Java 5) of calling the `swap` method to swap two `int` values is:

```Java
int x = 1;
int y = 2;
Integer wrapX = new Integer(x);
Integer wrapY = new Integer(y);
Swapper.swap(wrapX, wrapY);
x = wrapX.intValue();
y = wrapY.intValue();
```

Tedious to write.  Not very easy to read.

Java 5 introduces something called _autoboxing_ and _unboxing_, which creates the wrapper objects automatically (autoboxing) and retrieves its value (unboxing) automatically.  With autoboxing and unboxing, we can just write:
```Java
int x = 1;
int y = 2;
Swapper.swap(x, y);
```

Note that `swap` expects two `Integer` objects, but we pass in two `int`.  This would cause the `int` variable to automatically be boxed (i.e., be wrapped in Integer object) and put onto the call stack of `swap`.  After `swap` is done executing, before we clear the call stacks, the values are unboxed and copied to the calling method.

The wrapper class around primitive types is useful in another way.  Recall the generic class `Queue<E>` which we talked in [Lecture 4](lec4.md).  We can declare a `Queue` of `Point`, a `Queue` of `Circle`, etc, but we cannot create a `Queue` of `int` or a `Queue` of `boolean`.  We can only pass in a class name to the type parameter `E`, not a primitive type.  So, to create a queue of integers, we cannot use `Queue<int>` -- we have to use `Queue<Integer>`.

```
Queue<Integer> iq = new Queue<Integer>(10);
cq.enqueue(4);
cq.enqueue(8);
cq.enqueue(15);
```

!!! note "Type Erasure"
    The reason why Java compiler does not allow generic class with primitive types, is that internally, the compiler uses _type erasure_ to implement generic class.  Type erasure just means that during compile time, the compiler replaces the type parameter with the most general type.  In the example given in [Lecture 4](lec4.md), `E` in `Queue<E>` is replaced with `Object`,  The compiler then inserts necessary cast to convert the `Object` to the type argument (e.g., `Circle`), exactly like how it is done in the `ObjectQueue` example, and additional checks to ensure that only objects of given type is used as `E` (e.g., cannot add `Point` to `Queue<Circle>`).  Since primitive types are not subclass `Object`, replacing `E` with primitive types would not work with type erasure.  

	Note that, as a consequence of type erasure, at runtime, Java has no information about `E`.

In short, the wrapper class allows us to pass primitive types by references and use primitive types to parameterize a generic class, and we do not have to write code to box and unbox the primitive types.  

### Performance Penalty

If the wrapper class is so great, why not use it all the time and forget about primitive types?  Like:

```Java
Integer x = 1;
Integer y = 2;
Swapper.swap(x, y);
```

The answer: performance.  Because using an object comes with the cost of allocating memory for the object and collecting of garbage afterwards, it is less efficient than primitive types.  Consider the following two programs:

```Java
Double sum;
for (int i = 0; i < Integer.MAX_VALUE; i++)
{
	sum += i;
}
```
vs.
```Java
double sum;
for (int i = 0; i < Integer.MAX_VALUE; i++)
{
	sum += i;
}
```

The second one is XXX times faster!  Due to autoboxing and unboxing, the cost of creating objects become hidden and often forgotten.

## String and StringBuilder

Another place with hidden cost for object creation and allocation is when dealing with `String`. 

A `String` object is _immutable_.  What this means is that once you create a `String` object, it cannot be changed.  When we do:
```Java
String words = "";
words += "Hello ";
words += "World!";
```
A new `String` object is created everytime we concatenate it with another `String`.

Java provides a mutable version of `String`, called `[StringBuilder](https://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html)`.  To build up a string, we could do:
```Java
StringBuilder wordsBuilder = new StringBuilder();
wordsBuilder.append("Hello ");
wordsBuilder.append("World!");
String words = wordsBuilder.toString();
```

Code that uses `StringBuilder` is not as intuitive and readable than just using the `+` operator.  My preference is to use `String` and `+` for occasional concatenation, and `StringBuilder` for frequent concatenation that could become performance bottleneck.

## Equality for Strings and Numbers

The following is a common bug, so worthy of a special mention here, with its own header!

One common mistake when comparing strings and numbers is to do the following:

```Java
String s1 = "Hello";
String s2 = "Hello";
if (s1 == s2) { ... }
```

or 
```Java
Integer i1 = 2342;
Integer i2 = 2342;
if (i1 == i2) { ... }
```

Remember that `==` compares only references: whether the two references are pointing the the same object or not.   The `equals` method has been overridden to compare if the values are the same or not.  So, the right way to compare two strings or two numbers are:

```Java
if (s1.equals(s2)) { ... }
if (i1.equals(i2)) { ... }
```

!!! note "Integer Cache"
    If you try:
		```Java
		Integer i1 = 1;
		Integer i2 = 1;
		if (i1 == i2) { ... }
		```
		and it might return `true`!  This behaviour is caused by some autoboxing optimization in the Integer class so that it does not create too many objects for frequently requested values.  It is called `Integer caching`.  If another `Integer` object with the same value has been autoboxed before, JVM just returns that object instead of returning a new one.
		Do not rely on Integer caching for proper comparisons of `==`.  Use `equals()`, always.

## Java Collections

Now, we turn our attention to the Java Collection Framework.  Java provides a rich set of classes for managing and manipulating data.  They efficiently implement many useful data structures (hash tables, red black trees, etc.) and algorithms (sorting, searching, etc.) so that we no longer have to.  As computer scientists, it is still very important for us to know how these data structures and algorithms can be implemented, how to prove some behaviors (such as running time) and their correctness, how certain trade offs are made, etc. They are so important that we have two modules dedicated to them: CS2040 and CS3230 in the core CS curriculum.  

For CS2030, however, we focus on how to use them.  

### Collection

One of the basic interface in Java Collection Framework is `Collection<E>`, it looks like:

```Java
public interface Collection<E> extends Iterable<E> {
  boolean add(E e);
  boolean contains(Object o);
  boolean remove(Object o);
  void clear();
  boolean isEmpty();
  int size();

  boolean equals(Object o);
  int hashCode();
  
  Object[] toArray();
  <T> T[] toArray(T[] a);

  boolean addAll(Collection<? extends E> c);
  boolean containsAll(Collection<?> c);
  boolean removeAll(Collection<?> c);
  boolean retainAll(Collection<?> c);
    :
}
```

There are some newly added methods in Java 8 that we will visit in the second half of this module, but first, 
let's try to understand what the definition above means.  First, like a generic class that you have seen, `Collection` is a _generic interface_ parameterized with a type parameter `E`.  It extends a generic `Iterable<E>` interface (we will get to this later).

The first six methods of `Collection<E>` should be self-explanatory.  `add` adds an element into the collection; `contains` checks if a given object is in the collection;  `remove` removes a single instance of the given object from the collection;  `clear` removes all objects from the collection;  `isEmpty()` checks if the collection has no elements or not; and finally, `size` returns the number of elements.

One point of note is that `contains()` relies of the implementation of `equals()` to check if the object exists in the collection or not.  Similarly, `remove()` relies on `equals()` to find the matching objects.  We said earlier that it is useful to override the `equals` methods of `Object` instead of implement our own `equals` because the overriden `equals()` will be called elsewhere.  This is one of the "elsewhere" I mentioned.  The documentation of `contains(o)` mentions that it is gurantee to return `true` if there exists an element `e` such that `e.equals(o)` or `e == null` (if `o == null`).  Ditto for `remove(o)`.


!!! note "Non-generic Methods"
    You might notice that, instead of `contains(E e)`and `remove(E e)`, the `Collection` interface uses `contains(Object o)` and `remove(Object o)`.  This little inconsistency, however, is harmless.  For instance, if you have a collection intended for circles only, adding a non-circle could be disastrous.  Trying to remove an non-circle or checking for a non-circle, would just return false.  
	More information can be found on this [StackOverflow](https://stackoverflow.com/questions/104799/why-arent-java-collections-remove-methods-generic) thread.

Java Collection Framework allows classes that implements an interface to throw an `UnsupportedOperationException` if the implementation decides not to implement one of the operations (but still need to have the method in the class).

The methods on Lines 9-10 should also be familiar.  A collection can check if it is equal to another collection (which inevitably also a subclass of `Object`).  As before, we will explain why we need `hashCode()` later.  Just bear with it a little longer.

The method `toArray()` on Line 12 returns an array containing all the elements inside this collection.  The second overloaded `toArray` method takes in an array of generic type `T`.  If the collections fit in `a`, `a` is filled and returned.  Else, it allocates a new array of type `T` and returned.  

The second `toArray` method is a _generic method_.  It is declared with `<T>` to indicate that the method can take any type `T`.  When we call generic method, we do not have to pass in a type argument.  Instead, the Java compiler infers the type from the arguments.  If we call `toArray(new String[10)`, it would return a `String[]`, if we call `toArray(new Point[0])`, it would return a `Point[]` and so on.
It is the caller resonsibility to pass in the right type, otherwise, an `ArrayStoreException` will be thrown.

The next group of methods operate on another collection.  `addAll` add all the elements of collection `c` into the current collection; `containsAll` checks if all the elements of collection `c` are contained in the current collection; `removeAll` removes all elements from collection `c`, and finally, `retainsAll` remove all elements not in `c`.

What is more interesting about the methods is the type of `c`.  In `containsAll`, for instance, the collection `c` has the type `Collection<?>`.  `?` is known as wildcard type, or _unknown_ type.  This notation is used to denote the supertype of all parameterized interfaces created from `Collection<E>`.

In `addAll`, `c` is declared as `Collection<? extends E>`.  The type parameter `<? extends E>` is an example of bounded type in generics.  It means that the type argument is still unknown, but we know that it extends `E`.  So, suppose I have a parameterized interface `Collection<Circle>` and `Circle extends PaintedCircle`, I can pass in a collection that has type `Collection<PaintedCircle>`.

Finally, let's get back to supertype of `Collection<E>`, `Iterable<E>`.  The `Iterable<E>` interface provides only a single interface, `Iterator<E> iterator()`, which returns a generic interface called `Iterator<E>` over the collection.  An `Iterator` is another interface that allows us to go through all the elements in a `Collection<E>`.  It has four method interfaces, three of which we will talk about today: `hasNext()`, which returns if there is a next element in the `Collection<E>`; `next()`, which returns the next element (with paramterized type `E`; and `remove()`, which removes the last returned element from the `Collection<E>`.

OK, so far I have talked about lots of methods but haven't showed any code.  This is because Java Collection Framework does not provide a class that implements the `Collection<E>` directly.  The documentation recommends that we implement the `Collection<E>` interface[^1] if we want a collection of objects that allows duplicates and does not care about the orders.

Let's move to somethat Java does have a concrete class implementation.  

## Set and List

The `Set<E>` `List<E>` interfaces extend the `Collection<E>` class.  `Set<E>` is meant for implementing a collection of objects that does not allow duplicates (but still does not care about order of elements), while `List<E>` is for implementing a collection of objects that allow duplicates, but the order of elements matters.  

Mathematically, a `Collection<E>` is used to implement a bag, `Set<E>`, a set, and `List<E>`, a sequence.

[^1]: If you want to do so, however, it is likely more useful to inherit from the abstract class `AbstractCollection<E>` (which implements most of the basic methods of the interface) rather than implementing the interface `Collection<E>` directly.

THe `List<E>` interface has additional methods for adding and removing elements.  `add(e)` by default would just add to the end of the list.  `add(i, e)` inserts `e` to position `i`.  `get(i)` returns the element at position `i`, `remove(i)` removes the elements at position `i`; `set(i,e)` replace the `i`-th element with `e`.  

Useful classes in Java collection that implements `List<E>` includes `ArrayList` and `LinkedList`, and useful classhes that implements `Set<E>` includes `HashSet`. 

Let's see some examples:

```Java
List<String> names = new ArrayList();
aryaList.add("Cersei");
aryaList.add("Joffrey");
aryaList.add(0, "Gregor");
System.out.println(aryaList.get(1));
```

Line 1 above creates a empty array list.  The second line adds two strings into the list, each appending them to the list.  After executing Line 3, it would contain the sequence `<"Cersei","Joffrey">`.  Line 4 inserts the string `"Gregor"` to position 0, moving the rest of the list down by 1 position.  The sequence is now `<"Gregor","Cersei","Joffrey">`.  Finally, calling `get(1)` would return the string `"Cersei"`.

Note that we declare `names` with the interface type `List<String>`.  We should always do this to keep our code flexible.  If we want to change our implementation to `LinkedList`, we only need to change Line 1 to:
```Java
List<String> names = new LinkedList();
```

### Comparator

The `List<E>` interface also specifies a `sort` method, with the following specification: 
```Java
default void sort(Comparator<? super E> c)
```

Remember at the end of Lecture 3 when we said there are "unpure" interfaces, that is interface that comes with implementation?  This is one of them.  The keyword `default` indicates that the interface `List<E>` comes with a default implementation of `sort` method.  So a class that implements the interface needs not implement it again if they do not want to.

This method specification is also interesting and worth looking closer.  It takes in an object `c` with generic interface `Comparator<? super E>`.  Like `<? extends E>` that we have seen before, this is a _bounded_ wildcard type.  While `<? extends E>` is an unknown type upper bounded by `E`, `<? super E>` is an unknown type lower bounded by `E`.  This means that we can pass in `E` or any supertype of E.

What does the `Comparator` interface do?  We can specify how to compare two elements of a given type, by implementing a `compare()` method.
`compare(o1,o2)` should return 0 if the two elements are equals, a negative integer if o1 is "less than" o2, and a positive integer otherwise.

Let's write `Comparator` class[^2]:

```Java
class NameComparator implements Comparator<String> {
	public int compare(String s1, String s2) {
		return s1.compareTo(s2);
	}
}
```

In the above, we use the `compareTo` method provided by the `String` class to do the comparison.  With the above, we can now sort the `names`:

```Java
names.sort(new NameComparator());
```

This would result in the sequence being changed to `<"Cersei","Gregor","Joffrey">`.

[^2]: Later in CS2030, you will see how we significantly reduce the verbosity of this code!  But let's do it the hard way first.

## Map

One of the more powerful data structures provided by Java Collection is maps (also known as dictionary in other languages).  A map allows us to store a (unique key, value) pair into the collection, and retrieve the value later by looking up the key.

The `Map<K,V>` interface is again generic, but this time, has two type parameters, `K` for the type of the key, and `V` for the type of the value.  These makes the `Map` interface flexible -- we can use any type as the key and value.

The two most important methods for `Map` is `put` and `get`:

```Java
	V put(K key, V value);
	K get(Object k);
```

A useful class that implements `Map` interface is `HashMap`:

```Java
Map<String,Integer> population = new HashMap<String,Integer>();
population.put("Oldtown",500000);
population.put("Kings Landing",500000);
population.put("Lannisport",300000);
```

Later, if we want to lookup the value, we can:
```Java
population.get("Kings Landing");
```

Internally, to implement `put`, `HashMap` calls key's `hashCode` to return a `int`, which it uses to determine which "bucket" to store the (key, value) pair.  When `get`, `HashMap` again calls `hashCode` on the given key to determine which bucket, then it looks for the key in the bucket.  This process, called _hashing_, circumvents the need to look through every pair in the map to find the right key.

You will learn more about hashing and hash tables in CS2040.

But, what is important here is that, two keys (two objects, in general) which are the same (`equals()` returns `true`), must have the same `hashCode()`.  Otherwise, `HashMap` would fail!  

So it is important to ensure that if `o1.equals(o2)`, then `o1.hashCode() == o2.hashCode()`.  Note that the reverse does not have to be true -- two objects with the same hash code does not have to be equals.

This property is also useful for implementing `equals()`.  For a complex object, comparing every field for equality can be expensive.  If we can compare the hash code first, we could filter out objects with different hash code (since they cannot be equal).  We only need to compare field by field if the hash code is the same.

Ditto for implementation of `HashSet` -- to checks if an element to add already exists, `HashSet` uses hash code, instead of going through all the elements and compare one by one.

Calculating good hash code is an involved topic, and is best left to the expert (some of you might become expert in this), but for now, we can rely on the static `hashCode` methods in the `Array` class to help us.

## Which Collection Class?

Java provides many collection classes, more than what we have time to go through.  It is important to know which one to use to get the best performance out of them.  For the few classes we have seen:

- Use `HashMap` if you want to keep a (key, value) pair for lookup later.
- Use `HashSet` if you have a collection of elements with no duplicates and order is not important.
- Use `ArrayList` if you have a collection of elements with possibly duplicates and order is important, and retriving a specific location is more important than removing elements from the list.
- Use `LinkedList` if you have a collection of elements with possibly duplicates and order is important, retriving a specific location is less important than removing elements from the list.

Further, if you want to check if a given object is contained in the list, then `ArrayList` and `LinkedList` are not good candidates.

You should understand why after CS2040.