# 深入理解函数式编程之monad

标签（空格分隔）： Java8 函数式编程 monad模式

---
## 从一个简单例子说起 ##
在[深入理解函数式编程之functor][1]中，我们给出了一个简单例子来说明`functor`函子，这个例子就是-我们有一个基于整型`List`对象，我们希望把该对象中的每一个元素都加1。我们使用`functor`是这么做的：

    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
    list.stream().map(i -> i + 1).forEach(System.out::println);、
通过上文的讲述，我们都知道，所谓`functor`模式，就是实现了`map`方法。
`functor`模式能够解决一系列这样的问题，但是遇到了下面的问题，又该如何解决呢？
现在，假设我们有如下的`List`对象：

    List<List<Integer>> lists = new ArrayList<>();
    lists.add(Arrays.asList(1));
    lists.add(Arrays.asList(2, 3));
    lists.add(Arrays.asList(4, 5, 6));
我们需要对`List`对象的每一个元素（`List`对象）的每一个元素（`Integer`对象）加1，该怎么做呢？
对于这么一个简单问题，我们就不能使用`functor`模式了，因为Stream对象的`map`后，里面的元素仍然是`List`对象，我们不能直接对`List`对象做加1的操作。
学过Stream对象的人都知道，这个问题是使用Stream对象的`flatMap`方法解决，代码如下：

    lists.stream().flatMap((list) -> list.stream()).map(i -> i + 1).forEach(System.out::println);
这里就使用了`monad`模式。
所谓的`monad`模式，在Java中，也就是指实现了`flatMap`函数的类。
## 图解monad模式 ##
`functor`模式的图例如下所示：
![functor模式过程][2]
根据图例，整个`functor`模式分为如下几步：

 1. 从包装里取出元素；
 2. 对每一个元素应用`map`方法里传递的函数；
 3. 得到了每一个新的元素（图例中假设经过函数运算后，每一个元素都变大了）；
 4. 再将新的元素重新放回包装里；
 5. 将整个包装返回。

而`monad`模式的图例则如下图所示：
![monad模式的过程][3]
根据图例，整个`monad`模式分为如下几步：

 1. `monad`模式的原始对象是一个大包装里套小包装的包装，如上图第一个长方体所示；
 2. 从小包装里取出元素，放到大包装里；
 3. 使得对象只有一个包装；
 4. 从包装里取出元素；
 5. 对每一个元素应用`map`方法里传递的函数；
 6. 得到了每一个新的元素（图例中假设经过函数运算后，每一个元素都变大了）；
 7. 再将新的元素重新放回包装里；
 8. 将整个包装返回。


 ## monad模式的简单应用 ##
 在SNS应用中，我们经常会查某个人的朋友的朋友，或者某个人的粉丝的粉丝，等等。
 某个人在SNS系统中是一个普通用户：
 

    public class User {
        private final String name;
        private final List<User> friends = new ArrayList<>();
        public void addFriend(User user)
        {
            this.friends.add(user);
        }
        public void addFriends(User... users)
        {
            this.friends.addAll(Arrays.asList(users));
        }
        public User(String name) {
            this.name = name;
        }
        public String getName() {
            return name;
        }
        @Override
        public String toString() {
            return "User{" +
                "name='" + name + '\'' +
                '}';
        }
        public List<User> getFriends() {
            return Collections.unmodifiableList(this.friends);
        }
    }

现在，我们来建立用户：

    User tom = new User("Tom");
    User mike = new User("Mike");
    User alice = new User("Alice");
    User jack = new User("Jack");
    User john = new User("John");
    User rose = new User("Rose");
    User ben = new User("Ben");

接着，我们来建立人与人之间的关系：

    tom.addFriends(mike, alice);

    mike.addFriends(jack, john, rose);

    alice.addFriends(rose, ben);

    jack.addFriends(rose, john);

    john.addFriends(rose, ben);

    rose.addFriends(tom, mike, alice);

    ben.addFriends(rose, jack);

然后，我们就可以查tom这个用户的朋友的朋友了：

    tom.getFriends().stream().flatMap(user -> user.getFriends().stream()).forEach(System.out::println);

我们还可以查tom这个用户朋友的朋友的朋友：

    tom.getFriends().stream().flatMap(user -> user.getFriends().stream()).flatMap(user -> user.getFriends().stream()).forEach(System.out::println);

看看，是不是很有意思？
后面，我们会接着讲讲monad在设计模式中的使用，敬请关注，谢谢！



 


  [1]: https://github.com/wallace1/java8-and-Functional-programming/tree/master/functor1
  [2]: https://raw.githubusercontent.com/wiki/wallace1/java8-and-Functional-programming/functor.png
  [3]: https://raw.githubusercontent.com/wiki/wallace1/java8-and-Functional-programming/monad.png