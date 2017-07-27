# 一口气写完23种设计模式

标签： Java 设计模式

---

# 创建型模式
### 1.工厂模式
定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类吧实例化推迟到子类。
```
// 产品类
public interface IProduct {
}
public class ProductA1 implements IProduct{}
public class ProductA2 implements IProduct{}
public class ProductB1 implements IProduct{}
public class ProductB2 implements IProduct{}
// 产品枚举类
public enum ProductEnum {
    A1, A2, B1, B2
}
// 工厂类
public interface IFactory {
    IProduct create(ProductEnum productEnum); // 此处加入类型参数，只是为了更好的展示工厂方法
}
// 工厂A （工厂子类--看起来是不是像简单工厂，嘿嘿）
public class FactoryA implements IFactory {
    public IProduct create(ProductEnum productEnum) {
        if(ProductEnum.A1.equals(productEnum)) {
            return new ProductA1();
        } else if(ProductEnum.A2.equals(productEnum)) {
            return new ProductA2();
        }
        return null;
	}
}
// 工厂B
public class FactoryB implements IFactory {
    ....
}
// 客户端调用
// 创建产品A1
IFactory factoryA = new FactoryA();
factoryA.create(ProductEnum.A1);
// 创建产品B2
IFactory factoryB = new FactoryB();
factoryB.create(ProductEnum.B2)
```
> 简单工厂和工厂方法的区别：简单工厂把全部的事情在一个地方处理完了，然而工厂方法却是创建一个框架，让子类决定要如何实现。

```
// 简单工厂
public class SimpleFactory implements IFactory {
    public IProduct create(ProductEnum productEnum) {
        switch(productEnum) {
            case A1:
                return new ProductA1();
            case A2:
                return new ProductA2();
            case B1:
                return new ProductB1();
            case B2:
                return new ProductB2();
        }
        return null;
    }
}
```

### 2.抽象工厂
提供了一个接口，用于创建相关或依赖对象的**家族**，而不需明确指明具体类。
```
// 产品家族 之 产品A
public interface IProductA {
}
public class ProductA1 implements IProductA{} // 1号产品A
public class ProductA2 implements IProductA{} // 2号产品A
// 产品家族 之 产品B
public interface IProductB {
}
public class ProductB1 implements IProductB{} // 1号产品B
public class ProductB2 implements IProductB{} // 2号产品B
// 工厂类 -- 注意：如果需要增加C类产品，必须改变接口
public interface IFactory {
    IProductA createProductA();
    IProductB createProductB();
}
// 工厂1: 具体工厂使用“工厂方法”来实现
public class Factory1 implements IFactory {
    public IProduct createProductA() {
        return new ProductA1();
    }
    
    public IProduct createProductB() {
        return new ProductB1();
    }
}
// 工厂2
public class Factory2 implements IFactory {
    ....
}
// 商店
public class Store {
    private IFactory factory;
    
    public Store(IFactory factory) {
        this.factory = factory;
    }
} 
// 客户端调用
// 创建1号产品
Store store1 = new Store(new Factory1());
store1.createProductA();
store1.createProductB();
// 创建2号产品
Store store2 = new Store(new Factory2());
store2.createProductA();
store2.createProductB();
```

### 3.建造者模式
又称“生成器模式”，封装一个产品的构造过程，并允许按步骤构造。
> StringBuilder

### 4.单例模式
确保一个类只有一个实例，并提供一个全局访问点
```
// 懒汉式：双重校验锁
public class Singleton {
    private volatile static Singleton instance;
    
    private Singleton() {}
    
    public Singleton getInstance() {
        if(instance == null) { // Single Checked
            synchronized(Singleton.class) {
                if(instance == null) { // Double Checked
                    return new Singleton();
                }
            }
        }
        return instance;
    }
}
```
> ``volatile``
有些人认为使用volatile的原因是可见性，也就是可以保证线程在本地不会存有 instance 的副本，每次都是去主内存中读取。但其实是不对的。使用volatile的主要原因是其另一个特性：**禁止指令重排序优化**。也就是说，在volatile变量的赋值操作后面会有一个内存屏障（生成的汇编代码上），读操作不会被重排序到内存屏障之前。

```
// 静态内部类
public class Singleton {
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    private Singleton (){}  
    
    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
```
// 饿汉式
public class Singleton{
    //类加载时就初始化
    private static final Singleton instance = new Singleton();
 
    private Singleton(){}
 
    public static Singleton getInstance(){
        return instance;
    }
}
```

### 5.原型模式
当创建给定类的实例的过程很昂贵或很复杂时，就使用原型模式。
> Object.clone()

# 结构型模式
### 1.适配器模式
将一个类的接口，转换成客户期望的另一个接口。适配器让原本不兼容的类可以合作无间。
```
/** 对象适配器 -- 比较常用 **/
public interface ITarget {
    void request();
}
// 被适配类
public class Adaptee {
    public void specificRequest(){}
}
// 适配类：采用组合的方法
public class Adapter implements ITarget {
    private Adaptee adaptee;
    
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    
    public void show() {
        adaptee.specificRequest();
    }
}
// 客户端调用
ITarget target = new ITarget(new Adaptee());
target.request();
```
```
/** 类适配器 **/
public interface ITarget {
    void request();
}
// 被适配类
public class Adaptee {
    public void specificRequest(){}
}
// 适配类：采用继承的方式
public class Adapter extends Adaptee implements ITarget {
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    
    public void show() {
        specificRequest();
    }
}
// 客户端调用
ITarget target = new ITarget();
target.request();
```

### 2.桥接模式
不知改变你的实现，也改变你的抽象。
> 用于处理两个或多个维度的变化的场景

### 3.组合模式
允许你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。
> 其实就是在具体类中维护一组Item
> 组合模式虽然违反了单一原则，但更有价值

### 4.装饰模式
动态的将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。
```
// 抽象组件类
public abstract class AbsComponent {
    public abstract void perform();
}
// 组件
public class Component extends AbsComponent {
    public void perform() {
        ....
    }
}
// 抽象装饰者(定义此类只是为了更好的分离)
public abstract class AbsDecorator extends AbsComponent {
}
// 装饰者A
public class DecoratorA extends AbsDecorator {
    AbsComponent component; // 引用组件类，用于装饰
    
    public DecoratorA(AbsComponent component) {
        this.component = component;
    }
    
    public void perform() {
       ....
    }
}
// 装饰者A
public class DecoratorB extends AbsDecorator {
    ....
}
// 客户端调用
Component component = new Component();
DecoratorA decoratorA = new DecoratorA(component);
DecoratorB decoratorB = new DecoratorB(decoratorA);
decoratorB.perform();
```
> 实际场景： InputStream FileInputStream...

### 5.外观模式
提供一个统一的接口，用来访问子系统的一群接口。外观定义了一个高层接口，让子系统更容易使用。

### 6.享元模式
如想让某个类的一个实例能用来提供许多“虚拟实例”，就使用享元（Flyweight）模式。
> 应用场景：缓存

### 7.代理模式
为另一个对象提供一个替身或占位符以控制对这个对象的访问。
```
/** 静态代理 **/
// 被代理类
public interface ISubject {
    void request();
}
public class RealSubject implements ISubject {
    public void request() {}
}
// 代理类
public class Proxy implements ISubject {
    private ISubject subject;
    
    public Proxy(ISubject subject) {
        this.subject = subject;
    }
    
    public void request() {
        subject.request();
    }
}
// 客户端调用
ISubject subject = new RealSubject();
Proxy proxy = new Proxy(subject);
proxy.request();
```
```
/** JDK动态代理：被代理类必须实现接口 **/
public class MyInvocationHandler implements InvocationHandler {
    private Object target;
   
    public DyProxy(Object target) {
        this.target = target;
    }
    
    @override
    public Object invoke(Object proxy, Method method, Object[] args) throw Throwable {
        // 此处可添加前置条件
        Object result = method.invoke(target, args);
        // 此处可添加后置条件
        return result;
    }
    
    // 获取代理对象
    public Object getProxy() {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        Class<?>[] infs = target.getClass().getInterfaces();
        return Proxy.newProxyInstance(loader, infs, this);
    }
}
// 客户端调用
ISubject subject = new RealSubject();
MyInvocationHandler handler = new MyInvocationHandler(serive);
ISubject subjectProxy = (ISubject)handler.getProxy();
subjectProxy.request();
```
> JDK动态代理只能对实现了接口的类生成代理，而不能针对类
> CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法(又称织入)

# 行为型模式
### 1.责任链模式
当你想要让一个以上的对象有机会能够处理某个请求的时候，就是用责任链模式。
> FilterChain
> 游戏： 击鼓传花

### 2.命令模式
将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持撤销操作。
```
// 命令
public interface ICommand {
    void exec();
    void undo();
}
// 接收者
public class Receiver {
    public void action() {
        ....
    }
}
// 命令实现类
public class CommandImpl {
    private Receiver receiver;
    
    public CommandImpl() {
        this.receiver = receiver;
    }

    public void exec() {
        receiver.action();
    }
    ...
}
// 调用者
public class Invoker {
    private ICommand command;
    
    public void setCommand(ICommand command) {
        this.command = command;
    }
    
    public void exec() {
        command.exec();
    }
}
// 客户端调用
ICommand command = new CommandImpl(new Receiver());
Invoker invoker = new Invoker();
invoker.setCommand(command);
invoker.exec();
```
> 应用场景：队列请求、日志请求

### 3.解释器模式
为语言创建解释器
> 没用过也没想用过...

### 4.迭代器模式
提供一个方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。
> Iterator

### 5.中介者模式
集中对象之间复杂的沟通和控制方式。
> 举个栗子：CPU调度

### 6.备忘录模式
让对象返回之前的状态（例如:撤销）。
> 说白了，就是在内部维护一个state

### 7.观察者模式
定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖着都会收到通知并自动更新。
```
// 观察者接口
public interface IObserver {
    void update();
}
// 观察者A
public class ObserverA implements IObserver {
    ....
}
// 观察者B
public class ObserverB implements IObserver {
    ....
}
// 主题接口
public interface ISubject {
    void register(IObserver observer);
    void remove(IObserver observer);
    void notifyObservers();
}
// 主题实现类
public class SubjectImpl implements ISubject {
    private List<IObserver> observers;
    
    public SubjectImpl() {
        observers = new ArrayList<>();
    }
    
    public void register(IObserver observer) {
        observers.add(observer);
    }
    
    public void remove(IObserver observer) {
        observers.remove(observer);
    }
    
    public void notifyObservers() {
        for(IObserver observer : observers) {
            observer.update();
        }
    }
}
// 客户端调用
ISubject subject = new SubjectImpl();
subject.register(new ObserverA());
subject.register(new ObserverB());
subject.notifyObservers();
```
```
/** JDK内置的观察者模式：通过继承Observable实现通知 **/
// 观察者
public interface Observer {
    void update(Observable o, Object arg);
}
// 主题
public class Observable {
    private boolean changed = false;
    private Vector obs;
    
    public synchronized void addObserver(Observer o) {
        ....
        obs.addElement(o);
    }
    
    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }
    
    public void notifyObservers(Object arg) {
        ....
        ((Observer)arrLocal[i]).update(this, arg);
        ....
    }
    
    // 多了一个setChange方法，用于保证当达到临界条件时，才会通知，而不
    // 是只要改变就通知
    protected synchronized void setChanged() {
        changed = true;
    }
}
```

### 8.策略模式
定义了算法家族，分别封装起来，让它们之间可以相互替换，此模式让算法的变化独立于使用算法的客户。
```
// 策略接口
public interface IStrage {
    void opreate();
}
// 具体策略类
public class StrageA implements IStrage {
    ....
}
public class StrageB implements IStrage {
    ....
}
// 上下文
public class Context {
    private IStrage strage;
    public void setStrage(IStrage strage) {
        this.strage = strage;
    }
    public void opreate() {
        strage.opreate();
    }
}
// 客户端调用
IStrage strage = new StrageA();
Context ctx = new Context();
ctx.setStrage(strage);
ctx.opreate();
```
> 应用场景：session策略

### 9.状态模式
允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。
```
// 状态接口
public interface IState {
    void handle();
}
// 具体状态类A
public class StateA implements IState {
    private Context ctx;
    
    public StateA(Context ctx) {
        this.ctx = ctx;
    }
    
    public void handle() {
        // 可以直接处理
        ....
        // 也可以转移状态
        ctx.setState(ctx.getStateB());
    }
}
// 具体状态类B
public class StateB implements IState {
    ....
}
// 上下文
public class Context {
    private IState stateA;
    private IState stateB;
    
    private IState state = stateA; // 默认状态A
    
    public Context() {
        stateA = new StateA(this);
        stateB = new StateB(this);
    }
    
    public void setState(IState state) {
        this.state = state;
    }
    
    public void setStateA(IState stateA) {
        this.stateA = stateA;
    }
    
    public void setStateB(IState stateB) {
        this.stateB = stateB;
    }
    
    public void opreate() {
        state.handle();
    }
    ...
}
// 客户端调用
Context ctx = new Context();
ctx.opreate(); // 状态A -> 转移到B状态
ctx.opreate(); // 状态B
```
> **是不是和策略模式很像？**
> 以**状态模式**而言，我们将一群行为封装在状态对象中，context的行为随时可以委托到那些对象中的一个。随着时间的流逝，当前状态在状态对象集合中游走改变。但是context的客户对于状态对象了解不多，甚至根本是浑然不觉。
> 而以**策略模式**而言，客户通常注定指定Context所要组合的策略对象是哪一个。现在，固然策略模式让我们更具有弹性，能够在运行时改变策略，但对于某个context对象来说，通常都只有一个最适当的策略对象。

### 10.模版方法
在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模版方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。
> 其实就是使用抽象类

### 11.访问者模式
> 见名知义，最简单的体现就是``setter/getter``方法

# 设计原则
1. 封装变化
2. 针对接口编程，不针对实现编程
3. 多用组合，少用继承
4. 为交互对象之间的松耦合设计而努力
5. 开放-关闭原则：类应该对扩展开放，对修改关闭
6. 依赖倒置原则：依赖抽象，不要依赖具体类
7. 最少知识原则：减少对象之间的交互
8. 好莱坞原则：让低层组件挂钩进计算中，而又不会让高层组件依赖低层组件
9. 单一原则：一个类应该只有一个引起变化的原因
