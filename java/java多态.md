# Java多态

## 定义

* 多态指的是一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定
* 从宏观表现上面看，多态也是指允许不同类的对象对同一消息做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用）
* 多态可以称之为动态绑定，后期绑定和运行时绑定（来自于《java编程思想（第四版）》）

## 作用

通过分离做什么和怎么做，从另一个角度讲接口和实现分离开。改善了代码的组织结构和可读性，还能创建可扩展的程序。

## 概念说明

### 重写（运行时绑定）

* 定义：子类对父类的函数进行重新定义。

* 条件：如果在子类中定义某方法与其父类有相同的名称和参数，我们说该方法被重写 \(Overriding\)。

注：子类函数的访问修饰权限不能少于父类的，子类函数的抛出的异常范围要比父类重写函数的异常范围要小，返回值类型相同，否则编译不通过

### 重载（编译时绑定）

* 定义：方法重载是让类以统一的方式处理不同类型数据的一种手段。
* 条件：java同一个类中，可以有多个同名的方法，参数不同，返回值和访问权限没有要求。

注：重载不属于多态，因为它是编译时多态

## 实现条件（java）

* 继承：在多态中必须存在有继承关系的子类和父类。

* 重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。

* 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。

## 实现形式

在Java中有两种形式可以实现多态。继承和接口。

### 继承

基于继承的实现机制主要表现在父类和继承该父类的一个或多个子类对某些方法的重写，多个子类对同一方法的重写可以表现出不同的行为。

```java
public class Wine {
    private String name;
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Wine(){
    }
    
    public String drink(){
        return "喝的是 " + getName();
    }
    
    /**
     * 重写toString()
     */
    public String toString(){
        return null;
    }
}

public class JNC extends Wine{
    public JNC(){
        setName("JNC");
    }
    
    /**
     * 重写父类方法，实现多态
     */
    public String drink(){
        return "喝的是 " + getName();
    }
    
    /**
     * 重写toString()
     */
    public String toString(){
        return "Wine : " + getName();
    }
}

public class JGJ extends Wine{
    public JGJ(){
        setName("JGJ");
    }
    
    /**
     * 重写父类方法，实现多态
     */
    public String drink(){
        return "喝的是 " + getName();
    }
    
    /**
     * 重写toString()
     */
    public String toString(){
        return "Wine : " + getName();
    }
}

public class Test {
    public static void main(String[] args) {
        //定义父类数组
        Wine[] wines = new Wine[2];
        //定义两个子类
        JNC jnc = new JNC();
        JGJ jgj = new JGJ();
        
        //父类引用子类对象
        wines[0] = jnc;
        wines[1] = jgj;
        
        for(int i = 0 ; i < 2 ; i++){
            System.out.println(wines[i].toString() + "--" + wines[i].drink());
        }
        System.out.println("-------------------------------");

    }
}
OUTPUT:
Wine : JNC--喝的是 JNC
Wine : JGJ--喝的是 JGJ
-------------------------------
```

### 接口

在接口的多态中，指向接口的引用必须是指定这实现了该接口的一个类的实例程序，在运行时，根据对象引用的实际类型来执行对应的方法。

Spring中能有比较好的体现

## 

## 



