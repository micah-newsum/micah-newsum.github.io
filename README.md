## Software Engineering Reference Guide
My curated reference guide for all things SWE.
### Contents
- [Object-Oriented Programming](#object-oriented-programming)
    - [The Four Pillars of Object-Oriented Programming](#the-four-pillars-of-object-oriented-programming)
        - [Encapsulation](#encapsulation)
        - [Inheritance](#inheritance)
        - [Polymorphism](#polymorphism)
        - [Abstraction](#abstraction)
    - [Design Patterns](#design-patterns)
        - [Strategy Pattern](#strategy-pattern)
        - [Adapter](#adapter)
        - [Facade](#facade)
    - [Object-Oriented Design Principles](#object-oriented-design-principles)
        - [Principle of Least Knowledge](#principle-of-least-knowledge)
- [System Design](#system-design)
  - [REST API](#rest-api)
- [Testing](#testing)
  - [REST API Testing](#rest-api-testing)
- [Software Project Management](#software-project-management)
  - TBA
- [Cloud](#cloud)
  - [AWS](#aws)
  - [Terraform](#terraform)
    - [Resource Drift](#resource-drift)
- [Soft Skills](#soft-skills)
  - [Leadership](#leadership)
    - [Providing Feedback](#providing-feedback)
  - [Being Lead](#being-lead)

### Object-Oriented Programming
Object-Oriented Programming (OOP) is a programming paradigm centered around the concept of "objects," which represent real-world entities, or abstract concepts.

#### The Four Pillars of Object-Oriented Programming
The four pillars of OOP are encapsulation, inheritance, polymorphism, and abstraction. These pillars lay the foundation for creating modular, resuable, and maintainable code.
#### Encapsulation
Encapsulation is the bundling of data (attributes) and methods (functions) that operate on the data into a single unit or class. It restricts direct access to an object's components to prevent unintended consequences resulting from misuse of the methods and data. 

Encapsulation helps to protect an object's internal state and expose a controlled interface for interacting with that object.

**Example**
```typescript
class Account {
    private balance: number = 0;

    // Method to deposit money
    deposit(amount: number): void {
        if (amount > 0) {
            this.balance += amount;
        }
    }

    // Method to retrieve the current balance
    getBalance(): number {
        return this.balance;
    }
}
```
Notice there's no `setter` for the private `balance` attribute. This ensures modifications to `balance` are done so though a controlled interface, in this case `deposit(amount)`.
#### Inheritance
Inheritance allows a class (child or subclass) to inherit attributes and methods from an existing class (parent or superclass). The child class can add its own attributes and methods or override the inherited ones.

Inheritance promotes code reuse and establishes a hierarchy between classes.

**Example**
```typescript
class Vehicle {
    constructor(public make: string, public model: string) {}

    startEngine(): void {
        console.log('Engine started');
    }
}

class Car extends Vehicle {
    constructor(make: string, model: string, public doors: number) {
        super(make, model); // Call the parent class constructor
    }

    lockDoors(): void {
        console.log('Doors locked');
    }
}
```
The `Car` class extends the more generic `Vehicle` class, reusing the common attributes and methods, while defining it's own `lockDoors()` function.

### Polymorphism
Polymorphism allows objects of different classes to be treated as objects of a common supertype. In addition, it enables a single function to operate on objects of different types. 

Polymorphism simplifies code by providing a shared interface for a generic action. It also makes code more flexible and extensible by adding new implementations without altering existing code.

**Example**
```typescript
interface Vehicle {
    move(): void
}

class Car implements Vehicle {
    move(): void {
        console.log('The car is driving');
    }
}

class Bike implements Vehicle {
    move(): void {
        console.log('The bike is riding');
    }
}

// Polymorphism in action
const vehicles: Vehicle[] = [new Car(), new Bike()];
vehicles.forEach(vehicle => vehicle.move());
```
The interface `Vehicle` type defines a single function `move()` which any subtype, for example `Car` or `Bike`, implements.

### Abstraction
Abstraction serves to hide complex implementations and only expose the necessary features or behaviors. It focuses on what an object does rather than how it does it. 

Abstraction reduces the cognitive load required by the developer when interacting with an object. Developers are able to focus on higher level interactions without needing to understand all the intricate details. 
```typescript
abstract class Animal {
    abstract makeSound(): void; // Abstract method
}

class Dog extends Animal {
    makeSound(): void {
        console.log('Woof! Woof!');
    }
}

class Cat extends Animal {
    makeSound(): void {
        console.log('Meow! Meow!');
    }
}

const animals: Animal[] = [new Dog(), new Cat()];
animals.forEach(animal => animal.makeSound());
```
Clients don't need to know how each `Animal` makes a sound, but simply that they're capable making a sound by calling `makeSound()`. In addition, each `makeSound()` implementation is decoupled from the parent class.

#### Design Patterns
##### Strategy Pattern
TBA
#### Object-Oriented Design Principles
##### Principle of Least Knowledge
The "Principle of Least Knowledge," also known as the Law of Demeter, is a design principle that encourages reducing the dependencies between objects. It essentially states that a method of a class should only call methods on:
- The object itself.
- Objects passed in as arguments.
- Objects it creates.
- Direct components (e.g., field) of the object.

Following this principle prevents us from creating designs that have a large number of classes coupled together so that changes in one part of the system cascade to other parts. Designs with lots of dependencies between many classes make for fragile systems that will be costly to maintain and complex to understand.

**Example**

Imagine you're creating an `Order` class that contains a collection of `OrderLine` objects. The goal is to interact with the collection in a way that does not expose the internal details of the `OrderLine` objects, or allow too much traversal through the object graph. This way, you limit the interactions with the collection to specific methods provided by the `Order` class.
```typescript
class Order {
    private orderLines: OrderLine[] = [];

    // Add an order line
    addOrderLine(orderLine: OrderLine): void {
        this.orderLines.push(orderLine);
    }

    // Remove an order line
    removeOrderLine(orderLineId: string): void {
        this.orderLines = this.orderLines.filter(line => line.id !== orderLineId);
    }

    // Get total price of all order lines
    getTotalPrice(): number {
        return this.orderLines.reduce((total, line) => total + line.getPrice(), 0);
    }

    // Get total quantity of all order lines
    getTotalQuantity(): number {
        return this.orderLines.reduce((total, line) => total + line.getQuantity(), 0);
    }
}
```
A common mistake is to expose the internal collection of `orderLines` directly to `Order` clients.
```typescript
// This is not recommended
getOrderLines(): OrderLine[] {
  return this.orderLines;
}
```
Instead, a better approach is to provide methods that perform specific operations on the collection. For example, create methods on the `Order` class that handle the specific needs:
```typescript
// A better approach
findOrderLineById(orderLineId: string): OrderLine | undefined {
    return this.orderLines.find(line => line.id === orderLineId);
}
```
This ensures that the internal details of the `OrderLine` objects are hidden and that the `Order` class maintains control over how data is accessed and modified.

**Tradeoffs**

Following the Principle of Least Knowledge comes with tradeoffs. For example, imagine you have a `Car` class with an `Engine` component, which in turn has a `FuelTank` component, and you would like to provide a method that returns the fuel level of the `Car`.
```java
public class Car {
    private Engine engine;

    public Car() {}

    public int getFuelLevel() {
        return engine.getFuelLevel();
    }
}

public class Engine {
    private FuelTank fuelTank;

    public Engine() {}

    public int getFuelLevel() {
        return fuelTank.getFuelLevel();
    }
}

public class FuelTank {
    private int fuelLevel;

    public FuelTank() {}

    public int getFuelLevel() {
        return fuelLevel;
    }
}
```
Abiding by this principle increases complexity because it's now less clear where the `fuelLevel` data is actually coming from. Additionally, it forces us to write more boilerplate code, which in this example is the repeated `getFuelLevel()` method in each class. Each method simply delegates calls to the next object in the chain without adding any functionality.
## Cloud
### AWS
### Terraform
#### Resource Drift
The Terraform state file contains all resources Terraform manages. Therefore, you should not make manual changes to resources controlled by Terraform; otherwise, the state file will become out of sync, or "drift," from the real infrastructure. If your state and configuration do not match your infrastructure, Terrafrom will attempt to reconcile your infrastructure, potentially leading to unintended changes.

**References**
- https://developer.hashicorp.com/terraform/tutorials/state/resource-drift
## Soft Skills
### Leadership
"There are no bad teams, only bad leaders." - *Extreme Ownership: How U.S. Navy SEALs Lead and Win* 
#### Providing Feedback
 - Set up a private meeting, one-on-one, and ideally in-person.
 - Start with empathy and understanding.
    - Recognize their contributions and hard work.
    - Express your intentions.
        - Clarify the purpose of the feedback is to help them grown and improve, not to criticize or undermine them.
        - Seek their perspective
- Address the issue directly but kindly
    - Describe what you've observed
    - Ask open-ended questions
        - Invite them to share their thoughts
    - Listen actively
- Provide constructive feedback
    - Be specific
        - Clearly articulate the areas where improvement is needed. 
    - Relate to goals
        - Tie your feedback to their personal and professional goals.
    - Encourage a growth mindset
- Collaborate on a Plan Forward
    - Ask for their input
    - Set clear expectations
    - Offer support
- Follow Up
    - Check-In Regularaly
    - Acknowledge Progress
- Be Patient and Persistent
    - Give them time
    - Stay open
- Consider the Bigger Picture
    - Assess Team Dynamics