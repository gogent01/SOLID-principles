# What is a SOLID programming?

SOLID is an abbreviation of five principles that make complex object-oriented projects easy to maintain and extend. These principles are a subset of even more principles formulated by Robert Martin (Design Principles and Design Patterns). The abbreviation may be deciphered as:
- S — Single Responsibility Principle (SRP)
- O — Open/Closed Principle (OCP)
- L — Liskov Substitution Principle (LSP)
- I — Interface Segregation Principle (ISP)
- D — Dependency Inversion Principle (DIP)

More on each of them below.

## Single Responsibility Principle (SRP)



## Open/Closed Principle (OCP)



## Liskov Substitution Principle (LSP)



## Interface Segregation Principle (ISP)



## Dependency Inversion Principle (DIP)



# Other stuff

## Dependency injection (DI)

Dependency injection simply means passing (complex) instance variables (IV) to a class constructor or to a setter method. The idea brings three benefits:
1. It allows to change various interface implementations of an IV without having to modify a class code.
2. It allows to interchange IV implementations after a creation of a class instance (if setter methods are used).
3. It allows to easily mock IV data during testing — by simply passing the desired IV implementation, again, without any changes to the class.
- Additionally, DI follows OCP (the class becomes open to modifications with different IV implementations) and promotes LSP (by requiring a common interface for all possible IV classes).

Below is a simple example of dependency injection. Note that without DI changing tires or engine in car whould require creating a totally new car!
```
class Car {
  engine: Engine;
  tires: Tires;
  
  constructor(engine: Engine, tires: Tires) { // <- Dependency Injection!
    this.engine = engine;
    this.tires = tires;
  }
  
  replaceEngine(engine: Engine): void { // <- Dependency Injection!
    this.engine = engine; 
  }
  
  replaceTires(tires: Tires): void { // <- Dependency Injection!
    this.tires = tires; 
  }
}

const summerTires = new Tires('summer');
const v6Engine = new Engine('v6');
const ford = new Car(v6Engine, summerTires);

const winterTires = new Tires('winter');
ford.replaceTires(winterTires);
```
