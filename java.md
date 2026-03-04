# Java Interview & DSA Notes

Comprehensive notes covering Java fundamentals, OOP, collections, and commonly asked interview topics.

## Table of Contents

1. [Java Basics](#1-java-basics-must-know) — Data types, variables, operators, control flow  
2. [OOP Concepts](#2-oop-concepts-very-important) — Encapsulation, inheritance, polymorphism, abstraction  
3. [Classes and Objects](#3-classes-and-objects) — Constructors, this, super, chaining  
4. [Access Modifiers](#4-access-modifiers)  
5. [Static Keyword](#5-static-keyword)  
6. [Memory Management](#6-memory-management-important) — Heap, stack, method area  
7. [Garbage Collection](#7-garbage-collection)  
8. [String Handling](#8-string-handling) — String, StringBuilder, StringBuffer  
9. [Collections Framework](#9-collections-framework-very-important)  
10. [HashMap Internals](#10-hashmap-internal-working-very-important)  
11. [Exception Handling](#11-exception-handling)  
12. [Multithreading Basics](#12-multithreading-basics)  
13. [Interfaces](#13-interfaces) — Default, static, functional  
14. [Lambda Expressions](#14-lambda-expressions)  
15. [Java 8 Features](#15-java-8-features)  
16. [Streams API](#16-streams-api-important)  
17. [Equals vs ==](#17-equals-vs-)  
18. [Comparable vs Comparator](#18-comparable-vs-comparator)  
19. [Immutable Classes](#19-immutable-classes)  
20. [Important Internal Q&A](#20-important-internal-questions--quick-answers)  
- [What to Skip](#what-you-can-skip-for-dsa-focused-roles)  
- [What Interviewers Test](#what-interviewers-actually-test-dsa--java)  
- [Practice Exercises](#practice-exercises)  
- [Interview Q&A](#quick-reference-common-interview-qa)  
- [Additional Topics](#additional-topics)  
- [More Exercises](#more-exercises)  

---

# 1. Java Basics (Must Know)

These questions appear very frequently.

## Data Types

### Primitive vs Non-primitive

| Aspect | Primitive | Non-primitive (Reference) |
|--------|-----------|---------------------------|
| Stored in | Stack (or heap if part of object) | Reference on stack, object on heap |
| Default value | Has default (0, false, etc.) | `null` |
| Size | Fixed | Variable |
| Examples | `int`, `double`, `boolean` | Classes, arrays, interfaces |

**Primitive types:** `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`

**Non-primitive:** All classes (including `String`), arrays, interfaces.

### Size of primitives

| Type | Size | Range |
|------|------|--------|
| `byte` | 8 bits | -128 to 127 |
| `short` | 16 bits | -2¹⁵ to 2¹⁵ - 1 |
| `int` | 32 bits | -2³¹ to 2³¹ - 1 |
| `long` | 64 bits | -2⁶³ to 2⁶³ - 1 |
| `float` | 32 bits | IEEE 754 |
| `double` | 64 bits | IEEE 754 |
| `char` | 16 bits (Unicode) | 0 to 65535 |
| `boolean` | JVM-dependent | `true` / `false` |

### Wrapper classes

Each primitive has a wrapper in `java.lang`:

- `Byte`, `Short`, `Integer`, `Long`, `Float`, `Double`, `Character`, `Boolean`

Use cases: generics (e.g. `List<Integer>`), nullability, utility methods.

```java
Integer i = Integer.valueOf(10);  // boxing
int n = i.intValue();             // unboxing
int parsed = Integer.parseInt("42");
```

**Autoboxing / Unboxing:** Automatic conversion between primitive and wrapper.

```java
Integer a = 5;   // autoboxing
int b = a;       // unboxing (NPE if a is null)
```

---

## Variables

### Instance variables

- Declared inside a class, outside any method.
- One copy per object; live on the heap with the object.
- Get default values if not initialized.
- Accessible via `this` or directly in instance methods.

```java
class Person {
    String name;   // instance variable
    int age;
}
```

### Local variables

- Declared inside a method, constructor, or block.
- Stored on the stack; exist only during execution of that scope.
- **No default value** — must be initialized before use.
- Not accessible from other methods.

```java
void method() {
    int x = 10;           // local
    final int y = 20;     // effectively final for lambdas
}
```

### Static variables (class variables)

- Declared with `static`; one copy per class, shared by all instances.
- Stored in method area; loaded when class is loaded.
- Access via class name: `ClassName.varName`.

```java
class Counter {
    static int count = 0;  // shared across all instances
}
```

---

## Operators

### Arithmetic

`+`, `-`, `*`, `/`, `%`, `++`, `--`

- Integer division truncates: `7 / 2` → `3`.
- `+` overloaded for String concatenation.

### Logical

`&&`, `||`, `!` — short-circuit evaluation (right side may not be evaluated).

```java
if (a != null && a.length() > 0)  // safe; no NPE if a is null
```

### Bitwise

`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`

- `&`, `|`, `^` can be used on booleans (no short-circuit).
- `>>` signed right shift; `>>>` unsigned.
- Useful for flags, masks, and low-level logic.

```java
int a = 5;   // 0101
int b = 3;   // 0011
a & b;       // 1
a | b;       // 7
a ^ b;       // 6
```

---

## Control Statements

### if / else

```java
if (condition) { }
else if (condition) { }
else { }
```

Ternary: `result = condition ? valueIfTrue : valueIfFalse;`

### switch

Works with `byte`, `short`, `int`, `char`, `String`, enums, and (Java 7+) wrapper types. No fall-through if you use `break`.

```java
switch (x) {
    case 1: break;
    case 2: break;
    default: break;
}
```

Java 14+ switch expression:

```java
String result = switch (day) {
    case "Mon" -> "Work";
    case "Sat", "Sun" -> "Rest";
    default -> "Other";
};
```

### Loops

- **for:** `for (int i = 0; i < n; i++) { }`
- **enhanced for:** `for (Item x : collection) { }`
- **while:** `while (condition) { }`
- **do-while:** `do { } while (condition);`

`break` exits loop; `continue` skips to next iteration.

### Common pitfalls

- **Switch fall-through:** Missing `break` runs into the next case (use deliberately or avoid).
- **Floating-point comparison:** Don’t use `==`; use tolerance or `BigDecimal`.
- **Integer division:** `7 / 2` is `3`; use `7.0 / 2` or cast for floating result.

---

### Exercise 1.1

What is the output?

```java
Integer a = 128;
Integer b = 128;
System.out.println(a == b);
System.out.println(a.equals(b));
```

<details>
<summary>Answer</summary>

`false` then `true`. `Integer` caches -128 to 127; outside that range, `==` compares references. `equals()` compares values.
</details>

---

# 2. OOP Concepts (Very Important)

## Encapsulation

Bundling data and methods that operate on that data; restricting direct access via **data hiding**.

- Use `private` fields.
- Expose controlled access via **getters** and **setters** (or builders, constructors).

```java
class BankAccount {
    private double balance;

    public double getBalance() { return balance; }
    public void setBalance(double balance) {
        if (balance >= 0) this.balance = balance;
    }
}
```

Benefits: validation, easier evolution, controlled state.

---

## Inheritance

A subclass gets fields and methods of a superclass. **Code reuse** and **polymorphism**.

### Types

- **Single:** B extends A.
- **Multilevel:** A → B → C.
- **Hierarchical:** B extends A, C extends A.

Java **does not support multiple inheritance with classes** (to avoid diamond problem and ambiguity). Multiple inheritance is supported via **interfaces** (multiple interface implementation).

```java
class Animal { }
class Dog extends Animal { }           // single
class Puppy extends Dog { }           // multilevel
class Cat extends Animal { }          // hierarchical (Animal has Dog and Cat)
```

---

## Polymorphism

“Many forms” — same interface, different behavior.

### Compile-time (static) polymorphism

- **Method overloading:** same method name, different parameter list (number or types). Resolved at compile time.

```java
void print(int x) { }
void print(String s) { }
void print(int x, String s) { }
```

Return type alone is not enough to overload.

### Runtime (dynamic) polymorphism

- **Method overriding:** subclass provides its own implementation of a superclass method. Resolved at runtime (JVM picks actual object type).

```java
class Animal {
    void speak() { System.out.println("..."); }
}
class Dog extends Animal {
    @Override
    void speak() { System.out.println("Woof"); }
}
Animal a = new Dog();
a.speak();  // "Woof"
```

Rules: same signature, return type compatible (covariant allowed), access not more restrictive.

---

## Abstraction

Hiding implementation details; exposing only what is necessary.

### Abstract class

- Declared with `abstract`.
- Can have abstract methods (no body) and concrete methods.
- Cannot be instantiated; used as a base class.
- Can have constructors, instance fields, any access modifier.

```java
abstract class Shape {
    abstract double area();
    void describe() { System.out.println("Shape"); }
}
class Circle extends Shape {
    double r;
    double area() { return Math.PI * r * r; }
}
```

### Interface

- Contract of methods (and optionally default/static methods).
- Before Java 8: only public abstract methods and public static final fields.
- Java 8+: `default` and `static` methods.
- A class can implement multiple interfaces.

```java
interface Drawable {
    void draw();
    default void erase() { System.out.println("Erasing"); }
}
```

### Abstract class vs interface

| Aspect | Abstract class | Interface |
|--------|----------------|-----------|
| Inheritance | Single only | Multiple allowed |
| Methods | Abstract + concrete | Abstract; default/static in Java 8+ |
| Fields | Any | `public static final` (constants) only |
| Constructors | Yes | No |
| When to use | “is-a”, shared base state | “can-do”, contract only |

Use abstract class when subclasses share substantial state/behavior; use interface for capabilities and multiple contracts.

---

### Exercise 2.1

Can you overload `main`? Can you override it?

<details>
<summary>Answer</summary>

Overload: yes — you can add `main(String[] args, int x)` etc.; only `public static void main(String[] args)` is the JVM entry point. Override: not in the usual sense — `main` is static; static methods are hidden, not overridden, in subclasses.
</details>

---

# 3. Classes and Objects

## Constructors

- Special method used to initialize objects; same name as class, no return type.
- If you don’t write one, compiler inserts a **default no-arg constructor** (only if no other constructor is defined).
- Can be overloaded.

```java
class Person {
    String name;
    int age;

    Person() { name = "Unknown"; }
    Person(String name) { this.name = name; }
    Person(String name, int age) {
        this(name);  // constructor chaining
        this.age = age;
    }
}
```

## Constructor overloading

Multiple constructors with different parameter lists. Reuse logic via `this(...)`.

## this keyword

- Refers to current instance.
- `this.field`, `this.method()` — disambiguate or pass current object.
- `this()` — call another constructor in the same class (must be first statement).

## super keyword

- Refers to superclass.
- `super.field`, `super.method()` — access overridden/hidden members.
- `super()` — call superclass constructor (must be first statement if present).

```java
class Child extends Parent {
    Child() {
        super();  // or super(args)
    }
}
```

## Constructor chaining

- In a constructor, `this()` or `super()` is called first (implicitly or explicitly).
- Default `super()` is added if you don’t call `super(...)` or `this(...)`.
- Order: from child constructor up to `Object`.

---

### Exercise 3.1

What happens here?

```java
class A {
    A() { System.out.println("A"); }
}
class B extends A {
    B() {
        System.out.println("B");
    }
}
new B();
```

<details>
<summary>Answer</summary>

Prints "A" then "B". Superclass constructor runs before subclass body.
</details>

---

# 4. Access Modifiers

Control visibility of classes, methods, and fields.

| Modifier   | Same class | Same package | Subclass (any package) | World |
|------------|------------|--------------|------------------------|-------|
| `private`  | ✓          | ✗            | ✗                      | ✗     |
| default    | ✓          | ✓            | ✗ (only if same pkg)   | ✗     |
| `protected`| ✓          | ✓            | ✓                      | ✗     |
| `public`   | ✓          | ✓            | ✓                      | ✓     |

- **private:** Only inside the same class.
- **default (package-private):** No modifier; same package only.
- **protected:** Same package + subclasses (even in other packages).
- **public:** Everywhere.

Top-level classes can only be `public` or default (package-private).

---

# 5. Static Keyword

- **Static variable:** One per class; shared by all instances; often used for constants or counters.
- **Static method:** Belongs to class; called as `ClassName.method()`; cannot use `this`; **cannot directly access instance fields/methods** (no “current object”).
- **Static block:** Runs once when class is loaded (e.g. initialize static fields).

```java
class Util {
    static int count;
    static {
        count = 0;
    }
    static void increment() { count++; }
}
```

- **Static nested class:** Nested class that doesn’t need an outer instance. Not the same as inner class (which has an implicit reference to outer object).

**Why can’t static methods access non-static members?** Static methods run without any object. Instance fields/methods require an object. So from a static context there is no `this` and no instance to read or call.

---

# 6. Memory Management (Important)

## Heap

- Stores **objects** (including arrays and instance fields).
- Shared by all threads.
- Garbage collector reclaims unreachable objects.

## Stack

- Per-thread.
- Stores **local variables**, method parameters, partial results.
- Method calls push frames; return pops them.
- References (variables) live on stack; the objects they point to are on heap.

## Method area / Metaspace

- Per JVM.
- **Class metadata**, static fields, constant pool, method data.
- In older JVM versions: “PermGen”. In Java 8+: **Metaspace** (native memory).

Summary:

- **Where are objects stored?** Heap.
- **Where are local variables stored?** Stack (references on stack, objects on heap).

---

# 7. Garbage Collection

- **Garbage collection (GC):** Reclaiming memory of objects that are no longer reachable.
- **Eligibility:** No references from live threads (no path from GC roots).

```java
Object o = new Object();
o = null;  // previous object now eligible (if no other refs)
```

- **`System.gc()`:** Hint to run GC; JVM may ignore it. Don’t rely on it for correctness.
- Generational GC (young vs old generation) and different algorithms (e.g. G1, ZGC) are advanced topics.

---

# 8. String Handling

## String

- **Immutable.** Any “change” creates a new object.
- Stored in heap; literals may be interned in string pool.

```java
String s = "hello";
s = s + " world";  // new String object; "hello" unchanged
```

## StringBuilder

- **Mutable.** In-place modification; no thread safety.
- Use when doing many concatenations in one thread.

```java
StringBuilder sb = new StringBuilder("hello");
sb.append(" world");
```

## StringBuffer

- **Mutable** and **thread-safe** (synchronized methods).
- Use when multiple threads modify the same buffer.

**Why is String immutable?** Security (e.g. no accidental change of sensitive data), safety for sharing (e.g. as key in HashMap), string pool optimization, thread safety without locking.

**String vs StringBuilder:** Immutable vs mutable; use StringBuilder for heavy concatenation in a single thread.

---

### Exercise 8.1

What is the output?

```java
String s1 = "java";
String s2 = "java";
String s3 = new String("java");
System.out.println(s1 == s2);
System.out.println(s1 == s3);
System.out.println(s1.equals(s3));
```

<details>
<summary>Answer</summary>

`true`, `false`, `true`. Literals may be interned (same reference for `s1` and `s2`). `new String(...)` creates a new object, so `s1 == s3` is false. `equals` compares content, so true.
</details>

---

# 9. Collections Framework (Very Important)

## Hierarchy

```
Iterable
  └── Collection
        ├── List
        ├── Set
        └── Queue
Map (separate; key-value)
```

## List (ordered, allows duplicates)

- **ArrayList:** Resizable array; fast get/set by index; slower insert/delete in middle.
- **LinkedList:** Doubly linked list; fast insert/delete at ends; slower random access.

## Set (no duplicates)

- **HashSet:** Hash table; O(1) average add/contains; no order.
- **LinkedHashSet:** Insertion order.
- **TreeSet:** Sorted (natural or Comparator); O(log n).

## Queue

- **LinkedList** (as Queue), **PriorityQueue**, **ArrayDeque**.

## Map (not part of Collection)

- **HashMap:** O(1) average; no order; allows one null key.
- **LinkedHashMap:** Insertion (or access) order.
- **TreeMap:** Sorted keys; O(log n).

**ArrayList vs LinkedList:** ArrayList for random access and general use; LinkedList for frequent insert/remove at head/tail or when used as queue/deque.

**HashMap vs TreeMap:** HashMap for speed and no order; TreeMap for sorted keys.

**HashSet vs HashMap:** HashSet is a set of elements; HashMap is key-value. HashSet can be implemented using a HashMap with dummy value.

---

# 10. HashMap Internal Working (Very Important)

- **Hashing:** `hashCode()` returns an int; HashMap converts it to bucket index (e.g. `hash & (n-1)` when capacity is power of 2).
- **Buckets:** Array of buckets; each bucket holds entries (linked list or, in Java 8+, tree after threshold).
- **Collisions:** Same index for different keys; chaining (list or tree) resolves.
- **Put:** Compute hash → index → add/replace in bucket; if size > threshold, **resize** (e.g. double capacity, rehash).
- **Load factor:** Default 0.75; when `size/capacity` exceeds it, resize to reduce collisions.
- **Java 8+:** When a bucket has many keys (e.g. 8), list turns into **red-black tree** for O(log n) in that bucket.

Key methods: `hashCode()` and `equals()` must be consistent; override both for custom keys.

---

# 11. Exception Handling

- **try:** Guarded block.
- **catch:** Handles exception (by type); multiple catch blocks; more specific first.
- **finally:** Runs (usually) after try/catch; used for cleanup.
- **throw:** Throws an exception instance.
- **throws:** Declares checked exceptions the method may throw.

**Checked exceptions:** Must be declared or caught (e.g. `IOException`).  
**Unchecked (runtime):** Subclasses of `RuntimeException` (e.g. `NullPointerException`, `IllegalArgumentException`); need not be declared.

```java
try {
    // code
} catch (IOException e) {
    // handle
} finally {
    // cleanup
}
```

---

# 12. Multithreading Basics

### Creating threads

1. **Extend Thread:** override `run()`; start with `start()` (not `run()`).
2. **Implement Runnable:** pass to `new Thread(runnable)`; better for “task” vs “thread” separation.

```java
Thread t = new Thread(() -> System.out.println("Running"));
t.start();
```

### Thread lifecycle

New → Runnable → (Running) → Blocked/Waiting/Timed Waiting → Terminated.

### Synchronization

- **race condition:** Multiple threads updating shared state without coordination.
- **synchronized:** On method or block; one thread at a time holds the lock.
- **volatile:** Visibility across threads (no compound atomicity).

---

# 13. Interfaces

- **Default methods (Java 8):** `default void method() { }` — backward compatibility; can be overridden.
- **Static methods (Java 8):** `static void method() { }` — belong to interface; called as `InterfaceName.method()`.
- **Functional interface:** Single abstract method; can be used with lambdas (e.g. `Runnable`, `Comparator`).

```java
@FunctionalInterface
interface Adder {
    int add(int a, int b);
}
Adder adder = (a, b) -> a + b;
```

---

# 14. Lambda Expressions

Concise syntax for functional interface implementation.

```java
(a, b) -> a + b
() -> System.out.println("Hi")
x -> x * 2
(int a, int b) -> a + b
```

Variables used in lambdas must be effectively final.

---

# 15. Java 8 Features

- **Lambdas** and **functional interfaces**
- **Streams API**
- **Optional**
- **Method references** (e.g. `String::valueOf`, `list::add`)
- **Default and static methods in interfaces**
- **New date/time API:** `java.time`

---

# 16. Streams API (Important)

- **filter(Predicate):** Keep elements that match.
- **map(Function):** Transform each element.
- **reduce:** Combine to one value.
- **collect:** To list, set, map, etc.

```java
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .collect(Collectors.toList());
```

Other: `sorted`, `distinct`, `limit`, `forEach`, `anyMatch`, `allMatch`, `findFirst`, etc.

---

# 17. Equals vs ==

- **==:** For primitives, compares values; for references, compares **references** (same object or not).
- **equals():** For objects, compares **content** (if overridden correctly). Default in `Object` is same as `==`.

Override `equals()` (and `hashCode()`) for your classes when you need value equality (e.g. for use in `HashMap`, `HashSet`).

```java
String a = new String("hi");
String b = new String("hi");
a == b;       // false
a.equals(b); // true
```

---

# 18. Comparable vs Comparator

- **Comparable:** `compareTo(T other)`; natural ordering; implement in the class.
- **Comparator:** Separate class or lambda; `compare(T a, T b)`; multiple orderings.

```java
// Comparable
class Person implements Comparable<Person> {
    public int compareTo(Person o) { return this.name.compareTo(o.name); }
}
Collections.sort(people);

// Comparator
Comparator<Person> byAge = (a, b) -> Integer.compare(a.getAge(), b.getAge());
people.sort(byAge);
```

---

# 19. Immutable Classes

- All fields private (and ideally final).
- No setters; don’t expose mutable internals (or return defensive copies).
- Class final or constructors private (to avoid subclass mutation).
- Initialize via constructor (or factory).

```java
public final class ImmutablePoint {
    private final int x;
    private final int y;
    public ImmutablePoint(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
}
```

String is the classic example.

---

# 20. Important Internal Questions — Quick Answers

1. **Why is String immutable?** Security, safe sharing, string pool, thread safety.
2. **How does HashMap work?** Hash → bucket index → chaining (list/tree); load factor and resize.
3. **Why no multiple inheritance of classes?** Diamond problem and ambiguity; interfaces provide multiple “contracts” without state.
4. **Why is main static?** JVM calls it without creating an object; single entry point.
5. **Abstract class vs interface?** Single vs multiple inheritance; state + constructors vs contract; use abstract for “is-a” with state, interface for “can-do” and multiple roles.

---

# What You Can Skip (for DSA-focused roles)

Unless the role is Java backend:

- Spring Boot
- Hibernate / JPA
- JDBC
- Microservices

---

# What Interviewers Actually Test (DSA + Java)

1. **Collections** — List/Set/Map, when to use which.
2. **OOP** — Encapsulation, inheritance, polymorphism, abstraction.
3. **HashMap internals** — Hashing, buckets, collisions, resize.
4. **String handling** — Immutability, StringBuilder vs String.
5. **Exception handling** — try/catch/finally, checked vs unchecked.
6. **Java 8** — Lambdas, Streams, Optional, functional interfaces.

---

# Practice Exercises

## Exercise A: Equals and hashCode

Implement `equals` and `hashCode` for a `Person` class with `name` and `age` so that two persons with same name and age are considered equal and work correctly in a `HashSet`.

<details>
<summary>Solution</summary>

```java
class Person {
    private final String name;
    private final int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```
</details>

## Exercise B: Stream

Given `List<Integer> list`, return a list of doubled values for elements greater than 10.

<details>
<summary>Solution</summary>

```java
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .collect(Collectors.toList());
```
</details>

## Exercise C: Immutable class

Design an immutable class `DateRange` with `start` and `end` (use `LocalDate` or two ints). Ensure no mutable state leaks.

## Exercise 1.2: Wrappers and ==

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);
```

<details>
<summary>Answer</summary>

`true`. Integer caches -128 to 127; 100 is cached, so both refer to the same cached instance.
</details>

<details>
<summary>Solution</summary>

```java
public final class DateRange {
    private final LocalDate start;
    private final LocalDate end;
    public DateRange(LocalDate start, LocalDate end) {
        this.start = start;
        this.end = end;
    }
    public LocalDate getStart() { return start; }
    public LocalDate getEnd() { return end; }
}
```
</details>

---

# Quick Reference: Common Interview Q&A

**Q: ArrayList vs LinkedList?**  
A: ArrayList: array-based, fast get/set, slower insert in middle. LinkedList: doubly linked, fast insert/remove at ends, slower random access.

**Q: HashSet vs TreeSet?**  
A: HashSet: O(1) average, unordered. TreeSet: sorted, O(log n).

**Q: Checked vs unchecked exception?**  
A: Checked: must be declared or caught. Unchecked: RuntimeException and its subclasses; optional to declare.

**Q: Difference between final, finally, finalize?**  
A: final: constant/unchanged reference/no override/no extend. finally: block that runs after try/catch. finalize: deprecated method called by GC before collection (avoid using).

**Q: Can we override static method?**  
A: No; static methods are hidden in subclasses, not overridden. Overriding is for instance methods and dynamic dispatch.

**Q: Why is main(String[] args)?**  
A: JVM looks for this exact signature as the program entry point; `args` holds command-line arguments.

**Q: What is the diamond problem?**  
A: In multiple inheritance, if A has method m(), B and C override it, and D extends B and C, then D doesn’t know which m() to use. Java avoids this by disallowing multiple class inheritance.

**Q: Can a constructor be private?**  
A: Yes. Used for singletons, factory methods, or utility classes to prevent instantiation.

**Q: What is method hiding?**  
A: When a subclass defines a static method with the same signature as a superclass static method; the subclass method “hides” the superclass one (no polymorphism).

**Q: String pool / interning?**  
A: Literal strings are stored in a pool. `String.intern()` returns the pool instance for the same content, so you can use it to deduplicate and compare by reference when content is equal.

---

# Additional Topics

## final, finally, finalize

- **final:** Variable (constant), method (no override), or class (no extend).
- **finally:** Block after try/catch; runs (unless JVM exits or thread dies) for cleanup.
- **finalize():** Deprecated; called by GC before collection — do not rely on it.

## Pass by value

Java is **pass-by-value** only. For objects, the value passed is the **reference** (copy of the reference). So you can change the object’s state inside a method, but reassigning the parameter does not change the caller’s variable.

## Object class methods

- `equals`, `hashCode` — override together; contract: equal objects must have same hashCode.
- `toString`, `clone` (Cloneable), `getClass`, `wait`, `notify`, `notifyAll`, `finalize`.

## Generics

- Type safety at compile time: `List<String>`, `Map<K,V>`.
- Type erasure: generic type info removed at runtime.
- Bounds: `<T extends Comparable<T>>`, `<? super T>`.

## Optional (Java 8+)

Avoid null; represent optional value: `Optional.of(x)`, `Optional.empty()`, `opt.orElse(default)`, `opt.ifPresent(...)`.

---

# More Exercises

## Exercise D: Predict output

```java
String s = "Hello";
s.concat(" World");
System.out.println(s);
```

<details>
<summary>Answer</summary>

`Hello`. String is immutable; `concat` returns a new string, which is not assigned.
</details>

## Exercise E: HashMap key

What’s wrong with using a mutable class as HashMap key? What if we change the key after putting the entry?

<details>
<summary>Answer</summary>

If the key’s `hashCode()` or `equals()`-relevant fields change after insertion, the map can’t find the entry (wrong bucket / broken equality). Keys should be immutable or never modified while in the map.
</details>

## Exercise F: Try-finally

What is printed?

```java
try {
    System.out.println("A");
    return;
} finally {
    System.out.println("B");
}
```

<details>
<summary>Answer</summary>

A then B. `finally` runs before the method actually returns.
</details>

## Exercise G: Overloading

Which method is called?

```java
void m(Object o) { System.out.println("Object"); }
void m(String s) { System.out.println("String"); }
// call: m(null);
```

<details>
<summary>Answer</summary>

`String`. Both are applicable; `String` is more specific than `Object`, so `m(String)` is chosen. If you add `m(Integer i)`, then `m(null)` is ambiguous (String vs Integer) and won’t compile.
</details>

## Exercise H: Comparable

Sort a list of `Person` by age descending using a Comparator (without implementing Comparable).

<details>
<summary>Solution</summary>

```java
people.sort((a, b) -> Integer.compare(b.getAge(), a.getAge()));
// or
people.sort(Comparator.comparingInt(Person::getAge).reversed());
```
</details>

---

*End of Java notes.*
