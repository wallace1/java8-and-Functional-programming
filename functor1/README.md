# 深入理解函数式编程之functor

标签（空格分隔）： Java 函数式编程 函子 functor

---

在函数式编程中，函子(functor)可以说是一个很基础的概念。
当然了，还有一个更基础的概念是函数(function)。函数是每一个编程语言都会涉及到的概念，它实际上表达的是一个对象到另一个对象的映射关系。请原谅我在这里使用了对象一词，因为作为一个Java程序员，对象可以说是最熟悉的概念。
上面的一个对象，指的是函数的输入参数；而另外一个对象，则是指函数的返回参数；映射关系指的是函数。
比如：

    public int add1(int someNumber)
    ｛
        return someNumber + 1;
    ｝

这就是一个典型的函数。
函数处理的是对象，而函子则处理的是范畴。所谓“范畴”，从编程的角度来理解，我们可以理解成高阶对象。
那么，什么是“高阶对象”？
实际上，就是更为复杂的对象，或者类型。
比如：
一个int类型的对象是一个简单对象，那么一个int数组则是一个更为高阶的对象。
一个IM类型对象是一个简单对象。

    class Communication
    ｛
        private int type;
    ｝
    class IM extends Communication
    ｛
        private String code;
    ｝
    class Address
    ｛
        private String provinceId;
        private String cityId;
        private String district;
        private String detail;
    ｝
    class Employee
    {
        private int emplNo;
        private String firstName;
        private String lastName;
        private Communication communication;
        private Address addresse;
    }
那么，一个Employee对象就是另外一种更为高阶的对象，或类型，注意：为了减少代码量，省略了get和set方法。
前面说了，函数add1是对简单的int对象做映射；那么对应的函子则是对int数组做映射，具体说来就是对int数组的每一个元素做加1的映射。
说到这里，我们可以想到Java8的Stream对象可以做这样的映射。对了，Stream对象就是一个函子，它实现函子功能的函数就是map函数。
因此，所有的函子必须实现map函数。
像下面的例子，就是Stream作为函子的一个典型应用：

    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    list.stream().map(i -> i + 1).forEach(System.out::println);
所谓函子，就是把一个作用于简单对象的函数，应用到一个高阶对象的过程。比如，上例中的`i + 1`函数，本来是作用于int类型对象的，现在我们通过`map`方法，把它应用到`List<Integer>`对象上，所以，Stream就是一个典型的函子。
重要的事情说三遍，下面再以图例的形式来说说如何从函数到函子的：
我们把简单对象称之为“值”：
![简单对象][1]
下面的图例表示简单对象和函数之间的关系：
![简单对象与函数][2]
高阶对象好比是一个打包了的对象：
![高阶对象][3]
对于高阶对象，不能直接应用函数：
![高阶对象与函数][4]
我们要定义一个新的运算-map来解决在高阶对象上使用函数的问题：
![高阶对象与函子][5]
这个实现了map方法的类，我们就称之为函子。
所谓的map方法，就是将高阶对象打开，取出里面的简单对象，然后再对简单对象应用map方法传递过来的函数，再把应用函数后的结果又包装成高阶对象，最后将这个高阶对象返回。
我们都知道，对于上例中的int[]对象，我们可以通过使用Stream对象来构建函子，可以完成一系列的map运算，得到我们想要的结果，在这里就不再详细阐述。
下面，我们来看看上面的Employee对象，如何应用到函子。
首先，我们需要创建一个函子接口和它的子类：

    public interface Functor<T, F extends Functor<?, ?>> {
        <R> F map(Function<T, R> f);
    }
    
    public class CommonFunctor<T> implements Functor<T, CommonFunctor<?>> {
        private T value;
        public CommonFunctor(T value) {
            this.value = value;
        }
        @Override
        public <R> CommonFunctor<R> map(Function<T, R> f) {
            final R result = f.apply(value);
            return new CommonFunctor<>(result);
        }
        public T getValue() {
            return value;
        }
    }
我们再次可以看到，所谓的函子，就是实现了map方法的对象。在map方法中，很重要的是，必须在map方法体内包装函子对象，然后返回，如`return new CommonFunctor<>(result);`，这是函子(functor)和单子(monad)的最大区别。单子，我们后续会讲到。
最后，看看我们如何使用这个函子的：

    IM qq = new IM();
    qq.setType(1);
    qq.setCode("123223");

    Address address = new Address();
    address.setProvince("浙江");
    address.setCity("杭州");
    address.setDistrict("西湖");
    address.setDetail("西湖国家广告产业园");

    Employee zhangsan = new Employee();
    zhangsan.setEmplNo(23);
    zhangsan.setFirstName("张");
    zhangsan.setLastName("三");
    zhangsan.setCommunication(qq);
    zhangsan.setAddresse(address);
以上是初始化一些数据，用来测试，下面是如何使用函子的代码：

    CommonFunctor<Employee> employeeCommonFunctor = new CommonFunctor<>(zhangsan);
通过函子求该员工的全名：

    String fullName = employeeCommonFunctor.map(employee -> employee.getFirstName()+employee.getLastName()).getValue();
通过函子求该员工的全地址：

    String fullAddress = employeeCommonFunctor.map(Employee::getAddress).map(addr -> addr.getProvince()+addr.getCity()+addr.getDistrict()+addr.getDetail()).getValue();
这是函子的一些基本用法，我们在Stream类的用法中可以深刻的感受到。
在后面的章节，我们将陆续讲讲函子在传统的设计模式中是如何使用的，敬请关注，谢谢！


----------
参考文献：[图解 Monad-阮一峰][6]


  [1]: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071603.png
  [2]: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071604.png
  [3]: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071608.png
  [4]: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071609.png
  [5]: http://www.ruanyifeng.com/blogimg/asset/2015/bg2015071610.png
  [6]: http://www.ruanyifeng.com/blog/2015/07/monad.html