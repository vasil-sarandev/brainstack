# Creational Software Design Patterns

#deep-dive

These are the design patterns that deal with the creation of an object.

---
## List of Creational Design Patterns

- [_Singleton_](#singleton) - The singleton pattern restricts the initialization of a class to ensure that only one instance of the class can be created.
- [_Factory_](#factory) - The factory pattern takes out the responsibility of instantiating a object from the class to a Factory class.
- [_Abstract Factory_](#abstract-factory) - Allows us to create a Factory for factory classes.
- [_Builder_](#builder) - Creating an object step by step and a method to finally get the object instance.
- _Prototype_ - Creating a new object instance from another similar instance and then modify according to our requirements.

---
## Singleton

Singleton pattern restricts the instantiation of a class and ensures that only one instance of the class exists. The singleton class must provide a global access point to get the instance of the class. Singleton pattern is used for logging, drivers objects, caching, and thread pool.

To implement a singleton pattern, we have different approaches, but all of them have the following common concepts.

- Private constructor to restrict instantiation of the class from other classes.
- Private static variable of the same class that is the only instance of the class.
- Public static method that returns the instance of the class, this is the global access point for the outer world to get the instance of the singleton class.


```JAVA
// eager singleton
public class EagerInitializedSingleton {
    private static final EagerInitializedSingleton instance = new EagerInitializedSingleton();

    // private constructor to avoid client applications using the constructor
    private EagerInitializedSingleton(){}

    public static EagerInitializedSingleton getInstance() {
        return instance;
    }
}
// lazy singleton
public class LazyInitializedSingleton {
    private static LazyInitializedSingleton instance;

    private LazyInitializedSingleton(){}

    public static LazyInitializedSingleton getInstance() {
        if (instance == null) {
            instance = new LazyInitializedSingleton();
        }
        return instance;
    }
}
```

---
## Factory
The factory design pattern is used when we have a superclass with multiple sub-classes and based on input, we need to return one of the sub-class.

This pattern takes out the responsibility of the instantiation of a class from the client program to the factory class.

Factory classes are useful when you need a complicated process for constructing the object, when the construction need a dependency that you do not want for the actual class, when you need to construct different objects etc.

```JavaScript
class Car {
  constructor(brand, model) {
    this.vehicleType = 'Car';
    this.brand = brand;
    this.model = model;
  }
  drive() {
    return `Driving a ${this.brand} ${this.model} car.`;
  }
}
class Bike {
  constructor(brand, model) {
    this.vehicleType = 'Bike';
    this.brand = brand;
    this.model = model;
  }
  ride() {
    return `Riding a ${this.brand} ${this.model} bike.`;
  }
}
// Vehicle factory that creates vehicles based on type
class VehicleFactory {
  static createVehicle(type, brand, model) {
    switch (type) {
      case 'car':
        return new Car(brand, model);
      case 'bike':
        return new Bike(brand, model);
      default:
        throw new Error('Vehicle type not supported.');
    }
  }
}
```

---
## Abstract Factory

In the Abstract Factory pattern, we get rid of if-else block and have a factory class for each sub-class. Then an Abstract Factory class that will return the sub-class based on the input factory class. At first, it seems confusing but once you see the implementation, it’s really easy to grasp and understand the minor difference between Factory and Abstract Factory pattern. Like our factory pattern post, we will use the same superclass and sub-classes.

```TypeScript
abstract class VehicleAbstractFactory {
    createVehicle(brand, model): Vehicle
}
class CarFactory extends VehicleAbstractFactory {
    createVehicle = (brand, model) => new Car(brand, model)
}
class BikeFactory extends VehicleAbstractFactory {
    createVehicle = (brand, model) => new Bike(brand, model)
}

```

---
## Builder

Builder pattern was introduced to solve some of the problems with Factory and Abstract Factory design patterns when the Object contains a lot of attributes. There are three major issues with Factory and Abstract Factory design patterns when the Object contains a lot of attributes.

Too Many arguments to pass from client program to the Factory class that can be error prone because most of the time, the type of arguments are same and from client side its hard to maintain the order of the argument.
Some of the parameters might be optional but in Factory pattern, we are forced to send all the parameters and optional parameters need to send as NULL.
If the object is heavy and its creation is complex, then all that complexity will be part of Factory classes that is confusing.

```JavaScript
class Dashboard {
  constructor(builder) {
    this.widgets = builder.widgets;
  }
}

class DashboardBuilder {
  constructor() {
    this.widgets = [];
  }
  addChart(type, data) {
    this.widgets.push({ type: 'chart', chartType: type, data });
    return this;
  }
  addTable(data) {
    this.widgets.push({ type: 'table', data });
    return this;
  }
  addNotification(message) {
    this.widgets.push({ type: 'notification', message });
    return this;
  }
  build() {
    return new Dashboard(this);
  }
}

// Usage
const dashboard = new DashboardBuilder()
  .addChart('line', [1, 2, 3])
  .addTable({ header: ['Name', 'Age'], rows: [['Alice', 30], ['Bob', 25]] })
  .addNotification('Welcome to your dashboard!')
  .build();
```
