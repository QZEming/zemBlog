---
title: typescript基础知识（三）类
abbrlink: '23853775'
date: 2019-03-19 22:37:38
tags: typescript
categories: typescript
---
# 类的定义
在typescript中类的使用和ES6中类的使用很类似，但是属性，方法中的参数和返回值需要明确类型
<!-- more -->
```typescript
class Person{
    constructor(name:string){
        this.name=name;
    }
    name:string;
    getName():string{
        return this.name;
    }
    setName(name:string):void{
        this.name=name;
    }
}
 
let person=new Person('小明');
 
console.log(person.getName()); // 小明
```
# 类的继承
在typescript中类的继承和ES6中的class继承是相似的，都通过extends关键字来实现继承
```typescript
class Person{
    name:String;
    constructor(name:String){
        this.name=name;
    }
    introduce():void{
        console.log(`I am ${this.name}`);
    }
}
 
class Student extends Person{}
 
var p=new Person('小明');
p.introduce(); // I am 小明
```
使用一个类继承另一个类时，如果没有写构造方法，会默认生成一个调用父类构造方法的构造方法，像上面的代码，其实Student类是这样的
```typescript
class Student extends Person{
    constructor(name:string){
        super(name);
    }
}
```
super是用于调用父类的，这点和ES6中是一样的。

# 类的修饰符
typescript中有修饰符来限定属性/方法的访问权限，修饰符分别为public，protected，private，跟Java的修饰符类似

## public
使用public做修饰符时，该属性/方法可以被类内部，子类，和类外部调用
```typescript
class Person{
    public name:String;
    constructor(name:String){
        this.name=name;
    }
    introduce():void{
        console.log(`I am ${this.name}`);
    }
}
 
class Student extends Person{
    constructor(name:string){
        super(name);
    }
    say():void{
        console.log(`${this.name} say hello`);
    }
}
 
var p=new Person('小明');
p.introduce(); // 类的内部调用
 
var s=new Student('小红'); 
s.say(); // 子类中调用
 
console.log(p.name);// 类的外部调用
```
## protected
使用protected做修饰符时，该属性/方法可以被类的内部，子类调用，不能被类外部调用
```typescript
class Person{
    protected name:String;
    constructor(name:String){
        this.name=name;
    }
    introduce():void{
        console.log(`I am ${this.name}`);
    }
}
 
class Student extends Person{
    constructor(name:string){
        super(name);
    }
    say():void{
        console.log(`${this.name} say hello`);
    }
}
 
var p=new Person('小明');
p.introduce(); // 类的内部调用
 
var s=new Student('小红'); 
s.say(); // 子类中调用
 
console.log(p.name);// 类的外部调用 报错！！！！！
```
## private
```typescript
class Person{
    private name:String;
    constructor(name:String){
        this.name=name;
    }
    introduce():void{
        console.log(`I am ${this.name}`);
    }
}
 
class Student extends Person{
    constructor(name:string){
        super(name);
    }
    say():void{
        console.log(`${this.name} say hello`); // 报错！！！！！！
    }
}
 
var p=new Person('小明');
p.introduce(); // 类的内部调用
 
var s=new Student('小红'); 
s.say(); // 子类中调用  报错！！！！！！
 
console.log(p.name);// 类的外部调用  报错！！！！！！
```
## 静态方法/属性
typescript中的静态方法和静态属性的使用和ES6中是一样的，都是通过在前面加static关键字来实现的，静态方法是通过类本身来调用的，而实例方法是通过实例对象来调用的
```typescript
class Person{
    name:string;
    constructor(name:string){
        this.name=name;
    }
    introduce():void{
        console.log(`I am ${this.name}`);
    }
    static sta():void{
        console.log('static');
    }
}
 
let p= new Person('小明');
p.introduce();
Person.sta();
```
要注意的是，静态方法不能调用非静态属性/方法，否则会报错
```typescript
class Person{
    name:string;
    static age:number=20;
    constructor(name:string){
        this.name=name;
    }
    introduce():void{
        console.log(`I am ${this.name}`);
    }
    //static getAge():void{
    //    console.log(`${this.name} is ${this.age} years old`); //会报错，因为调用了实例属性name
    //}
    static getAge():void{
        console.log(`I am ${this.age} years old`);
    }
}
 
let p= new Person('小明');
p.introduce();
Person.getAge();
```
## 抽象类/方法
抽象方法是用于定义类的名字类型而不写出方法体，在其所在类的子类中写出方法体的方法，而只有抽象类中能有抽象方法，抽象类中可以有一般方法，这和Java是一样的。
```typescript
abstract class Person{
    abstract work():void;
}
 
class Student extends Person{
    work():void{
        console.log('读书');
    }
}
 
class Teacher extends Person{
    work():void{
        console.log('教书');
    }
}
 
let s=new Student();
s.work();
 
let t=new Teacher();
t.work();
```
要注意的是，抽象类是不能构建实例的，即使有构造方法也不行，抽象类的子类必须实现抽象类中所有的抽象方法