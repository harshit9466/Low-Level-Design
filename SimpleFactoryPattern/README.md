# Simple Factory Pattern Explained

This document explains the difference between the code in `WithoutFactoryPattern` and `WithSimpleFactoryPattern`, illustrating the purpose and benefits of the Simple Factory design pattern.

---

## `WithoutFactoryPattern` - The Problem

In this version, the `PizzaStore` is directly responsible for creating the pizza objects.

**File: `WithoutFactoryPattern/PizzaStore.java`**
```java
Pizza orderPizza(String pizzaType){
    Pizza pizza = null;

    // Problem Area: The store must know how to make every pizza.
    if (pizzaType.equals("cheese")){
        pizza = new CheesePizza();
    }
    else if (pizzaType.equals("greek")){
        pizza = new GreekPizza();
    }
    // ... more pizza types would require more `else if` blocks.

    pizza.prepare();
    //...
    return pizza;
}
```

### Issues with this approach:

1.  **Tight Coupling:** The `PizzaStore` is tightly coupled to every concrete pizza class (`CheesePizza`, `GreekPizza`). It must know about every single type of pizza that exists.
2.  **Violation of Design Principles:**
    *   **Single Responsibility Principle:** The `PizzaStore`'s job should be to manage the ordering process (prepare, bake, cut, pack), but here it's also responsible for knowing how to create every pizza.
    *   **Open/Closed Principle:** The class is not "closed for modification." If you want to add a new pizza type (e.g., `MushroomPizza`), you have to open up the `PizzaStore` class and modify its `orderPizza` method. In a large application, this is risky and hard to maintain.

---

## `WithSimpleFactoryPattern` - The Solution

This version introduces a new class, `SimplePizzaFactory`, to handle the creation of objects, solving the problems above.

### 1. The Factory (`WithSimpleFactoryPattern/SimplePizzaFactory.java`)

This class has one job: create pizzas. The `if-else` logic from the old `PizzaStore` is moved here.

```java
public class SimplePizzaFactory {
    public Pizza createPizza(String pizzaType){
        Pizza pizza = null;

        if (pizzaType.equals("cheese")){
            pizza = new CheesePizza();
        }
        else if (pizzaType.equals("pepperoni")){
            pizza = new PepporoniPizza();
        }
        //...
        return pizza;
    }
}
```

### 2. The Client (`WithSimpleFactoryPattern/PizzaStore.java`)

The `PizzaStore` is now much simpler and cleaner. It doesn't create pizzas anymore. Instead, it asks the factory to do it.

```java
public class PizzaStore {
    SimplePizzaFactory factory;

    public PizzaStore(SimplePizzaFactory factory){
        this.factory = factory;
    }

    public Pizza orderPizza(String pizzaType){
        Pizza pizza;

        // The PizzaStore is now DECOUPLED from the creation process.
        // It delegates the creation to the factory.
        pizza = factory.createPizza(pizzaType);

        pizza.prepare();
        pizza.bake();
        //...
        return pizza;
    }
}
```

---

## Summary of Differences & Benefits

| Aspect              | Without Factory Pattern                                       | With Simple Factory Pattern                                                  |
| :------------------ | :------------------------------------------------------------ | :--------------------------------------------------------------------------- |
| **Object Creation** | Done inside the client class (`PizzaStore`).                  | Centralized in a separate `SimplePizzaFactory` class.                        |
| **Coupling**        | Client is tightly coupled to all concrete product classes.    | Client is decoupled from concrete products; it only knows the factory.       |
| **Maintenance**     | To add a new pizza, you must change the `PizzaStore` code.    | To add a new pizza, you only change the `SimplePizzaFactory`. The `PizzaStore` remains untouched. |
| **Code Smell**      | Large `if-else` or `switch` statements for object creation often appear in client code. | Creation logic is encapsulated, making the client code cleaner and more focused on its primary role. |

**The primary benefit is decoupling the client from the concrete implementation of the objects it needs to create.** This makes the system more flexible and easier to maintain as it grows.

---

## Use in Real Enterprise Projects

You use a Simple Factory when you need to create different objects based on some condition, but you want to hide this complex creation logic from the client code.

*   **Database Connections:** A factory could return a `PostgresConnection`, `MySqlConnection`, or `OracleConnection` object based on a `dbType` string from a config file. Your application's data access layer doesn't need to know the specifics of each database; it just asks the factory for a connection.
*   **Payment Processing:** An e-commerce site might process payments through Stripe or PayPal. A `PaymentGatewayFactory` could create the correct processor object (`StripeProcessor`, `PayPalProcessor`) based on the user's selection. The order processing code simply calls `processor.charge()`, regardless of which gateway is being used.
*   **Report Generation:** An application might need to generate reports in different formats. A `ReportFactory` could create a `PdfGenerator`, `CsvGenerator`, or `ExcelGenerator` object, allowing the client to simply call `generator.createReport(data)`.
