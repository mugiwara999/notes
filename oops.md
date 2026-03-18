# Object-Oriented Programming (OOP) — Interview Preparation

Comprehensive, interview-focused OOP notes covering core concepts, design principles, patterns, and common interview questions.

---

## Table of Contents

1. [Core Concepts](#1-core-concepts)
   - [Class and Object](#11-class-and-object)
   - [Encapsulation](#12-encapsulation)
   - [Abstraction](#13-abstraction)
   - [Inheritance](#14-inheritance)
   - [Polymorphism](#15-polymorphism)
2. [Class Relationships](#2-class-relationships)
3. [SOLID Principles](#3-solid-principles)
4. [Other Design Principles](#4-other-design-principles-dry-kiss-yagni)
5. [Design Patterns](#5-design-patterns)
6. [Advanced Topics](#6-advanced-topics)
7. [Common Interview Questions](#7-common-interview-questions)
8. [Key Differences](#8-key-differences)
9. [Quick Revision Cheat Sheet](#9-quick-revision-cheat-sheet)

---

## 1. Core Concepts

### 1.1 Class and Object

| Term | Definition |
|------|-----------|
| **Class** | Blueprint/template that defines attributes (fields) and behaviors (methods). Does not occupy memory by itself. |
| **Object** | A runtime instance of a class. Occupies memory in the heap. |

```java
// Class — blueprint
class Car {
    String brand;
    int speed;

    void accelerate() {
        speed += 10;
    }
}

// Object — instance
Car myCar = new Car();
myCar.brand = "Toyota";
myCar.accelerate();
```

**Key points:**
- A class is a logical entity; an object is a physical entity.
- Objects have **state** (fields), **behavior** (methods), and **identity** (reference/address).
- `new` allocates memory on the heap and calls the constructor.

---

### 1.2 Encapsulation

> **Bundling data (fields) and methods that operate on the data into a single unit, and restricting direct access to internal state.**

- Achieved via **access modifiers** (`private`, `protected`, `public`).
- Exposes controlled access through **getters/setters**.

```java
class BankAccount {
    private double balance; // hidden from outside

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) balance += amount; // validation logic centralized
    }
}
```

**Why it matters in interviews:**
- Prevents unintended modification of state.
- Makes the class easier to maintain and test.
- Changes to internal implementation don't break callers.

**Encapsulation vs Data Hiding:**  
Data hiding is a *subset* of encapsulation — encapsulation also means grouping related data and behavior together.

---

### 1.3 Abstraction

> **Hiding implementation complexity and exposing only the essential interface to the user.**

- Achieved via **abstract classes** and **interfaces**.
- Focus: *what* an object does, not *how*.

```java
// Abstract class — partial abstraction
abstract class Shape {
    abstract double area(); // must be implemented

    void printArea() {
        System.out.println("Area: " + area()); // shared behavior
    }
}

class Circle extends Shape {
    double radius;
    Circle(double r) { this.radius = r; }

    @Override
    double area() { return Math.PI * radius * radius; }
}
```

```java
// Interface — pure abstraction (contract)
interface Drawable {
    void draw(); // implicitly public abstract
}

interface Resizable {
    void resize(double factor);
}

class Rectangle implements Drawable, Resizable {
    public void draw() { System.out.println("Drawing rectangle"); }
    public void resize(double factor) { /* ... */ }
}
```

**Abstract Class vs Interface (Java 8+):**

| | Abstract Class | Interface |
|---|---|---|
| Instantiation | No | No |
| Constructor | Yes | No |
| Fields | Any type | `public static final` only |
| Methods | Abstract + concrete | Abstract + `default` + `static` |
| Multiple inheritance | No | Yes |
| Use when | Shared base implementation | Defining a contract/capability |

---

### 1.4 Inheritance

> **A mechanism where a child class acquires the properties and behaviors of a parent class.**

```java
class Animal {
    String name;
    void eat() { System.out.println(name + " is eating"); }
}

class Dog extends Animal {
    void bark() { System.out.println(name + " is barking"); }
}

Dog d = new Dog();
d.name = "Rex";
d.eat();  // inherited
d.bark(); // own
```

#### Types of Inheritance

```
Single:          A → B
Multilevel:      A → B → C
Hierarchical:    A → B, A → C
Multiple:        A, B → C  (not allowed in Java with classes; use interfaces)
Hybrid:          Mix of above
```

#### The Diamond Problem

```
      A
     / \
    B   C
     \ /
      D
```

- Both `B` and `C` inherit from `A`.
- `D` inherits from both `B` and `C`.
- Ambiguity: which version of `A`'s method does `D` inherit?

**Java's solution:** Classes don't support multiple inheritance. Interfaces support it via `default` methods, and if conflict arises, the implementing class **must** override the method.

```java
interface B { default void hello() { System.out.println("B"); } }
interface C { default void hello() { System.out.println("C"); } }

class D implements B, C {
    @Override
    public void hello() { B.super.hello(); } // must resolve explicitly
}
```

#### Advantages and Disadvantages

| Advantages | Disadvantages |
|-----------|--------------|
| Code reuse | Tight coupling (fragile base class problem) |
| Extensibility | Breaks encapsulation (child can access parent internals) |
| Polymorphism support | Deep hierarchies are hard to maintain |

**Rule of thumb:** Prefer **composition over inheritance** unless a true "is-a" relationship exists.

---

### 1.5 Polymorphism

> **The ability of a single interface to represent different underlying forms (data types or classes).**

#### Compile-Time Polymorphism (Static Dispatch)

Also called **method overloading** — same method name, different parameter signatures.

```java
class MathUtils {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
```

- Resolved at **compile time** by the compiler based on method signature.
- Return type alone cannot differentiate overloaded methods.

#### Runtime Polymorphism (Dynamic Dispatch)

Also called **method overriding** — subclass provides a specific implementation of a parent method.

```java
class Animal {
    void sound() { System.out.println("Some sound"); }
}

class Dog extends Animal {
    @Override
    void sound() { System.out.println("Woof"); }
}

class Cat extends Animal {
    @Override
    void sound() { System.out.println("Meow"); }
}

Animal a = new Dog(); // upcasting
a.sound(); // "Woof" — resolved at runtime via vtable
```

**How it works internally:** JVM uses a **virtual method table (vtable)**. Each object has a reference to its class's vtable. Method calls are dispatched through the vtable at runtime.

#### Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Binding | Compile-time | Runtime |
| Signature | Must differ | Must be same |
| Return type | Can differ | Must be same (or covariant) |
| `static` methods | Can be overloaded | Cannot be overridden (hidden) |
| `final` methods | Can be overloaded | Cannot be overridden |
| Access modifier | No restriction | Cannot be more restrictive |

---

## 2. Class Relationships

### Overview

```
Dependency    ------>     "uses temporarily"         (weakest)
Association   ———————     "knows about"
Aggregation   ◇———————    "has-a" (weak ownership)
Composition   ◆———————    "has-a" (strong ownership)
Inheritance   ———▷        "is-a"
Realization   - - -▷      "implements"              (strongest)
```

---

### 2.1 Association

> One class uses another class — both can exist independently.

- Can be one-to-one, one-to-many, many-to-many.
- No ownership implied.

```java
class Teacher {
    String name;
}

class Student {
    String name;
    Teacher teacher; // association — student knows about teacher
}
```

**Real-world:** A student has a teacher, but both exist independently.

---

### 2.2 Aggregation ("Has-A" — Weak)

> A specialized association where one class "contains" another, but the contained object can exist independently.

```java
class Department {
    String name;
    List<Employee> employees; // aggregation
}

class Employee {
    String name;
    // Employee can exist without Department
}
```

**Real-world:** A department has employees, but employees exist independently of the department.

```
Department ◇————— Employee
(whole)           (part, survives independently)
```

---

### 2.3 Composition ("Has-A" — Strong)

> The contained object **cannot exist** without the container. Lifecycle is tied to the owner.

```java
class House {
    private final Room room; // composition

    House() {
        this.room = new Room(); // created inside, owned by House
    }
}

class Room {
    // Room has no meaning without a House (in this model)
}
```

**Real-world:** A house has rooms. If the house is destroyed, so are the rooms.

```
House ◆————— Room
(whole)       (part, destroyed with whole)
```

**Aggregation vs Composition:**

| | Aggregation | Composition |
|---|---|---|
| Ownership | Weak (shared) | Strong (exclusive) |
| Lifecycle | Independent | Dependent on owner |
| Example | Team → Player | Car → Engine |

---

### 2.4 Dependency

> The weakest relationship — a class uses another class temporarily (as method parameter, local variable, or return type).

```java
class OrderService {
    void placeOrder(PaymentGateway gateway) { // dependency
        gateway.charge(100.0);
    }
}
```

`OrderService` depends on `PaymentGateway` only during method execution.

---

### 2.5 Realization (Interface Implementation)

> A class "realizes" or implements an interface — fulfills the contract.

```java
interface Flyable {
    void fly();
}

class Bird implements Flyable { // realization
    public void fly() { System.out.println("Bird flying"); }
}
```

```
Flyable - - - ▷ Bird
(interface)     (realization)
```

---

### Relationship Summary Table

| Relationship | Coupling | Object Lifecycle | Example |
|---|---|---|---|
| Dependency | Very weak | Independent | Service uses DTO |
| Association | Weak | Independent | Student — Teacher |
| Aggregation | Medium | Independent | Department — Employee |
| Composition | Strong | Dependent | House — Room |
| Inheritance | Strong | N/A (class hierarchy) | Dog — Animal |
| Realization | Contract | N/A | Bird — Flyable |

---

## 3. SOLID Principles

### S — Single Responsibility Principle (SRP)

> A class should have **only one reason to change**.

```java
// BAD — one class does too much
class UserService {
    void saveUser(User u) { /* DB logic */ }
    void sendEmail(User u) { /* email logic */ }
    void generateReport(User u) { /* report logic */ }
}

// GOOD — responsibilities separated
class UserRepository { void save(User u) { /* DB */ } }
class EmailService    { void send(User u) { /* email */ } }
class ReportService   { void generate(User u) { /* report */ } }
```

**Interview tip:** "One reason to change" means one stakeholder/actor should drive changes to the class.

---

### O — Open/Closed Principle (OCP)

> Classes should be **open for extension** but **closed for modification**.

```java
// BAD — adding new shapes requires modifying existing code
class AreaCalculator {
    double calculate(Object shape) {
        if (shape instanceof Circle) { /* ... */ }
        else if (shape instanceof Square) { /* ... */ } // keep modifying
    }
}

// GOOD — extend without modifying
interface Shape { double area(); }

class Circle implements Shape {
    double radius;
    public double area() { return Math.PI * radius * radius; }
}

class Square implements Shape {
    double side;
    public double area() { return side * side; }
}

class AreaCalculator {
    double calculate(Shape shape) { return shape.area(); } // never changes
}
```

---

### L — Liskov Substitution Principle (LSP)

> Subclasses should be **substitutable for their base class** without altering program correctness.

```java
// VIOLATION — Square breaks Rectangle's contract
class Rectangle {
    int width, height;
    void setWidth(int w) { width = w; }
    void setHeight(int h) { height = h; }
    int area() { return width * height; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w) { width = height = w; } // breaks invariant!
}

// Code assuming Rectangle behavior breaks with Square
Rectangle r = new Square();
r.setWidth(5);
r.setHeight(10);
// Expected area: 50, Actual area: 100 — LSP violated
```

**Fix:** Don't inherit `Square` from `Rectangle`. Use a shared `Shape` abstraction instead.

**Key question:** "Can I replace the parent with the child everywhere and have the program still work correctly?"

---

### I — Interface Segregation Principle (ISP)

> Clients should not be forced to depend on **interfaces they don't use**. Prefer many small interfaces over one large one.

```java
// BAD — fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    public void work() { /* ok */ }
    public void eat() { throw new UnsupportedOperationException(); } // robots don't eat
    public void sleep() { throw new UnsupportedOperationException(); }
}

// GOOD — segregated interfaces
interface Workable  { void work(); }
interface Feedable  { void eat(); }
interface Restable  { void sleep(); }

class Human implements Workable, Feedable, Restable { /* all make sense */ }
class Robot implements Workable { /* only what makes sense */ }
```

---

### D — Dependency Inversion Principle (DIP)

> High-level modules should not depend on low-level modules. **Both should depend on abstractions.**

```java
// BAD — high-level depends directly on low-level
class OrderService {
    private MySQLDatabase db = new MySQLDatabase(); // tightly coupled
    void placeOrder() { db.save(/* ... */); }
}

// GOOD — depend on abstraction
interface Database { void save(Object data); }

class MySQLDatabase implements Database {
    public void save(Object data) { /* MySQL */ }
}

class OrderService {
    private final Database db; // depends on abstraction

    OrderService(Database db) { this.db = db; } // injected

    void placeOrder() { db.save(/* ... */); }
}
```

**Relationship with Dependency Injection:** DIP is a *principle*; Dependency Injection is a *pattern/technique* to achieve it.

---

### SOLID Summary

| Principle | One-liner |
|-----------|-----------|
| **S**RP | One class, one job |
| **O**CP | Extend, don't modify |
| **L**SP | Subtypes must be substitutable |
| **I**SP | Small, focused interfaces |
| **D**IP | Depend on abstractions, not concretions |

---

## 4. Other Design Principles: DRY, KISS, YAGNI

### DRY — Don't Repeat Yourself

> Every piece of knowledge must have a **single, authoritative representation** in a system.

- Duplication = two places to update when logic changes = bugs.
- Extract repeated logic into methods, classes, constants, or configs.

```java
// BAD
double salaryA = baseSalary * 1.2 * 0.85;
double salaryB = baseSalary * 1.2 * 0.85; // same formula, copy-pasted

// GOOD
double calculateNetSalary(double base) {
    return base * 1.2 * 0.85;
}
```

---

### KISS — Keep It Simple, Stupid

> Systems work best when they are **kept simple**. Avoid unnecessary complexity.

- Prefer readable, straightforward code over clever one-liners.
- Simple code is easier to test, debug, and maintain.

```java
// Overly clever
boolean isEven = (n & 1) == 0;

// KISS (unless performance-critical and team knows bit ops)
boolean isEven = n % 2 == 0;
```

---

### YAGNI — You Aren't Gonna Need It

> Don't add functionality until it is **actually needed**.

- Over-engineering adds complexity without immediate value.
- Build for current requirements; refactor when new requirements arrive.

**Bad:** Adding a plugin system, caching layer, or abstract factory "just in case."  
**Good:** Solve the current problem cleanly; extend when requirements demand it.

---

## 5. Design Patterns

> **Design patterns** are reusable solutions to commonly occurring design problems.

### 5.1 Creational Patterns

#### Singleton

> Ensures a class has **only one instance** and provides a global access point.

**When to use:** Logger, configuration manager, connection pool, thread pool.

```java
class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) { // double-checked locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Interview pitfalls:**
- Not thread-safe without `volatile` + double-checked locking (or use `enum`).
- Hard to unit test (global state).
- `enum` Singleton is the safest approach in Java (handles serialization too).

```java
enum SingletonEnum {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
```

---

#### Factory Method

> Defines an interface for creating an object, but lets **subclasses decide** which class to instantiate.

**When to use:** When the exact type of object to create isn't known until runtime.

```java
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) { System.out.println("Email: " + message); }
}

class SMSNotification implements Notification {
    public void send(String message) { System.out.println("SMS: " + message); }
}

class NotificationFactory {
    public static Notification create(String type) {
        return switch (type) {
            case "EMAIL" -> new EmailNotification();
            case "SMS"   -> new SMSNotification();
            default      -> throw new IllegalArgumentException("Unknown type: " + type);
        };
    }
}

// Usage
Notification n = NotificationFactory.create("EMAIL");
n.send("Hello!");
```

---

#### Abstract Factory

> A factory of factories — creates **families of related objects** without specifying concrete classes.

**When to use:** UI toolkit (Windows vs Mac buttons, checkboxes), cross-platform components.

```java
interface Button   { void render(); }
interface Checkbox { void render(); }

class WindowsButton   implements Button   { public void render() { System.out.println("Windows Button"); } }
class WindowsCheckbox implements Checkbox { public void render() { System.out.println("Windows Checkbox"); } }
class MacButton       implements Button   { public void render() { System.out.println("Mac Button"); } }
class MacCheckbox     implements Checkbox { public void render() { System.out.println("Mac Checkbox"); } }

interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

class WindowsFactory implements GUIFactory {
    public Button createButton()     { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

class MacFactory implements GUIFactory {
    public Button createButton()     { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}
```

---

#### Builder

> Separates the **construction** of a complex object from its **representation**.

**When to use:** Objects with many optional parameters (avoid telescoping constructors).

```java
class Pizza {
    private String size;
    private boolean cheese, pepperoni, mushrooms;

    private Pizza(Builder b) {
        this.size = b.size;
        this.cheese = b.cheese;
        this.pepperoni = b.pepperoni;
        this.mushrooms = b.mushrooms;
    }

    static class Builder {
        private String size;
        private boolean cheese, pepperoni, mushrooms;

        Builder(String size) { this.size = size; }

        Builder cheese()    { this.cheese = true;    return this; }
        Builder pepperoni() { this.pepperoni = true;  return this; }
        Builder mushrooms() { this.mushrooms = true;  return this; }

        Pizza build() { return new Pizza(this); }
    }
}

// Usage — fluent API
Pizza p = new Pizza.Builder("Large").cheese().pepperoni().build();
```

---

### 5.2 Structural Patterns

#### Adapter

> Converts the **interface of a class** into another interface that clients expect. Bridges incompatible interfaces.

**When to use:** Integrating legacy code, third-party libraries with incompatible APIs.

```java
// Existing interface expected by client
interface MediaPlayer {
    void play(String fileName);
}

// Incompatible third-party library
class VLCPlayer {
    void playVLC(String file) { System.out.println("Playing VLC: " + file); }
}

// Adapter wraps VLCPlayer to implement MediaPlayer
class VLCAdapter implements MediaPlayer {
    private VLCPlayer vlc = new VLCPlayer();

    public void play(String fileName) {
        vlc.playVLC(fileName); // translates the call
    }
}

// Client code never changes
MediaPlayer player = new VLCAdapter();
player.play("movie.vlc");
```

---

#### Decorator

> Attaches **additional responsibilities** to an object dynamically. A flexible alternative to subclassing.

**When to use:** Adding features (logging, caching, compression) at runtime without modifying original class.

```java
interface Coffee {
    String getDescription();
    double getCost();
}

class SimpleCoffee implements Coffee {
    public String getDescription() { return "Simple Coffee"; }
    public double getCost()        { return 1.0; }
}

// Decorator base
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    CoffeeDecorator(Coffee c) { this.coffee = c; }
}

class MilkDecorator extends CoffeeDecorator {
    MilkDecorator(Coffee c) { super(c); }
    public String getDescription() { return coffee.getDescription() + ", Milk"; }
    public double getCost()        { return coffee.getCost() + 0.25; }
}

class SugarDecorator extends CoffeeDecorator {
    SugarDecorator(Coffee c) { super(c); }
    public String getDescription() { return coffee.getDescription() + ", Sugar"; }
    public double getCost()        { return coffee.getCost() + 0.10; }
}

// Usage
Coffee c = new SugarDecorator(new MilkDecorator(new SimpleCoffee()));
System.out.println(c.getDescription()); // Simple Coffee, Milk, Sugar
System.out.println(c.getCost());        // 1.35
```

---

### 5.3 Behavioral Patterns

#### Observer

> Defines a **one-to-many dependency** so that when one object changes state, all its dependents are notified automatically.

**When to use:** Event systems, pub/sub, MVC (view reacts to model changes), stock tickers.

```java
import java.util.*;

interface Observer {
    void update(String event);
}

class EventBus {
    private List<Observer> observers = new ArrayList<>();

    void subscribe(Observer o)   { observers.add(o); }
    void unsubscribe(Observer o) { observers.remove(o); }

    void publish(String event) {
        for (Observer o : observers) o.update(event);
    }
}

class EmailListener implements Observer {
    public void update(String event) { System.out.println("Email alert: " + event); }
}

class SMSListener implements Observer {
    public void update(String event) { System.out.println("SMS alert: " + event); }
}

// Usage
EventBus bus = new EventBus();
bus.subscribe(new EmailListener());
bus.subscribe(new SMSListener());
bus.publish("Order shipped"); // both get notified
```

---

#### Strategy

> Defines a **family of algorithms**, encapsulates each one, and makes them interchangeable. Lets the algorithm vary independently from clients.

**When to use:** Sorting strategies, payment methods, compression algorithms, routing.

```java
interface SortStrategy {
    void sort(int[] arr);
}

class BubbleSort implements SortStrategy {
    public void sort(int[] arr) { /* bubble sort */ System.out.println("Bubble Sort"); }
}

class QuickSort implements SortStrategy {
    public void sort(int[] arr) { /* quick sort */ System.out.println("Quick Sort"); }
}

class Sorter {
    private SortStrategy strategy;

    Sorter(SortStrategy strategy) { this.strategy = strategy; }

    void setStrategy(SortStrategy s) { this.strategy = s; } // swap at runtime

    void sort(int[] arr) { strategy.sort(arr); }
}

// Usage
Sorter sorter = new Sorter(new QuickSort());
sorter.sort(new int[]{5, 2, 8});

sorter.setStrategy(new BubbleSort()); // change strategy without changing Sorter
sorter.sort(new int[]{5, 2, 8});
```

---

### Pattern Summary

| Pattern | Category | Problem Solved | Key Idea |
|---------|----------|---------------|---------|
| Singleton | Creational | Single instance needed | Private constructor + static accessor |
| Factory Method | Creational | Object creation varies by type | Let factory decide the type |
| Abstract Factory | Creational | Families of related objects | Factory of factories |
| Builder | Creational | Complex object construction | Step-by-step with fluent API |
| Adapter | Structural | Incompatible interfaces | Wrapper that translates |
| Decorator | Structural | Add behavior dynamically | Wrap objects, add responsibility |
| Observer | Behavioral | React to state changes | Publish-subscribe |
| Strategy | Behavioral | Interchangeable algorithms | Encapsulate and swap behaviors |

---

## 6. Advanced Topics

### 6.1 Composition vs Inheritance

| | Inheritance | Composition |
|---|---|---|
| Relationship | "is-a" | "has-a" |
| Coupling | High (tightly coupled) | Low (loosely coupled) |
| Flexibility | Rigid (compile-time) | Flexible (runtime swap) |
| Code reuse | Via extending | Via delegation |
| Test ease | Harder (need parent) | Easier (mock dependencies) |

**When to prefer Inheritance:**
- A true "is-a" relationship exists (`Dog` is an `Animal`).
- You want polymorphic behavior.
- The hierarchy is shallow and stable.

**When to prefer Composition:**
- You want to reuse behavior from multiple sources.
- The "is-a" relationship is forced/unnatural.
- You need to change behavior at runtime.
- You want better testability.

```java
// Inheritance — tightly coupled
class LoggingList extends ArrayList<String> {
    @Override
    public boolean add(String s) {
        System.out.println("Adding: " + s);
        return super.add(s); // fragile: depends on ArrayList internals
    }
}

// Composition — flexible and safe
class LoggingList {
    private final List<String> list = new ArrayList<>();

    public boolean add(String s) {
        System.out.println("Adding: " + s);
        return list.add(s); // delegate; won't break if ArrayList changes
    }
}
```

> **Golden rule:** Favor composition over inheritance. Use inheritance only when the "is-a" relationship is clear, stable, and meaningful.

---

### 6.2 Immutability

> An object is **immutable** if its state cannot be changed after creation.

**Rules for immutable class (Java):**
1. Declare class as `final` (prevent subclassing).
2. All fields `private final`.
3. No setters.
4. Deep-copy mutable objects in constructor and getters.
5. Initialize all fields via constructor.

```java
final class Money {
    private final double amount;
    private final String currency;

    Money(double amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    double getAmount()   { return amount; }
    String getCurrency() { return currency; }

    Money add(Money other) {
        if (!this.currency.equals(other.currency))
            throw new IllegalArgumentException("Currency mismatch");
        return new Money(this.amount + other.amount, this.currency); // new object
    }
}
```

**Benefits:**
- Thread-safe by default (no synchronization needed).
- Easier to reason about (no hidden state changes).
- Safe to share and cache.
- Can be used as HashMap keys reliably.

**Examples in Java:** `String`, `Integer`, `LocalDate`, `BigDecimal`.

---

### 6.3 Method Overloading vs Overriding

| Aspect | Overloading | Overriding |
|--------|-------------|-----------|
| Definition | Same name, different params | Same name, same params, in subclass |
| Binding | Static (compile-time) | Dynamic (runtime) |
| Inheritance | Not required | Required |
| Return type | Can differ freely | Must be same or covariant |
| `static` | Can overload | Cannot override (method hiding) |
| `final` | Can overload | Cannot override |
| `private` | Can overload | Cannot override (not inherited) |
| `@Override` | Not applicable | Recommended (catches mistakes) |
| Access modifier | Any | Cannot be more restrictive |
| Exceptions | Can add or change | Cannot throw new/wider checked exceptions |

```java
class Parent {
    int getValue() { return 10; }
}

class Child extends Parent {
    @Override
    Integer getValue() { return 20; } // covariant return type is allowed
}
```

---

### 6.4 Dependency Injection (DI)

> **Providing dependencies to an object from the outside** rather than having the object create them itself.

**Three types:**
1. **Constructor Injection** (preferred — makes dependencies explicit and final)
2. **Setter Injection** (for optional dependencies)
3. **Field/Interface Injection** (via reflection — avoid in production code)

```java
// Without DI — tightly coupled
class UserService {
    private UserRepository repo = new UserRepository(); // created internally
}

// With Constructor Injection — loosely coupled
class UserService {
    private final UserRepository repo;

    UserService(UserRepository repo) {
        this.repo = repo; // injected from outside
    }
}

// Usage
UserRepository repo = new UserRepository();
UserService service = new UserService(repo);

// In tests — easy to mock
UserRepository mockRepo = mock(UserRepository.class);
UserService service = new UserService(mockRepo);
```

**DI Containers:** Spring, Guice, Dagger — they manage object creation and wiring automatically.

**DI achieves DIP:** Both the high-level (`UserService`) and low-level (`UserRepository`) depend on an abstraction (`interface`), not each other.

---

## 7. Common Interview Questions

### Core OOP

**Q: What are the four pillars of OOP?**  
A: Encapsulation, Abstraction, Inheritance, Polymorphism.

**Q: Can we achieve abstraction without interfaces?**  
A: Yes — abstract classes also provide abstraction. Interfaces provide 100% abstraction (prior to Java 8 default methods); abstract classes provide partial abstraction.

**Q: What is the difference between method overloading and overriding?**  
A: Overloading = same name, different parameters, resolved at compile time. Overriding = redefining a parent method in a subclass, resolved at runtime via dynamic dispatch.

**Q: What is an abstract class and can it have a constructor?**  
A: An abstract class cannot be instantiated but *can* have a constructor — it is called via `super()` from the subclass constructor.

**Q: What happens when you call an overridden method from a constructor?**  
A: The subclass version of the method is called even though the superclass constructor is executing. This is dangerous — the subclass may not be fully initialized yet.

```java
class Base {
    Base() { print(); } // calls overridden method!
    void print() { System.out.println("Base"); }
}
class Derived extends Base {
    int x = 10;
    @Override void print() { System.out.println(x); } // prints 0, not 10!
}
new Derived(); // prints 0 — x not yet initialized
```

**Q: What is the diamond problem and how does Java handle it?**  
A: When a class inherits from two classes that both have the same method from a common ancestor. Java avoids this for classes (no multiple class inheritance). For interfaces with `default` methods, the implementing class must explicitly resolve the conflict.

**Q: Is Java fully object-oriented?**  
A: No — Java has primitive types (`int`, `char`, etc.) which are not objects. Purists say this violates OOP. Languages like Smalltalk are "more" OOP.

---

### Design

**Q: What is the difference between Aggregation and Composition?**  
A: Both are "has-a" relationships. Aggregation = weak ownership (child can exist without parent). Composition = strong ownership (child's lifecycle is tied to parent).

**Q: How is the Strategy pattern different from the State pattern?**  
A: Strategy = interchangeable algorithms (client chooses). State = object behavior changes based on internal state (state transitions are managed internally).

**Q: When would you use Singleton?**  
A: For shared stateless services (logger, config, connection pool). Avoid when it makes testing hard or creates hidden global dependencies.

**Q: What problem does the Builder pattern solve?**  
A: Telescoping constructor anti-pattern — when a class has many optional parameters, constructors become unreadable. Builder provides a fluent, readable way to construct complex objects.

---

### Common Misconceptions

- **Encapsulation ≠ data hiding.** Encapsulation is about bundling + controlled access. Data hiding is a consequence.
- **Abstraction ≠ interface.** Abstraction is a concept; interfaces and abstract classes are mechanisms to achieve it.
- **Inheritance ≠ code reuse.** Inheritance creates coupling. Use it for polymorphism; use composition for reuse.
- **Overriding doesn't work with `static` methods.** Static methods are resolved at compile time based on the reference type — this is *method hiding*, not overriding.
- **`final` class ≠ immutable.** A `final` class can't be subclassed, but its fields can still be mutable unless declared `final`.

---

## 8. Key Differences

### Abstraction vs Encapsulation

| | Abstraction | Encapsulation |
|---|---|---|
| **What** | Hiding complexity/implementation | Hiding data (wrapping + access control) |
| **Why** | Reduce complexity for caller | Protect internal state |
| **How** | Abstract classes, interfaces | Access modifiers, getters/setters |
| **Level** | Design level | Implementation level |
| **Example** | `List` interface hides LinkedList vs ArrayList internals | `private balance` with `getBalance()` |

---

### Interface vs Abstract Class

| | Interface | Abstract Class |
|---|---|---|
| Instantiation | No | No |
| Constructor | No | Yes |
| Fields | `public static final` only | Any |
| Methods (Java 8+) | Abstract, `default`, `static` | Abstract + concrete |
| Multiple inheritance | Yes (multiple interfaces) | No |
| Use for | Capability contract ("can do") | Shared base ("is a kind of") |
| `extends`/`implements` | `implements` | `extends` |

---

### Inheritance vs Composition

| | Inheritance | Composition |
|---|---|---|
| Relationship | "is-a" | "has-a" |
| Coupling | Tight | Loose |
| Flexibility | Compile-time | Runtime |
| Testing | Harder | Easier |
| Code reuse | Via parent | Via delegation |

---

### `==` vs `.equals()` (for context in OOP)

| | `==` | `.equals()` |
|---|---|---|
| For primitives | Compares values | N/A |
| For objects | Compares references (addresses) | Compares logical equality |
| Override? | No | Yes (should also override `hashCode`) |

---

### Shallow Copy vs Deep Copy

| | Shallow Copy | Deep Copy |
|---|---|---|
| What | Copies references | Copies entire object graph |
| Nested objects | Shared | Independent copies |
| Method | `Object.clone()` default | Custom, copy constructors, serialization |

---

## 9. Quick Revision Cheat Sheet

### OOP Pillars

| Pillar | One-liner |
|--------|-----------|
| **Encapsulation** | Bundle data + behavior; expose via controlled interface |
| **Abstraction** | Show what, hide how |
| **Inheritance** | Child acquires parent's properties/behaviors ("is-a") |
| **Polymorphism** | One interface, many implementations |

### Class Relationships (Weakest → Strongest)

```
Dependency → Association → Aggregation → Composition → Inheritance
"uses"       "knows"       "has-a(weak)" "has-a(strong)"  "is-a"
```

### SOLID (One line each)

- **S** — One class, one reason to change
- **O** — Extend behavior, don't modify existing code
- **L** — Subclasses must honor parent's contract
- **I** — Prefer small, specific interfaces
- **D** — Depend on abstractions, not concrete classes

### Design Patterns (Quick recall)

| Pattern | One-liner |
|---------|-----------|
| Singleton | One instance, global access |
| Factory | Hide object creation logic |
| Abstract Factory | Create families of related objects |
| Builder | Step-by-step complex object creation |
| Adapter | Make incompatible interfaces work together |
| Decorator | Add behavior to objects at runtime |
| Observer | Notify many when one changes |
| Strategy | Swap algorithms at runtime |

### Advanced Topics Quick Reference

| Topic | Key Takeaway |
|-------|-------------|
| **Composition > Inheritance** | Use "has-a" for flexibility and low coupling |
| **Immutability** | `final` class + `private final` fields + no setters + deep copy |
| **Overloading** | Same name, different params — compile-time |
| **Overriding** | Same signature in subclass — runtime via vtable |
| **Dependency Injection** | Inject dependencies from outside; enables DIP and testability |
| **Diamond Problem** | Multiple inheritance ambiguity — resolved via explicit override in Java |
| **DRY** | Single source of truth — no duplication |
| **KISS** | Simple is better than clever |
| **YAGNI** | Build what you need now, not what you might need later |

### Must-Remember Traps

- Calling overridden methods from constructors → subclass fields not yet initialized.
- `static` methods are **hidden**, not overridden — behavior depends on reference type.
- `final` class ≠ immutable object.
- Aggregation ≠ Composition — difference is lifecycle ownership.
- LSP violation: `Square extends Rectangle` is a classic pitfall.
- Singleton without `volatile` is not thread-safe.
- `equals()` override requires `hashCode()` override — contract.
