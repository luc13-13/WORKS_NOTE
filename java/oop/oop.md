# 继承
子类可以继承、调用和重写父类的public方法，并可以通过super指示编译器调用父类方法

```java
// 父类
public class Person {

    protected String ID = "123456";
    public String name;
    public int age;

    public Person(String name, int age, String ID) {
        this.name = name;
        this.age = age;
        this.ID = ID;
    }

    public String getName() {
        return this.name;
    }
}

// 子类
class Adult extends Person{

    public String job;
    public String salary;

    public Adult(String name, int age, String ID) {
        super(name, age, ID);
    }

    public Adult(String name, int age, String ID,  String job, String salary) {
        super(name, age, ID);
        this.job = job;
        this.salary = salary;
    }

    // 重写父类方法
    public String getName() {
        return this.name + ":" + this.job;
    }
    public String getSalary() {
        return salary;
    }
```
# 多态

Java中的对象分为编译阶段和运行阶段，支持将父类的引用，指向子类的实例对象。虚拟机会加载子类和父类的方法表(包括方法签名和参数列表)，
```java
public static void main(String[] args) {
        // 父类的引用指向子类实例，编译阶段，可以调用与父类的同名方法
        Person adult = new Adult("Mike", 30, "id","Programmer", "10000");
        // 运行阶段时，调用getName方法时，实现获取其运行类型的方法表，如果没有找到，再去编译类型方法表中查找
        System.out.println(adult.getName());
        Method[] pMethods = Person.class.getDeclaredMethods();
        // 打印adult的方法列表，及其声明类型
        for(Method method : pMethods) {
            System.out.println(method.getName() +":" + method.toGenericString());
            //getSalary:public java.lang.String algorithm.Adult.getSalary()
            //getName:public java.lang.String algorithm.Adult.getName()
        }
    }

```



