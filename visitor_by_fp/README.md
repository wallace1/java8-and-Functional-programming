# 函数式编程下的visitor模式

标签（空格分隔）： Java8 函数式编程 visitor模式 monad模式

---

在[深入理解函数式编程之monad][1]中，我们详细讲述了monad模式，以及monad模式和functor模式之间的区别。
这次，我们来使用monad到常规的设计模式中。
我们选取Visitor模式来作为第一个monad的应用。
Visitor模式在[Visitor模式示例][2]有一个典型的例子，其类图如下所示：
![Visitor模式demo类图][3]
在这个模式中，`Commander`、`Sergeat`和`Soldier`类基本上是不能再扩展的了。可以扩展的是`UnitVisitor`的子类。
从这个类图中，我们也可以看到，要实现一个示例的`Visitor`模式，需要8个类，并且有大量的样板代码：

    public class CommanderVisitor implements UnitVisitor {
	
	private static final Logger LOGGER = LoggerFactory.getLogger(CommanderVisitor.class);

	@Override
	public void visitSoldier(Soldier soldier) {

	}

	@Override
	public void visitSergeant(Sergeant sergeant) {

	}

	@Override
	public void visitCommander(Commander commander) {
		
		LOGGER.info("Good to see you {}", commander);
		
	}

}

上面的代码中，`visitSoldier`和`visitSergeant`方法就是典型的样板代码。
现在，我们使用monad来实现Visitor模式，又会是什么样子呢？
首先是`Soldier`类：

    public class Soldier {
        @Override
        public String toString() {
            return "soldier";
        }
    }
其次是`Sergeat`类：

    public class Sergeant {

        private List<Soldier> soldiers = new ArrayList<>();

        public void addSoldier(Soldier soldier)
        {
            this.soldiers.add(soldier);
        }

        public List<Soldier> getSoldiers() {
            return Collections.unmodifiableList(soldiers);
        }

        @Override
        public String toString() {
            return "sergeant";
        }
    }
然后是`Commander`类：

    public class Commander {

        private List<Sergeant> sergeants = new ArrayList<>();

        public void addSergeant(Sergeant sergeant)
        {
            this.sergeants.add(sergeant);
        }

        public List<Sergeant> getSergeants() {
            return Collections.unmodifiableList(sergeants);
        }

        @Override
        public String toString() {
            return "commander";
        }
    }
最后是monad类：

    public class VistorMonad<T> {

        private List<T> ts;

        public VistorMonad(List<T> ts) {
            this.ts = ts;
        }

        public<U> VistorMonad<U> flatMap(Function<? super T, List<U>> mapper) {
            return new VistorMonad<U>(this.ts.stream().flatMap(t -> mapper.apply(t).stream()).collect(Collectors.toList()));
        }
    }
可以看到，在`monad`类的`flatMap`方法中，我们明细有代码做了拆箱功能：

    this.ts.stream().flatMap(t -> mapper.apply(t).stream())
在`functor`中是不会有这样的拆箱工作的，因为`functor`里面的包装是一层包装，而`monad`里面的包装是多层包装，这是他们之间的最大区别。
最后，我们来使用这个模式.
先是初始化数据：

        Sergeant sergeant1 = new Sergeant();
        sergeant1.addSoldier(new Soldier());
        sergeant1.addSoldier(new Soldier());
        sergeant1.addSoldier(new Soldier());

        Sergeant sergeant2 = new Sergeant();
        sergeant2.addSoldier(new Soldier());
        sergeant2.addSoldier(new Soldier());
        sergeant2.addSoldier(new Soldier());

        Commander commander = new Commander();
        commander.addSergeant(sergeant1);
        commander.addSergeant(sergeant2);

        List<Commander> commanders = new ArrayList<>();
        commanders.add(commander);
    

最后就可以使用Visitor模式了：

    new VistorMonad<Commander>(commanders).flatMap(c -> {
            LOGGER.info("Good to see you {}", commander);
            return commander.getSergeants();
        }).flatMap(s -> {
            LOGGER.info("Hello {}", s);
            return s.getSoldiers();
        })
        .flatMap(soldier -> {
            LOGGER.info("Greetings {}", soldier);
            return new ArrayList<>();
        });
    

是不是很简单，省了三个Visitor类。
除了代码简单了很多以外，还可以对`Soldier`、`Sergeant`和`Commander`类进行灵活扩展。
比如，我们需要增加一个`General`类，就很容易了：

    public class General {

        private List<Commander> commanders = new ArrayList<>();

        public void addCommander(Commander commander)
        {
            this.commanders.add(commander);
        }

        public List<Commander> getCommanders() {
            return Collections.unmodifiableList(commanders);
        }

        @Override
        public String toString() {
            return "general";
        }
    }
我们再对这四个类做Visitor模式的测试！
首先，还是初始化数据：

        Sergeant sergeant1 = new Sergeant();
        sergeant1.addSoldier(new Soldier());
        sergeant1.addSoldier(new Soldier());
        sergeant1.addSoldier(new Soldier());

        Sergeant sergeant2 = new Sergeant();
        sergeant2.addSoldier(new Soldier());
        sergeant2.addSoldier(new Soldier());
        sergeant2.addSoldier(new Soldier());

        Commander commander = new Commander();
        commander.addSergeant(sergeant1);
        commander.addSergeant(sergeant2);

        General general = new General();
        general.addCommander(commander);

        List<General> generals = new ArrayList<>();
        generals.add(general);

测试：

    new VistorMonad<General>(generals).flatMap(g -> {
            LOGGER.info("Hi {}", g);
            return g.getCommanders();
        })
                .flatMap(c -> {
                    LOGGER.info("Good to see you {}", c);
                    return c.getSergeants();
                })
                .flatMap(s -> {
            LOGGER.info("Hello {}", s);
            return s.getSoldiers();
        })
                .flatMap(soldier -> {
                    LOGGER.info("Greetings {}", soldier);
                    return new ArrayList<>();
                });
            

看看，是不是扩展很简单？


----------
参考文献：[Visitor模式参考文献][4]


  [1]: https://github.com/wallace1/java8-and-Functional-programming/tree/master/monad1
  [2]: https://github.com/iluwatar/java-design-patterns/tree/master/visitor
  [3]: https://raw.githubusercontent.com/iluwatar/java-design-patterns/master/visitor/etc/visitor_1.png
  [4]: https://github.com/iluwatar/java-design-patterns/tree/master/visitor