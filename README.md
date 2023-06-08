# What is SOLID programming?

SOLID is an abbreviation of five principles that make complex object-oriented projects easier to maintain and extend. These principles are a subset of even more principles formulated by Robert Martin (Design Principles and Design Patterns). The abbreviation may be deciphered as:
- S — Single Responsibility Principle (SRP)
- O — Open/Closed Principle (OCP)
- L — Liskov Substitution Principle (LSP)
- I — Interface Segregation Principle (ISP)
- D — Dependency Inversion Principle (DIP)

More on each of them below.

## Single Responsibility Principle (SRP)

Originally stated as:
> A class should have only one reason to change.[^1]
[^1]: Robert C. Martin. (2003). *Agile Software Development, Principles, Patterns, and Practices.*

The "reason" here has the same meaning as an "actor". In other words *one class should be used by only one actor*.

This principle separates zones of responsibility for different actors, so no change would happen without any actor's notice. Consider a following example:
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
Both `calculateWagePerWeek()` and `calculateSnacksPerWeek()` use the same method of calculating work hours per week. But these functions are used by different actors! The actor for `calculateWagePerWeek()` is the accounting department and the actor for `calculateSnacksPerWeek()` is the purchase department. Let's suppose that the accounting department decides to calculate overtime hours not as regular ones, but twice as high. A developer should  change the code of `workHoursPerWeek()` to account for overtime, e.g.:
```
workHoursPerWeek(): number {
  const WEEKS_IN_MONTH = 4.5;
  const hoursPerWeek = this.employeeData.hoursWorkedLastMonth / WEEKS_IN_MONTH;
  if (hoursPerWeek > 40) {
    return 40 + (hoursPerWeek - 40) * 2;
  }
  return hoursPerWeek;
}
```
But this change is going to break the calculation of snacks for purchases department, as the average number of snacks per hour does not change with overtime! The correct solution is to split the original class into two separate ones and hide them behind an `EmployeeFacade`. The employee data may also be separated as a data structure or included in the `EmployeeFacade`.
```
class EmployeeFacade {
  employeeData: EmployeeData;
  
  constructor(employeeData: EmployeeData) {
    this.employeeData = employeeData;
  }

  calculateWagePerWeek(): number {
    const wageCalculator = new WageCalculator(this.employeeData);
    return wageCalculator.calculateWagePerWeek();
  }
  
  calculateSnacksPerWeek(): number {
    const snacksCalculator = new SnacksCalculator(this.employeeData);
    return snacksCalculator.calculateSnacksPerWeek();
  }
}

class WageCalculator {
  employeeData: EmployeeData;
  
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
  employeeData: EmployeeData;
  
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
const employee = new EmployeeFacade(employeeData);
const wagePerWeek = employee.calculateWagePerWeek();
const snacksPerWeek = employee.calculateSnacksPerWeek();
```

## Open/Closed Principle (OCP)

Originally stated as:
> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.[^2]
[^2]: Bertrand Mayer. (1988). *Object-Oriented Software Construction.*

A module is said to be open when one can add additional component extension (through inhertiance and polymorphism). A module is said to be closed when it is available for use by other modules (thus one cannot change this module's implementation to prevent unnoticed changes for other actors). To achive OCP one should separate logical levels of an application. Higher levels should be protected from changes in lower levels, which is done by making lower levels depend on the higher ones. Dependency inversion (through interfaces) may be required to abstract the details of lower levels realization from business logic. Lower levels may also be protected from the higher ones by using interfaces. This is helpful when a lower level component does not use a higher level component directly.

An example of OCP is below. Here one can add any more phrases and more formatters by implementing corresponding interfaces. During this addition earlier implementations do not require any changes.
```
interface Phrase {
  text: string;
  accept: (visitor: Visitor) => void;
}

class Greeting implements Phrase {
  text: string = 'Good morning!';
  
  accept(visitor: Visitor): void {
    visitor.visitGreeting(this);
  }
}

class Goodbye implements Phrase {
  text: string = 'See you later!';
  
  accept(visitor: Visitor): void {
    visitor.visitGoodbye(this);
  }
}

interface FormattingVisitor {
  visitGreeting: (greeting: Greeting) => string;
  visitGoodbye: (goodbye: Goodbye) => string;
}

class HTMLFormatter implements FormattingVisitor {
  visitGreeting(greeting: Greeting): string {
    return `<p style="font-style: italic;">${greeting.text}</p>`;
  }
  
  visitGoodbye(goodbye: Goodbye): string {
    return `<p style="font-weight: bold;">${goodbye.text}</p>`;
  }
}

class JSONFormatter implements FormattingVisitor {
  visitGreeting(greeting: Greeting): string {
    return `{ "text": "${greeting.text}", "style": "italic" }`;
  }
  
  visitGoodbye(goodbye: Goodbye): string {
    return `{ "text": "${goodbye.text}", "style": "bold" }`;
  }
}

const greeting = new Greeting();
const goodbye = new Goodbye();
const htmlFormatter = new HTMLFormatter();
const jsonFormatter = new JSONFormatter();

greeting.accept(htmlFormatter);
goodbye.accept(jsonFormatter);
```

## Liskov Substitution Principle (LSP)

Originally stated as:
> Subtype Requirement: Let ϕ(x) be a property provable about objects x of type T. Then ϕ(y) should be true for objects y of type S where S is a subtype of T.[^3]
[^3]: Barbara Liskov, Jeanette Wing. (1994). *A behavioral notion of subtyping.*

More clearly:
> Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

In other words, all subclasses should use the same interface in order to be able to interchange different realizations of a component. But not only that  any function arguments and returns should always be the same for any uses of a class family.

Example:
```
interface Communication {
  greet: () => string;
  introduce: () => string;
  goodbye: () => string;
}

class EnglishSpeech implements Communication {
  greet(): string {
    return '👨 Hello!';
  }
  
  introduce(): string {
    return 'My name is Georgy Mishurovsky. I'm a web developer 👨‍💻 (also an MD 👨‍⚕️!). My tech stack includes JS, TS, Vue and NodeJS. I can also perform data analysis in R and Python.';
  }
  
  goodbye(): string {
    return 'See you soon, bye! 👋';
  }
}

class RobotSpeech implements Communication {
  greet(): string {
    return '🤖 H3ll0!';
  }
  
  introduce(): string {
    return 'My n4m3 12 630r6y M12hur0v2ky. 1'm 4 w38 d3v3l093r 👨‍💻 (4l20 4n MD 👨‍⚕️!). My 73ch 274ck 1nclud32 j2, 72, vu3 4nd n0d3j2. 1 c4n 4l20 93rf0rm d474 4n4ly212 1n r 4nd 9y7h0n.';
  }
  
  goodbye(): string {
    return '233 y0u 200n, 8y3! 👋'
  }
}

class Polyglot {
  private _speechSource?: Communication;
  
  constructor(speechSource: Communication) {
    this.speechSource = speechSource;
  }
  
  set speechSource(speechSource: Communication) {
    this._speechSource = speechSource;
  }
  
  speak(): string {
    if (!this.speechSource) return '';
    return [this.speechSource.greet(), this.speechSource.introduce(), this.speechSource.goodbye()].join(' ');
  }
}

const polyglot = new Polyglot();
polyglot.speak(); // ''

polyglot.speechSource = new EnglishSpeech();
polyglot.speak(); // '👨 Hello! My name is Georgy Mishurovsky. I'm a web developer 👨‍💻 (also an MD 👨‍⚕️!). My tech stack includes JS, TS, Vue and NodeJS. I can also perform data analysis in R and Python. See you soon, bye! 👋'

polyglot.speechSource = new RobotSpeech();
polyglot.speak(); // '🤖 H3ll0! My n4m3 12 630r6y M12hur0v2ky. 1'm 4 w38 d3v3l093r 👨‍💻 (4l20 4n MD 👨‍⚕️!). My 73ch 274ck 1nclud32 j2, 72, vu3 4nd n0d3j2. 1 c4n 4l20 93rf0rm d474 4n4ly212 1n r 4nd 9y7h0n. 233 y0u 200n, 8y3! 👋'
```


## Interface Segregation Principle (ISP)

Originally stated as:
> No client should be forced to depend on methods it does not use.[^4]
[^4]: Robert Martin

Stated in other words, one should not extend existing interfaces with new methods. Instead, one should create new interface and to implement both of them in new classes, if required. Thus the classes using only the implementation of original interface would not depend on methods they do not use.

*Let no components depend on methods they do not use.*

An example violating ISP:
```
interface Printer {
  abstract print(document: Document): void;
}


class CanonPrinter implements Printer {
  print(document: Document): void { 
    // ... 
  }
}


interface MultiFunctionPrinter {
  abstract print(document: Document): void;
  abstract copy(): void;
  abstract scan(): Document;
}


class CanonMFU implements MultiFunctionPrinter {
  print(document: Document): void { 
    // ... 
  }
  
  copy(): void { 
    // ... 
  }
  
  scan(): Document { 
    // ... 
  };
}

class CanonPrinterAndCopyingMachine implements MultiFunctionPrinter {
  print(document: Document): void { 
    // ... 
  }
  
  copy(): void { 
    // ... 
  }
}
```

Instead, one should have created three interfaces for printing, copying and scanning functionalities and implement only required interfaces:
```
interface Printer {
  abstract print(document: Document): void;
}

interface CopyingMachine {
  abstract copy(): void;
}

interface Scanner {
  abstract scan(): Document;
}


class CanonPrinter implements Printer {
  print(document: Document): void { 
    // ... 
  }
}

class CanonMFU implements Printer, CopyingMachine, Scanner {
  print(document: Document): void { 
    // ... 
  }
  
  copy(): void { 
    // ... 
  }
  
  scan(): Document { 
    // ... 
  }
}

class CanonPrinterAndCopyingMachine implements Printer, CopyingMachine {
  print(document: Document): void { 
    // ... 
  }
  
  copy(): void { 
    // ... 
  }
}
```

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
