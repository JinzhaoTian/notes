TypeScript 在 [JavaScript 原型链](../JavaScript/JavaScript%20原型链.md)基础上，添加了静态类型检查和完整的面向对象特性：
1. **类型检查**：编译时类型检查，减少运行时错误
2. **完整 OOP 特性**：类、接口、抽象类、访问修饰符
3. **强类型泛型**：类型安全的通用编程
4. **装饰器支持**：元编程和 AOP 能力
5. **更好的工程化**：模块化、命名空间、类型定义

```typescript
// JavaScript 的不足
function Person(name) {
    this.name = name;
}
// 运行时才能发现错误
const p = new Person(123); // 应该传入字符串，但不会报错

// TypeScript 的类型安全
class Person {
    name: string;
    
    constructor(name: string) {
        this.name = name; // 编译时类型检查
    }
}

const p = new Person(123); // 编译错误：Argument of type 'number' is not assignable to parameter of type 'string'
```

## 核心特性

### 类定义

```typescript
class Animal {
    // 1. 属性（字段）
    private id: number;          // 私有属性，仅类内可访问
    protected name: string;      // 受保护，类内和子类可访问
    public age: number = 0;      // 公共属性，默认修饰符
    readonly species: string;    // 只读属性
    
    // 2. 静态属性
    static totalAnimals: number = 0;
    
    // 3. 构造函数
    constructor(name: string, species: string) {
        this.name = name;
        this.species = species;
        this.id = Math.random();
        Animal.totalAnimals++;
    }
    
    // 4. 实例方法
    move(distance: number = 0): void {
        console.log(`${this.name} moved ${distance}m.`);
    }
    
    // 5. Getter/Setter
    get description(): string {
        return `${this.name} is a ${this.species}`;
    }
    
    set newName(value: string) {
        if (value.length < 3) {
            throw new Error('Name too short');
        }
        this.name = value;
    }
    
    // 6. 静态方法
    static createAnimal(name: string): Animal {
        return new Animal(name, 'unknown');
    }
    
    // 7. 抽象方法（在抽象类中）
    abstract makeSound(): void; // 必须在子类实现
}

// 使用
const cat = new Animal('Kitty', 'Cat');
cat.move(10);
console.log(cat.description);
Animal.totalAnimals; // 访问静态属性
```

### 访问修饰符

```typescript
class BankAccount {
    private _balance: number = 0;      // 真正的私有（编译后为普通属性）
    #secretCode: string = '123';       // ES2022 私有字段（编译后仍私有）
    protected accountNumber: string;
    public owner: string;
    
    constructor(owner: string, accountNumber: string) {
        this.owner = owner;
        this.accountNumber = accountNumber;
    }
    
    // 通过公开方法访问私有属性
    public deposit(amount: number): void {
        if (amount > 0) {
            this._balance += amount;
        }
    }
    
    public getBalance(): number {
        return this._balance;
    }
    
    // 参数属性简写
    constructor(
        public id: string,
        private _pin: number
    ) {}
}

class SavingsAccount extends BankAccount {
    private interestRate: number = 0.03;
    
    calculateInterest(): number {
        // 可以访问 protected 属性
        console.log(this.accountNumber);
        // 不能访问私有属性
        // console.log(this._balance); // 错误
        return this.getBalance() * this.interestRate;
    }
}
```

### 继承

```typescript
// 基类（父类）
class Vehicle {
    constructor(
        protected brand: string,
        protected model: string,
        protected year: number
    ) {}
    
    start(): void {
        console.log(`${this.brand} ${this.model} started`);
    }
    
    stop(): void {
        console.log(`${this.brand} ${this.model} stopped`);
    }
    
    // 虚方法，可被子类重写
    getDetails(): string {
        return `${this.year} ${this.brand} ${this.model}`;
    }
}

// 派生类（子类）
class Car extends Vehicle {
    private doors: number;
    
    constructor(brand: string, model: string, year: number, doors: number) {
        super(brand, model, year); // 必须首先调用 super()
        this.doors = doors;
    }
    
    // 方法重写
    getDetails(): string {
        // 调用父类方法
        const baseDetails = super.getDetails();
        return `${baseDetails} with ${this.doors} doors`;
    }
    
    // 子类特有方法
    honk(): void {
        console.log('Beep beep!');
    }
}

class Motorcycle extends Vehicle {
    private hasSidecar: boolean;
    
    constructor(brand: string, model: string, year: number, hasSidecar: boolean) {
        super(brand, model, year);
        this.hasSidecar = hasSidecar;
    }
    
    // 重写父类方法
    getDetails(): string {
        const sidecarInfo = this.hasSidecar ? 'with sidecar' : 'without sidecar';
        return `${super.getDetails()} ${sidecarInfo}`;
    }
}
```

### 多态

```typescript
class Shape {
    constructor(public color: string) {}
    
    // 抽象方法模式
    getArea(): number {
        throw new Error('Method not implemented');
    }
    
    draw(): void {
        console.log(`Drawing a ${this.color} shape`);
    }
}

class Circle extends Shape {
    constructor(color: string, public radius: number) {
        super(color);
    }
    
    getArea(): number {
        return Math.PI * this.radius ** 2;
    }
    
    draw(): void {
        console.log(`Drawing a ${this.color} circle with radius ${this.radius}`);
    }
}

class Rectangle extends Shape {
    constructor(color: string, public width: number, public height: number) {
        super(color);
    }
    
    getArea(): number {
        return this.width * this.height;
    }
    
    draw(): void {
        console.log(`Drawing a ${this.color} rectangle ${this.width}x${this.height}`);
    }
}

// 多态使用
const shapes: Shape[] = [
    new Circle('red', 5),
    new Rectangle('blue', 4, 6),
    new Circle('green', 3)
];

shapes.forEach(shape => {
    console.log(`Area: ${shape.getArea()}`); // 多态调用
    shape.draw(); // 多态调用
});
```

## 抽象类

```typescript
// 抽象类：不能实例化，只能被继承
abstract class Database {
    constructor(protected connectionString: string) {}
    
    // 抽象方法：必须由子类实现
    abstract connect(): Promise<void>;
    abstract query(sql: string): Promise<any>;
    abstract disconnect(): Promise<void>;
    
    // 具体方法
    protected log(message: string): void {
        console.log(`[Database]: ${message}`);
    }
    
    // 模板方法模式
    async execute(sql: string): Promise<any> {
        await this.connect();
        this.log(`Executing: ${sql}`);
        const result = await this.query(sql);
        await this.disconnect();
        return result;
    }
}

// 具体实现
class MySQLDatabase extends Database {
    async connect(): Promise<void> {
        console.log(`Connecting to MySQL: ${this.connectionString}`);
        // 实际连接逻辑
    }
    
    async query(sql: string): Promise<any> {
        console.log(`MySQL query: ${sql}`);
        return { rows: [] };
    }
    
    async disconnect(): Promise<void> {
        console.log('Disconnecting from MySQL');
    }
}

class MongoDBDatabase extends Database {
    async connect(): Promise<void> {
        console.log(`Connecting to MongoDB: ${this.connectionString}`);
    }
    
    async query(query: string): Promise<any> {
        console.log(`MongoDB query: ${query}`);
        return { documents: [] };
    }
    
    async disconnect(): Promise<void> {
        console.log('Disconnecting from MongoDB');
    }
}
```

## 接口

```typescript
// 接口：定义契约，不包含实现
interface Logger {
    log(message: string): void;
    error(message: string, error?: Error): void;
    warn(message: string): void;
    info(message: string): void;
}

interface Configurable {
    config: object;
    configure(config: object): void;
}

// 类实现接口
class ConsoleLogger implements Logger {
    log(message: string): void {
        console.log(`[LOG] ${message}`);
    }
    
    error(message: string, error?: Error): void {
        console.error(`[ERROR] ${message}`, error);
    }
    
    warn(message: string): void {
        console.warn(`[WARN] ${message}`);
    }
    
    info(message: string): void {
        console.info(`[INFO] ${message}`);
    }
}

class FileLogger implements Logger, Configurable {
    constructor(public config: object) {}
    
    configure(config: object): void {
        this.config = config;
    }
    
    log(message: string): void {
        // 写入文件的逻辑
        console.log(`[FILE LOG] ${message}`);
    }
    
    error(message: string): void {
        console.error(`[FILE ERROR] ${message}`);
    }
    
    warn(message: string): void {
        console.warn(`[FILE WARN] ${message}`);
    }
    
    info(message: string): void {
        console.info(`[FILE INFO] ${message}`);
    }
}

// 接口继承
interface DatabaseLogger extends Logger {
    tableName: string;
    logToDatabase(message: string): Promise<void>;
}

// 可选属性和只读属性
interface User {
    readonly id: number;          // 只读
    name: string;
    age?: number;                 // 可选
    email: string;
    [key: string]: any;          // 索引签名
}
```

## 泛型

1. **泛型类**
```typescript
// 泛型容器类
class Container<T> {
    private items: T[] = [];
    
    add(item: T): void {
        this.items.push(item);
    }
    
    remove(item: T): boolean {
        const index = this.items.indexOf(item);
        if (index > -1) {
            this.items.splice(index, 1);
            return true;
        }
        return false;
    }
    
    getItem(index: number): T | undefined {
        return this.items[index];
    }
    
    getAll(): T[] {
        return [...this.items]; // 返回副本
    }
    
    // 泛型方法
    filter<P extends keyof T>(property: P, value: T[P]): T[] {
        return this.items.filter(item => item[property] === value);
    }
}

// 使用
const numberContainer = new Container<number>();
numberContainer.add(1);
numberContainer.add(2);

const userContainer = new Container<{id: number, name: string}>();
userContainer.add({id: 1, name: 'John'});
const john = userContainer.getItem(0);
```

2. **泛型约束**
```typescript
interface Identifiable {
    id: number | string;
}

// 泛型约束
class Repository<T extends Identifiable> {
    private data = new Map<T['id'], T>();
    
    add(item: T): void {
        this.data.set(item.id, item);
    }
    
    get(id: T['id']): T | undefined {
        return this.data.get(id);
    }
    
    update(id: T['id'], updates: Partial<T>): boolean {
        const item = this.data.get(id);
        if (item) {
            Object.assign(item, updates);
            return true;
        }
        return false;
    }
    
    delete(id: T['id']): boolean {
        return this.data.delete(id);
    }
}

// 使用
interface Product extends Identifiable {
    name: string;
    price: number;
    category: string;
}

const productRepo = new Repository<Product>();
productRepo.add({id: 1, name: 'Laptop', price: 999, category: 'Electronics'});
const laptop = productRepo.get(1);
```

## 装饰器（Decorators）

```typescript
// 类装饰器
function Injectable(constructor: Function) {
    console.log(`${constructor.name} is now injectable`);
}

function LogClass(target: Function) {
    console.log(`Class ${target.name} was defined at ${new Date()}`);
}

// 方法装饰器
function LogMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with args:`, args);
        const result = originalMethod.apply(this, args);
        console.log(`${propertyKey} returned:`, result);
        return result;
    };
    
    return descriptor;
}

// 属性装饰器
function DefaultValue(value: any) {
    return function(target: any, propertyKey: string) {
        target[propertyKey] = value;
    };
}

// 使用装饰器
@Injectable
@LogClass
class UserService {
    @DefaultValue('default user')
    private userName: string;
    
    constructor() {
        console.log('UserService created');
    }
    
    @LogMethod
    getUser(id: number): string {
        return `User ${id}`;
    }
}
```

## 编译原理和运行时机制

1. **TypeScript 编译过程**
```typescript
// TypeScript 源码
class Point {
    x: number;
    y: number;
    
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    
    distance(p: Point): number {
        const dx = this.x - p.x;
        const dy = this.y - p.y;
        return Math.sqrt(dx * dx + dy * dy);
    }
}

// 编译为 JavaScript（ES5）
var Point = /** @class */ (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.distance = function (p) {
        var dx = this.x - p.x;
        var dy = this.y - p.y;
        return Math.sqrt(dx * dx + dy * dy);
    };
    return Point;
}());
```


2. **类型擦除和运行时**
```typescript
// TypeScript 编译时类型检查
interface Person {
    name: string;
    age: number;
}

function greet(person: Person): string {
    return `Hello, ${person.name}!`;
}

// 编译后（类型信息被擦除）
function greet(person) {
    return "Hello, " + person.name + "!";
}

// 运行时没有任何类型信息
greet({ name: 123 }); // 编译时报错，但运行时不会报错
```