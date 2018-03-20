# 函数式编程下的简单工厂模式

标签（空格分隔）： Java8 函数式编程 简单工厂模式

---

工厂模式是我们比较常用的一种模式，工厂模式也有很多变形，其中，最简单是是简单工程模式。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。
意图：定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
主要解决：主要解决接口选择的问题。
何时使用：我们明确地计划不同条件下创建不同实例时。
如何解决：让其子类实现工厂接口，返回的也是一个抽象的产品。
关键代码：创建过程在其子类执行。
下面，我们举一个简单工厂模式的例子。
公司的员工有很多种类型，典型的如“普通员工”、“经理”和“老板”，做这样的区别是这些员工有很多不同，比如，发工资多少的不同。
下面，我们来模拟发工资的情况。
首先是员工类型：

    public enum EmployeeType {
        STAFF,
        MANAGER,
        BOSS
    ｝
然后，我们要定义一个抽象的员工接口：

    public interface Employee {
        public float salary();
    ｝
接着，是普通员工、经理和老板的具体实现：

    public class Staff implements Employee {
        @Override
	    public float salary() {
		    return 1000;
	    }
	｝
	
	public class Manager implements Employee {
	    @Override
	    public float salary() {
		    return 10000;
	    }
    ｝
    
    public class Boss implements Employee {
        @Override
        public float salary() {
		    return 1000000;
	    }
	｝
	

最后，我们就可以实现简单工厂模式了：

    public class SimpleFactory {
        public static Employee create(EmployeeType type)
        ｛
            Map<EmployeeType,Employee> map = new HashMap<>();
		
		    map.put(EmployeeType.STAFF, new Staff());
		    map.put(EmployeeType.MANAGER, new Manager());
		    map.put(EmployeeType.BOSS, new Boss());
		
		    return map.get(type);
        ｝
    ｝
简单工厂模式这样使用：

    Employee boss = EmployeeFactory.getEmployee(EmployeeType.BOSS);
    System.out.println(boss.salary());

这就完成一个简单工厂模式的实现，整个实现过程目标明确，实现起来也十分简单。
其类图如下所示：
![简单工厂模式类图][1]
从类图上可以看出，简单工厂模式使得客户端只依赖接口和工厂，不依赖各具体类的实现，如 Staff 、Manager 、 Boss等等。
这使得客户端在使用类、初始化类都很简单，这就是简单工厂模式的好处。
但是，使用面向对象语言实现简单工厂模式的不好的地方仍然是类的繁多和代码的冗余。我们需要实现三个实体类，每个类里也存在着框架代码和冗余代码。
下面，我们来看看如何使用函数式编程来实现简单工厂模式。
首先，员工类型不变，在这里就不再给出。
再来看看Employee，这里已经不是接口了，因为不需要接口：

    public class Employee {
        private Supplier<Float> supplier;
        public Employee(Supplier<Float> supplier) {
            this.supplier = supplier;
        }
        public float salary()
        ｛
            return supplier.get();
        ｝
    ｝
可以看到，由于函数式编程中，函数可以作为参数传递，所以我们只需要实现一个Employee类，然后实时的把`salary`方法的代码传递给对象就可以了。
我们最后来看看工厂类的实现：

    public class SimpleEmployeeFactory {
        private static final Map<EmployeeType, Employee> FACTORY = new HashMap<>();
        public static Employee getEmployee(EmployeeType type)
        ｛
            FACTORY.put(STAFF, new Employee(() -> new Float(1000)));
            FACTORY.put(MANAGER, new Employee(() -> new Float(10000)));
            FACTORY.put(BOSS, new Employee(() -> new Float(1000000)));
            return FACTORY.get(type);
        ｝
可以看到，使用了函数式编程的简单工厂模式实现起来就简单多了，也没有那么多的冗余代码。
应该把函数式编程大量应用到Java8及以上版本，用来减少Java的代码量。



  [1]: https://thumbnail0.baidupcs.com/thumbnail/87606d2347de99df6e5b7681c9cf07ca?fid=2048635255-250528-279357426542006&time=1521511200&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-b8ttnQQ%2bG7P9oPFdt3K4bbZuxXE=&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=1818792931620203618&dp-callid=0&size=c710_u400&quality=100&vuk=-&ft=video