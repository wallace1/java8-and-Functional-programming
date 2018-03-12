# 函数式编程下的策略模式

标签（空格分隔）： Java 函数式编程 策略模式

---

策略模式是我们常用的一种设计模式。
它属于对象的行为模式。其用意是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。策略模式使得算法可以在不影响到客户端的情况下发生变化。
具体的策略模式原理和实现方法，在此不详述，不熟悉的读者可以查看[github的该专题-iluwatar/java-design-patterns][1]。
其中有一个典型的例子，其类图如下所示：
![策略模式的一个实现类图][2]
从该类图可以看出，基于Java对象的策略模式实现起来，比较复杂，需要一个策略接口(DragonSlayingStrategy)和三个具体的实现(ProjectileStrategy、meleeStrategy、SpellStrategy)。
在Java8以前的这样纯面向对象的语言中，这样实现策略模式，是迫不得已的方法。其实，根本原因就在于函数在纯面向对象的语言中不能独立存在，它只能依附于一个接口，然后通过实现接口来具体实现。
由此，我们可以想到，在函数式编程模式下，由于函数可以以lambda表达式独立存在，那么，我们的策略模式实现起来，是不是就不一样了。
下面，我们来看看基于函数式编程下的策略模式的实现。
首先，我们可以想到，“DragonSlayingStrategy”接口在函数式编程模式下，可以作为一个函数式接口。
其次，三个策略的具体实现，也不必依赖于实体类了。
DragonSlayingStrategy接口的实现大概就是下面的样子了：

    @FunctionalInterface
    public interface DragonSlayingStrategy {
        void execute();
    ｝

多了这么一个函数式接口，是因为Java8的函数式接口中，只有Supplier、Consumer、Function等少量的几个，更多的需要我们自己实现。如果用到了生产者、消费者或Function这几个接口，那么连策略的接口都可以不用实现了。
接着，我们来实现策略，我们可以把所有的策略放在一个类里实现，减少策略类的个数：

    public class StrategyType {
        private static final Logger LOGGER = LoggerFactory.getLogger(StrategyType.class);
        public static DragonSlayingStrategy getStrategy(String type) {
            if ("MeleeStrategy".equals(type)) return MELEE_STRATEGY;
            else if ("ProjectileStrategy".equals(type)) return PROJECTILE_STRATEGY;
            else if ("SpellStrategy".equals(type)) return SPELL_STRATEGY;
            else return MELEE_STRATEGY;
        ｝
        private static DragonSlayingStrategy MELEE_STRATEGY = () -> {LOGGER.info("With your Excalibur you sever the dragon's head!");};
        private static DragonSlayingStrategy PROJECTILE_STRATEGY = () -> {LOGGER.info("You shoot the dragon with the magical crossbow and it falls dead on the ground!");};
        private static DragonSlayingStrategy SPELL_STRATEGY = () -> {LOGGER.info("You cast the spell of disintegration and the dragon vaporizes in a pile of dust!");};
    ｝

最后是DragonSlayer类，它基本上变化不大：

    public class DragonSlayer {
        private static final Logger LOGGER = LoggerFactory.getLogger(DragonSlayer.class);
        private DragonSlayingStrategy strategy;
        public DragonSlayer(DragonSlayingStrategy strategy) {
            this.strategy = strategy;
        ｝
        public void changeStrategy(DragonSlayingStrategy strategy) {
            this.strategy = strategy;
        }
        public void goToBattle() {
            this.strategy.execute();
        }
    }

测试代码如下所示：

    public class App {
        private static final Logger LOGGER = LoggerFactory.getLogger(App.class);
        public static void main(String[] args) {
            // GoF Strategy pattern
            LOGGER.info("Green dragon spotted ahead!");
            DragonSlayer dragonSlayer = new DragonSlayer(getStrategy("MeleeStrategy"));
            dragonSlayer.goToBattle();
            LOGGER.info("Red dragon emerges.");
            dragonSlayer.changeStrategy(getStrategy("ProjectileStrategy"));
            dragonSlayer.goToBattle();
            LOGGER.info("Black dragon lands before you.");
            dragonSlayer.changeStrategy(getStrategy("SpellStrategy"));
            dragonSlayer.goToBattle();
        ｝
    ｝

可以看到，函数式编程确实大大的减少了模式实现的类个数和代码量，同时，也大大增加了代码实现的灵活度。在后面其他模式的函数式编程实现中，我们还会继续看到这些效果。
实际上，如果某些策略只在某个客户端代码中有用，在其他地方不太用到，那么，我们的策略代码也可以转移到客户端中实现，这样，StrategyType类都没有存在的意义了：

    LOGGER.info("Green dragon spotted ahead!");
    dragonSlayer = new DragonSlayer(
        () -> LOGGER.info("With your Excalibur you severe the dragon's head!"));
    dragonSlayer.goToBattle();
    LOGGER.info("Red dragon emerges.");
    dragonSlayer.changeStrategy(() -> LOGGER.info(
        "You shoot the dragon with the magical crossbow and it falls dead on the ground!"));
    dragonSlayer.goToBattle();
    LOGGER.info("Black dragon lands before you.");
    dragonSlayer.changeStrategy(() -> LOGGER.info(
        "You cast the spell of disintegration and the dragon vaporizes in a pile of dust!"));
    dragonSlayer.goToBattle();


----------
参考文献：[iluwatar/java-design-patterns][3]


  [1]: https://github.com/iluwatar/java-design-patterns/tree/master/strategy
  [2]: https://raw.githubusercontent.com/iluwatar/java-design-patterns/master/strategy/etc/strategy_1.png
  [3]: https://github.com/iluwatar/java-design-patterns