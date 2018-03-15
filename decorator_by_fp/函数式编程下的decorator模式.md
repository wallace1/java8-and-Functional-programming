# 函数式编程下的decorator模式

标签（空格分隔）： Java 函数式编程 decorator模式

---

decorator模式被称为“装配器模式”，也叫“油漆工模式”。很形象的像油漆工刷油漆一样，一层一层的刷，功能一层一层的叠加。
解释decorator模式有一个很形象的例子，就是去咖啡店里喝咖啡。
一杯黑咖啡(原咖啡)假设卖10元，加糖得另付1元，加牛奶得另付3元，加鸡蛋另付2元，加冰淇淋另付4元，等等。所有买咖啡的顾客肯定都要黑咖啡，然后根据各人口味不同，有人喜欢加糖，有人喜欢加牛奶，有人喜欢加鸡蛋，还有人喜欢加三样、或更多。那么，我们要怎么很灵活的计算每一个顾客的费用呢？
对于这问题，使用decoractor模式是很好解决的。
首先，我们设计一个`Coffee`的接口：

    public interface Coffee {
        public int count();
    ｝
然后，我们可以单独计算每一种原料的价格：

    public class PureCoffee implements Coffee{
    ｛
        @Override
        public int count() {
            return 10;
        }
    ｝
    public class SugarCoffee implements Coffee{
        private Coffee coffee;
        public SugarCoffee(Coffee coffee) {
            this.coffee = coffee;
        }
        @Override
        public int count() {
            return this.coffee.count() + 1;
        }
    ｝
    public class MilkCoffee implements Coffee{
        private Coffee coffee;
        public MilkCoffee(Coffee coffee) {
            this.coffee = coffee;
        }
        @Override
        public int count() {
            return this.coffee.count() + 3;
        }
    ｝
下面，我们就可以使用这个模式了。
比如，我们买加糖的咖啡，需要多少钱？

    Coffee coffeeWithSugar = new SugarCoffee(new PureCoffee());
    coffeeWithSugar.count();
再比如，我们买加糖加牛奶的咖啡，需要多少钱？

    Coffee coffeeWithSugarAndMilk = new SugarCoffee(new MilkCoffee(new PureCoffee()));
    coffeeWithSugarAndMilk.count();
可以看到，在初始化咖啡类的时候，我们确实像刷油漆一样，一层一层的初始化。然后，调用`count`方法时，也是一层一层的调用的，原因是：

    this.coffee.count() + 1;
每次都会把前一个对象的`count`方法调用一次，跟刷油漆一样。
它的类图是这样的：
![decorator模式类图][1]
通过上面的例子，大家可以看到，这个模式确实很灵活，它唯一的不足是，我们需要创建很多的对象，如SugarCoffee、MilkCoffee等等。这使得我们实现这个模式的工作效率比较低。
现在，我们可以通过函数式编程来提高效率，较少类个数和代码量。
这次，我们通过函子(functor)来实现decorator模式。
首先，我们实现一个函子接口：

    public interface Functor<T, F extends Functor<?, ?>> {
        F map(Function<T, T> f);
    ｝
这个接口可以给我们大多数的函子类公用。
接着，我们就来实现这个函子：

    public class Coffee<Integer> implements Functor<Integer, Coffee<?>> {
        private Integer price;
        public Coffee(Integer price) {
            this.price = price;
        }
        @Override
        public Coffee<?> map(Function<Integer, Integer> f) {
            return new Coffee<>(f.apply(this.price));
        }
        public Integer getPrice() {
            return price;
        }
    ｝
最后，我们就可以使用这个函子类来实现decorator模式了,比如我们想买咖啡加糖加牛奶，所需要支付的费用就这样计算：

    Coffee coffee = new Coffee(5).map(p -> (Integer)p + 1).map(p -> (Integer)p + 2);
    coffee.getPrice();
是不是简单多了。
这就是函子(functor)在设计模式下的第一个应用，后面会路线讲到其他的应用，敬请关注，谢谢！



  [1]: https://thumbnail0.baidupcs.com/thumbnail/8ccffe5c3d09ce0c007bcd1f414cf4f7?fid=2048635255-250528-272929452653573&time=1521097200&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-imqX6MV41Ybgx3DEIyLqdlBHkd8=&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=1708331795493378943&dp-callid=0&size=c710_u400&quality=100&vuk=-&ft=video