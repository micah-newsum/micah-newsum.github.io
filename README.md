## Welcome to my "All Things Software Engineering Reference Guide"
### Contents
- [Object-Oriented Design Principles](https://github.com/micah-newsum/micah-newsum.github.io.git)
  - [Principle of Least Knowledge (Law of Demeter)](https://github.com/micah-newsum/micah-newsum.github.io.git)
 
#### Object-Oriented Design Principles
##### Principle of Least Knowledge (Law of Demeter)
The "Principle of Least Knowledge," also known as the Law of Demeter, is a design principle that encourages reducing the dependencies between objects. It essentially states that a method of a class should only call methods on:
- The object itself.
- Objects passed in as arguments.
- Objects it creates.
- Direct components (e.g., field) of the object.

Following this principle prevents us from creating designs that have a large number of classes coupled together so that changes in one part of the system cascade to other parts. Designs with lots of dependencies between many classes make for fragile systems that will be costly to maintain and complex to understand.

##### Example
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
