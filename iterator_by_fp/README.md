# 函数式编程下的Iterator模式

标签（空格分隔）： Java 函数式编程 Iterator模式

---

在模式下，Iterator模式是一个思路相对简单的模式。
迭代器（Iterator）模式，又叫做游标（Cursor）模式。GOF给出的定义为：提供一种方法访问一个容器（container）对象中各个元素，而又不需暴露该对象的内部细节。 
在“iluwatar/java-design-patterns”中，有一个Iterator模式例子，其类图如下所示：
![Iterator模式示例类图][1]

Iterator模式的关键就是要对外公开一个迭代器接口，该接口拥有`hasNext()`和`next()`这两个方法，如下所示：

    public interface ItemIterator {
        boolean hasNext();
        Item next();
    ｝
然后在迭代器的实现类里实现这两个方法，具体的代码就不再给出，请参考：[一个关于Iterator模式的例子][2]

我们知道，Iterator模式能够实现迭代的关键是`next()`方法；而函数式编程下，Stream类也有很多方法迭代的方法，如`forEach`和`peak`等，因此，我们认为实现了函子(functor)的Stream类就是一个完整的迭代器。
下面，我们使用Stream类实现“iluwatar/java-design-patterns”中的例子。
ItemType这个类还是不变：

    public enum ItemType {
        ANY, WEAPON, RING, POTION
    ｝
Item类也不变：

    public class Item {
        private ItemType type;
        private String name;
        public Item(ItemType type, String name) {
            this.setType(type);
            this.name = name;
        ｝
        @Override
        public String toString() {
            return name;
        }
        public ItemType getType() {
            return type;
        }
        public final void setType(ItemType type) {
            this.type = type;
        }
    ｝
变化较大的是TreasureChest类：

    public class TreasureChest {
        private List<Item> items;
        public TreasureChest() {
            items = new ArrayList<>();
            items.add(new Item(ItemType.POTION, "Potion of courage"));
            items.add(new Item(ItemType.RING, "Ring of shadows"));
            items.add(new Item(ItemType.POTION, "Potion of wisdom"));
            items.add(new Item(ItemType.POTION, "Potion of blood"));
            items.add(new Item(ItemType.WEAPON, "Sword of silver +1"));
            items.add(new Item(ItemType.POTION, "Potion of rust"));
            items.add(new Item(ItemType.POTION, "Potion of healing"));
            items.add(new Item(ItemType.RING, "Ring of armor"));
            items.add(new Item(ItemType.WEAPON, "Steel halberd"));
            items.add(new Item(ItemType.WEAPON, "Dagger of poison"));
        ｝
        public Stream<Item> stream()
        {
            return this.items.stream();
        }
    

此处输入代码基于Stream类的迭代器实现起来比Iterator模式简单多了，因为Stream类本身就实现了迭代方法。
下面，我们来看如何使用：

    public class App {
        private static final Logger LOGGER = LoggerFactory.getLogger(App.class);
        public static void main(String[] args) {
            TreasureChest chest = new TreasureChest();
            chest.stream().filter(item -> item.getType() == RING).forEach(item -> LOGGER.info(item.toString()));
            LOGGER.info("----------");
            chest.stream().filter(item -> item.getType() == POTION).forEach(item -> LOGGER.info(item.toString()));
            LOGGER.info("----------");
            chest.stream().filter(item -> item.getType() == WEAPON).forEach(item -> LOGGER.info(item.toString()));
            LOGGER.info("----------");
            chest.stream().filter(item -> item.getType() != ANY).forEach(item -> LOGGER.info(item.toString()));
        }
    }
 可以看到，借助于Stream类和函数式编程的迭代，远比面向对象实现Itreator模式的迭代要简单得多，而更加灵活的函数式编程实现的迭代，只需要我们掌握了函子(functor)，就可以实现。
 


----------
参考文献：[模式-Iterator模式][3]


  [1]: https://raw.githubusercontent.com/iluwatar/java-design-patterns/master/iterator/etc/iterator_1.png
  [2]: https://github.com/iluwatar/java-design-patterns/tree/master/iterator
  [3]: https://github.com/iluwatar/java-design-patterns/tree/master/iterator