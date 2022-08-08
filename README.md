# What is a SOLID programming?

SOLID is an abbreviation of five principles that make complex object-oriented projects easy to maintain and extend. These principles are a subset of even more principles formulated by Robert Martin (Design Principles and Design Patterns). The abbreviation may be deciphered as:
- S — Single Responsibility Principle (SRP)
- O — Open/Closed Principle (OCP)
- L — Liskov Substitution Principle (LSP)
- I — Interface Segregation Principle (ISP)
- D — Dependency Inversion Principle (DIP)

More on each of them below.

## Single Responsibility Principle (SRP)

Originally stated as:
> A class should have only one reason to change.[^1]
[^1]: Robert C. Martin. (2003). *Agile Software Development, Principles, Patterns, and Practices*

The "reason" here has the same meaning as an "actor". In other words *one class should be used by only one actor*.

This principle helps to prevent complications when using same function as a part of different functions required by different actors. Consider the following example:
```
class Employee {
  employeeData: EmployeeData;
  
  constructor(employeeData: EmployeeData) {
    this.employeeData = employeeData;
  }
  
  calculateWagePerWeek(): number {
    return this.workHoursPerWeek() * this.employeeData.hourlyWage;
  }
  
  calculateSnacksPerWeek(): number {
    return this.workHoursPerWeek() * 0.5;
  }
  
  workHoursPerWeek(): number {
    return this.employeeData.hoursWorkedLastMonth / 4.5;
  }
}
```
Both `calculateWagePerWeek()` and `calculateSnacksPerWeek()` use the same method of calculating work hours per week. But these functions are used by different actors! The actor for `calculateWagePerWeek()` is the accounting department and the actor for `calculateSnacksPerWeek()` is the purchase department. Suppose the accounting department decides to calculate overtime hours twice as regular hours. A developer should probably change the code of `workHoursPerWeek()` to account for overtime, e.g.:
```
workHoursPerWeek(): number {
  const hoursPerWeek = this.employeeData.hoursWorkedLastMonth / 4.5;
  if (hoursPerWeek > 40) {
    return 40 + (hoursPerWeek - 40)*2;
  }
  return hoursPerWeek;
}
```
But this change would break the calculation of snacks for purchases department, as the average number of snacks per hour does not change with overtime! The correct solution is to split the original class into two separate ones and hide them behind an `EmployeeFacade`. The employee data may be separated also as a data structure or included in the `EmployeeFacade`.
```
class EmployeeFacade {
  calculateWagePerWeek(employeeData: EmployeeData): number {
    const wageCalculator = new WageCalculator(employeeData: EmployeeData);
    return wageCalculator.calculateWagePerWeek();
  }
  
  calculateSnacksPerWeek(): number {
    const snacksCalculator = new SnacksCalculator(employeeData: EmployeeData);
    return snacksCalculator.calculateSnacksPerWeek();
  }
}

class WageCalculator {
  constructor(employeeData: EmployeeData) {
    this.employeeData = employeeData;
  }
  
  calculateWagePerWeek(): number {
    return this.workHoursPerWeek() * this.employeeData.hourlyWage;
  }
  
  workHoursPerWeek(): number {
    const hoursPerWeek = this.employeeData.hoursWorkedLastMonth / 4.5;
    if (hoursPerWeek > 40) {
      return 40 + (hoursPerWeek - 40)*2;
    }
    return hoursPerWeek;
  }
}

class SnacksCalculator {
  constructor(employeeData: EmployeeData) {
    this.employeeData = employeeData;
  }
  
  calculateSnacksPerWeek(): number {
    return this.workHoursPerWeek() * 0.5;
  }
  
  workHoursPerWeek(): number {
    return this.employeeData.hoursWorkedLastMonth / 4.5;
  }
}

const employeeData = new EmployeeData();
const employee = new EmployeeFacade();
const wagePerWeek = employee.calculateWagePerWeek(employeeData);
const snacksPerWeek = employee.calculateSnacksPerWeek(employeeData);
```

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


## Closures

Closure is a concept of keeping a local variable accessible even after a function execution IF one calls a child function of the executed function using this local variable. Closures have to be allowed in a language (JS, Ruby). If closures are not allowed (C, though there are ways to mimic them through pointers), the variable is blown away from memory and is not accessible from the child function. Usually it is function arguments that are subject to closure.

Example 1 (functional programming):
```
function makeTag(openTag, closeTag) {
  return function(content) {
    return openTag + content + closeTag;
  };
}

const h1 = makeTag('<h1>', '<\h1>');
const italic = makeTag('<i>', '<\i>');
console.log(h1(italic('Hello world!'))); // <h1><i>Hello world!<\i><h1>
```

Example 2 (callbacks):
```
function filterMales (employee) { return employee.gender === 'male'; };

function fetchSelectedEmployees(databaseUrl, filterFunction) {
  fetchEmployees(databaseUrl).then(employees => employees.filter(filterFunction));
}
```


## JS: [[Prototype]], Prototype, __proto__ and `new`
`[[Prototype]]` is a set of an object's actual properties and methods. `__proto__` is a combined getter/setter of `[[Prototype]]`. One should avoid changing `[[Prototype]]` through `__proto__` as this operation has very poor performance.

Any JS object inherits its `[[Prototype]]` from a constructor's function `Prototype`. `Prototype` is a set of properties and methods that is copied into `[[Prototype]]` of objects created by constructor functions. Only functions can have Prototypes.

When a function is invoked with the `new` keyword, 5 things happen:
1. An empty object (`{}`) is created.
2. Its `[[Prototype]]` is assigned a Prototype of the invoked function.
3. `this` property becomes bound to the object itself.
4. Constructor function is executed upon the object (using the freshly bound `this`).
5. The object is returned.

In other words, the action of `new` is equivalent to this function (assuming `New(A, arg1, arg2)` is the same as `new A(arg1, arg2)`:
```
function New(func) {
    var res = {};
    if (func.prototype !== null) {
        res.__proto__ = func.prototype;
    }
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    return res;
}
```
