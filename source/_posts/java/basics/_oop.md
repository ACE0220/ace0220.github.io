---
title: java基础-面向对象（快速了解|复习版）
date: 2025-3-12 18:02:30
categories:
  - java
  - basics
---

java面相对象快速过一遍法

<!-- more -->

# 方法

一个类可以有多个字段，people类具备了name,age,sex字段

```java
class People{
  public String name;
  public int age;
  pubilc int sex; 
}
```

为了保护封装性，将属性改成私有属性，通过person.name访问name字段的时候会编译报错，所以对外提供方法进行读写

```java
class People{
  private String name;
  private int age;
  private int sex;

  public String getName() {
    return this.name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public int getAge(){
    return this.age
  }

  public void setAge() {
    this.age = age;
  }

  # sex属性也是同理
  ...
}
```

使用方法

```java
Person alice = new Person();
System.out.println(alice.age); //编译错误
System.out.println(alice.getAge()) // 我们还没设置alice的年龄，所以输出0，int的默认值是0
```

## 方法的结构

```
修饰符 返回类型 方法名 (参数列表) {
  方法语句
  return 返回值
}
```

```java
class People{
  private String name;
  private int age;
  private int sex;


  public  String    getName() {
    return this.name;
  }
// 修饰符 返回类型    方法名   参数列表
  public boolean       setName(String name) {
    // 方法语句
    this.name = name;
    // 返回值
    return true;
  }

  ...........
}
```

### 方法结构各部分内容

#### 修饰符

  用于帮助开发控制访问权限，定义常量，多线程同步功能

  访问修饰符：
    - public 对所有类可见
    - private 同一个类内可见
    - protected 同一个包的类和所有子类可见
    - default 默认，在同一个包内可见，default即不写

  非访问修饰符：
    - static
    - final
    - abstract
    - synchronized
    - volatile
    - transient

#### 返回类型

方法有返回值则需要在方法标明，例如方法体最后一句return 10，那么返回类型要写int。
没有返回值则可以写void，并且不需要写return
（返回类型与数据类型的知识息息相关）

```java

class People{
  private String name;
  private int age;
  private int sex;


  public  String    getName() {
    return this.name;
  }
  // 返回boolean类型的值
  public boolean setName(String name) {
    this.name = name;
    return true;
  }

  // 没有返回值
  public void setName_1(String name) {
    this.name = name;
  }

  ...........
}
```

## this

在方法内部有一个隐含的变量this指向当前的实例，属性name为private，类的内部使用通过this.name进行访问，外部访问编译报错

```java
class People{
  private String name;
  private int age;
  private int sex;

  // 
  public String getName() {
    return this.name;
  }

  ...........
}

public class Main {
    public static void main(String[] args) {
        Perple alice = new People()
        // getName使用了public修饰符，所以实例可以访问这个方法，再通过方法内部访问this.name
        alice.getName(); // 嘻嘻
        alice.name // 不嘻嘻，编译错误
    }
}
```

# 构造方法

在复习方法的时候，我们没有设置alice的age，调用getAge的时候会返回int的默认值0。但是我们不能实例化之后，全部调用一次set方法去设置所有的属性，这时候构造方法就派上用场了。


```java
class Person{

  private String name;
  private int age;
  private int sex;

  // 构造方法，一次性传入name,age,sex
  public Person(String name, int age, int sex) {
    this.name = name;
    // 其实年龄也要加判断，总不能-1岁或200岁吧, 以sex判断作为参考就行
    this.age = age;
    // sex要加判断，sex就男女，不是1就是2
    if (sex !== 1 && sex !== 2) {
      throw new IllegalArgumentException("Invalid sex value, it should be 1 or 2")
    }
  }
}

public class Main{
  public static void main(String[] args) {
    People alice = new People("alice", 10, 2)
  }
}

```

## 多种构造方法

- 构造方法没有在类内编写，编译器会帮我们生成一个默认的构造方法
- 当我们编写了构造方法，默认的构造方法会失效
- 构造方法可以有不同的参数列表

```java

class Person{

  private String name;
  // 5.设定一个默认age是18岁
  private int age = 18;
  private int sex;

  // 1.默认构造方法，if 我们没写任何构造方法 { 编译器帮我们生成 }
  // public Person() {}

  // 2. 构造方法，一次性传入name,age,sex。 if 这里写了构造方法 { 上面那么默认生成的构造方法失效 }
  public Person(String name, int age, int sex) {
    this.name = name;
    this.age = age;
    this.sex = sex;
  }

  // 3. 那我确实想要一个只有默认属的实例，只能显式写构造方法,等我实例化之后再调用set方法写入属性值
  public Person() {}

  // 4.还有高手，我只想实例化的时候填入名字和性别就行，年龄默认18岁
  public Person(String name, int sex) {
    this.name = name;
    this.sex = sex;
  }
}

public class Main{
  public static void main(String[] args) {
    // new Person没传任何参数，调用的是构造方法3
    Person alice = new Person();
    alice.setAge(18)
    alice.setSex(2)
    alice.setName("alice")


    // 传了全部参数，调用的是构造方法2
    Person ace = new Person("ace", 10, 1);

    // 只传name和sex，调用的是构造方法4，在5的位置我们提前设置了age的默认值
    Person lilei = new Person("lilei", 1);
  }
}

```

# 方法重载overload

一系列同名方法，只需要参数不同，就可以一并存在于同一个类或子类，子类的后面再说

```java
class Say{
  public void what() {
    System.out.println("say: nothing");
  }
  public void what(String content) {
    System.out.println("say: " + content);
  }
  public void what(String name, String content) {
    System.out.println("say to " + name + ":" + content);
  }
  public void what(int content) {
    System.out.println("say: " + content);
  }
}

public class Main {
  public static void main(String[] args) {
    Say someoneSay = new Say();
    someoneSay.what();
    someoneSay.what("wow");
    someoneSay.what("ace", "wow");
    someoneSay.what(10);
    //someoneSay.what(10, "wow"); // 错误，是因为我没有重载，第一个参数是int，第二个参数是string的what方法
  }
}

```

# 继承

假定一个对哈希操作的类，我希望继承基础能力，再扩展自己的能力。

```java

class HashHandler{
  protected HashMap<String, String> hashmap =new HashMap<String, String>();
  // 基础能力
  public void put(String key, String value) {
    hashmap.put(key, value)
  }
  // 下面还有get，remove，replace等方法，省略
  ...
}

// 继承了HashHandler的能力，再增加新方法属于独有的方法
class NewHashHandler extends  HashHandler{
  // 增加了新的方法，传入的key必须是等于testKey,才能put进hash表
  public void newPut(String key, String value) {
    if (key == "testKey") {
      hashmap.put(key, value)
    }
  }
}
```

## 子类无法访问父类的属性问题

```java
// 狗，两只眼睛
class Dog{
  private int eyesNumber = 2;
}

// 泰迪继承了狗类
class ToyPoodle extends Dog{
  public int getEyesNumber() {
    return this.eyesNumber
  }
}

public class Main {
  public static void main(String[] args) {
    ToyPoodle toyPoodle = new ToyPoodle()
    System.out.println(toyPoodle.getEyesNumber())  // eyesNumber是父类属性，但使用了private，无法读取字段
  }
}
```

回想修饰符，private 同一个类内可见，protedcted 同一个包或子类可见，所以private要修改成protedcted

```java
// 狗，两只眼睛
class Dog{
  // private int eyesNumber = 2;
  protected int eyesNumber = 2; // private 改 protected
}
```

## super

在java中，任何class的构造方法调用前，都会先调用父类的构造方法，那么问题来了，Student构造前，编译器帮我们先调用父类的构造方法，但是父类的默认构造方法又因为我们自己写了父类的构造方法而失效了，这时候就会报错

Main.java:16: error: constructor Person in class Person cannot be applied to given types;

```java
public class Main {
  public static void main(String[] args) {
    Student stu = new Strdeng("ace", 19)
  }
}

class Person {
  protected String name;
  public Person(String name) {
    this.name = name
  }  
}

class Student extends Person {
  private int age;
  public Student(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
```

两种解决方案

```java
class Person {
  protected String name;
  public Person() {} // 写回一个默认构造方法
  public Person(String name) {
    this.name = name;
  }  
}
```

```java
class Student extends Person {
  private int age;
  public Student(String name, int age) {
    super(name); // 通过super调用父类的构造方法
    this.age = age;
  }
}
```

到这里明白了，父类没有构造方法或写了默认构造方法，可以不用super，
反之，父类写了构造方法又不保留默认构造方法，就要使用super显式调用父类构造方法


# 多态

子类继承父类，并且定义了一个与父类方法签名完全相同的方法,并且返回值也是一致，称为override覆写

overload则是方法签名不同，或者返回值不同

```java

public class Main{
  public static void main(String[] args) {
    People p1 = new People();
    p1.say();
    Son son = new Son();
  }
}

class People {
  public void say() {
    System.out.println("People.say");
  }
}

class Son extends People {
  // override 覆写父类的方法
  @Override
  public void say() {
    System.out.println("Son.say");
  }
}
```

## 多态

某个类型的方法调用，在运行期间动态决定调用子类方法。执行的时候，可能执行的是父类的方法，或者子类覆写的方法。

举个例子，people.drink,喝东西，那儿子可以喝水，孙子可能要喝饮料，但是喝这个行为是存在的

```java

public class main {
  public static void main(String[] args) {
    People p = new People();
    Son son = new Son();
    GrandSon gson = new GrandSon();
    drink(p);
    drink(son);
    drink(gson);
  }

  // 在运行之前，我也不知道传的是People，Son，还是GrandSon，但是他们都override了drink方法，我只要负责调用drink就好
  public static void drink(People p) {
    p.drink();
  }
}


// 人类只有喝的方法（行为）
class People {
  public void drink() {
    System.out.println("just drink");
  }
}

class Son extends People {
  @Override
  public void drink() {
    System.out.println("drink water");
  }
}

class GrandSon extends People {
  @Override
  public void drink() {
    System.out.println("drink coke");
  }
}
```


