# functor模式的应用

标签（空格分隔）： Java8 函数式编程 functor

---

## 从一个问题说起 ##
假设要给一个公司的员工计算年假，一般来说，年假的计算规则是：

 1. 工龄小于10年的，年假5天；
 2. 工龄大于或等于10年，而小于20年的，年假10天；
 3. 工龄大于或者等于20年的，年假20天。


现在要计算一批员工的年假，并且求这批员工的总假期数。
下面我们先给出员工类的代码：

    public class Employee {
        private final int emplNo;
        private final String name;
        private final int age;
        private final int workAge;
        private int holidayCount;
        public Employee(int emplNo, String name, int age, int workAge) {
            this.emplNo = emplNo;
            this.name = name;
            this.age = age;
            this.workAge = workAge;
        }
        public int getEmplNo() {
            return emplNo;
        }
        public String getName() {
            return name;
        }
        public int getAge() {
            return age;
        }
        public int getWorkAge() {
            return workAge;
        }
        public int getHolidayCount() {
            return holidayCount;
        }
        public void setHolidayCount(int holidayCount) {
            this.holidayCount = holidayCount;
        }
    }
然后，我们给出这批员工的初始化代码：

    List<Employee> employees = new ArrayList<>();
    employees.add(new Employee(1, "Tom", 45, 20));
    employees.add(new Employee(3, "Alice", 28, 3));
    employees.add(new Employee(5, "Mike", 25, 1));
    employees.add(new Employee(9, "Jack", 36, 11));
    employees.add(new Employee(120, "Sam", 50, 25));
    employees.add(new Employee(56, "Rose", 32, 7));
    employees.add(new Employee(77, "Andy", 39, 14));
那么，我们使用传统的命令式编程，过程如下。
先计算每个员工的年假：

    for (Employee employee : employees)
    {
        if (employee.getWorkAge() < 10) employee.setHolidayCount(5);
        else if (employee.getWorkAge() >= 10 && employee.getWorkAge() < 20) employee.setHolidayCount(10);
        else employee.setHolidayCount(20);
    }
然后，再对所有员工的年假求和：

    int totalHolidayCount = 0;
    for (Employee employee : employees)
    {
        totalHolidayCount += employee.getHolidayCount();
    }

    System.out.println(totalHolidayCount);
那么，我们使用函数式编程，该如何解决这个问题呢？
## 基于functor模式的帮助类 ##
我们知道，Stream类有filter方法是对stream对象的元素进行过滤，或者是peek或forEach是遍历所有的元素，这些方法都无法替换原来命令式编程的`if...else if...else`这样的结构。
既然没有，那么我们就自己来实现。

    public class StreamIfUtil<T> {
我们把这个类定义为`Stream`类的`if...else if...else`结果解决方案帮助类。

    private Stream<T> stream;
这个帮助类的处理对象是Stream对象。

    private List<Predicate> predicates = new ArrayList<>();
存放if条件。

    private int count = 0;
一个帮助计数器。

    public StreamIfUtil(Stream<T> stream) {
        this.stream = stream;
    }
对外的构造器。

    public StreamIfUtil(Stream<T> stream, List<Predicate> predicates, int count) {
        this.stream = stream;
        this.predicates = predicates;
        this.count = count;
    }
自己要用到的构造器。

    public StreamIfUtil<T> select(Predicate<T> predicate)
    {
        this.predicates.add(predicate);
        return this;
    }
该对外方法用来接收条件输入，函数式编程通常把条件和基于条件的运算分开。

    public StreamIfUtil<T> with(Consumer<T> consumer) throws Exception
    {
        this.count++;
        final int port = this.count;
        if (this.predicates.isEmpty() || this.predicates.size() > this.count) throw new Exception("必须先调用select方法!");
        Stream<T> stream = this.stream.peek(t -> {
                                if (this.predicates.get(port - 1).test(t)) {
                                    consumer.accept(t);
                                }
                            });
        return new StreamIfUtil<>(stream, this.predicates, this.count);
    }
核心的函数，实现了functor模式，functor模式的方法，是参入函数，然后解包，对包里的元素进行传入参数运算，最后，将结果包装返回。

    public StreamIfUtil<T> elseWith(Consumer<T> consumer) throws Exception
    {
        if (this.predicates.isEmpty()) throw new Exception("必须先调用select方法!");
        Stream<T> stream = this.stream.peek(t -> {
                                if (elseTest(t))
                                {
                                    consumer.accept(t);
                                }
                            });
        return new StreamIfUtil<>(stream);
    }
核心函数，同样实现了functor模式。

    private boolean elseTest(T t)
    {
        return this.predicates.stream().allMatch(predicate -> !predicate.test(t));
    }
一个帮助方法。

    public Stream<T> getStream() {
        return stream;
    }
}


StreamIfUtil帮助类，最终要把Stream对象返回。
## 帮助类的使用 ##
最后，我们来使用这个帮助类：

    int totalHolidayCount = new StreamIfUtil<Employee>(employees.stream())
        .select(e -> e.getWorkAge() < 10).with(e -> e.setHolidayCount(5))
        .select(e -> e.getWorkAge() >= 10 && e.getWorkAge() < 20).with(e -> e.setHolidayCount(10))
        .elseWith(e -> e.setHolidayCount(20))
        .getStream().map(e -> e.getHolidayCount())
        .reduce(Integer::sum).get();
    System.out.println(totalHolidayCount);
可以看到，这是一个很纯粹的函数式编程模式。
## 小结 ##
functor模式在函数式编程中应用很广，很多地方都能见到它们的身影。
一个functor模式，在于实现这么一个方法：

 1. 传入参数是一个函数，如`with(Consumer<T> consumer)`中的`Consumer<T> consumer`
 2. 返回值是该类的对象，如`public StreamIfUtil<T> elseWith(Consumer<T> consumer)`中的`StreamIfUtil<T>`。

满足这两个条件，就是实现了functor模式。

