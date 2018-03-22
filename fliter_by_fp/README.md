# 函数式编程下的filter模式

标签（空格分隔）： Java8 函数式编程 filter模式 过滤器模式 管道模式 拦截器模式

---
过滤器模式，也称为拦截器模式，还称为管道模式。
有3个具有代表性的定义：
定义1.Bucshmann & Meunier 定义:过滤器和管道体系结构风格为处理数据流的系统提供了一种结构。每个处理步骤封装在一个过滤器组件中。数据通过相邻过滤器之间的管道传输。重组过滤器可以建立相关系统族。
定义2.Shaw & Garlan定义: 管道和过滤器体系结构风格中的每个过滤器有一组输入端和输出端。一个过滤器从输入端读取数据流,通过本地转换和渐增计算,向输出端输出数据流。管道充当数据流的通道,将一个过滤器的输出端连接到另一个过滤器的输入端。
定义3. 信息管理系列委员会定义:在管道和过滤器软件体系结构中,每个模块都有一组输入和一组输出。每个模块从它的输入端接收输入数据流,在其内部经过处理后,按照标准的顺序,将结果数据流送到输出端,以达到传递一组完整的计算结果实例的目的。通常情况下,可以通过对输入数据流进行局部变换,并采用渐增式计算方法,在未处理完所有输入数据以前,就可以产生部分计算结果,并将其送到输出端口(类似于流水线结构)。因此,称这种模块为“过滤器“。在这种结构中,各模块之间的连接器充当了数据流的导管,将一个过滤器的输出传到下一个过滤器的输入端。所以,这种连接器称为“管道”。
在[filter模式][1]中有一个典型的例子，其类图如下所示：
![filter模式一个例子的类图][2]
在这个例子中，有两个类比较重要。

 1. AbstractFilter类
 它定义了如何组织过滤器链，主要是如下代码：

    @Override
	public void setNext(Filter next) {
		
		this.next = next;
		
	}
	
	@Override
	public Filter getNext() {
		
		return this.next;
		
	}
	
	@Override
	public Filter getLast() {

		Filter last = this;
		while(last.getNext() != null)
		{
			last = last.getNext();
		}
		return last;
	}
	

 2. FilterChain类
 它给出了一个添加过滤器类和执行过滤器链的接口：

    public void addFilter(Filter filter)
	{
		if(chain == null) chain = filter;
		else chain.getLast().setNext(filter);
	}
	
	public String execute(Order order)
	{
		if(chain != null) return chain.execute(order);
		else return "RUNNING...";
	}
	
整个实现过程相当繁琐，光过滤器的实现，就写了5个类。
下面，我们来看看如何使用函数式编程实现该模式。
首先，Order类是不变的：

    public class Order {
	    private String name;
	    private String contactNumber;
	    private String address;
	    private String depositNumber;
	    private String order;
	    public Order(String name, String contactNumber, String address, String depositNumber, String order) {
		    super();
		    this.name = name;
		    this.contactNumber = contactNumber;
		    this.address = address;
		    this.depositNumber = depositNumber;
		    this.order = order;
	    }
	    public String getName() {
		    return name;
	    }
	    public void setName(String name) {
		    this.name = name;
	    }
	    public String getContactNumber() {
		    return contactNumber;
	    }
	    public void setContactNumber(String contactNumber) {
		    this.contactNumber = contactNumber;
	    }
	    public String getAddress() {
		    return address;
	    }
	    public void setAddress(String address) {
		    this.address = address;
	    }
	    public String getDepositNumber() {
		    return depositNumber;
	    }
	    public void setDepositNumber(String depositNumber) {
		    this.depositNumber = depositNumber;
	    }
	    public String getOrder() {
		    return order;
	    }
	    public void setOrder(String order) {
		    this.order = order;
	    }
}

接着，比较重要的改变是`FilterChain`类，代码如下：

    public class FilterChain {
    private static List<Function<Order, String>> FILTERS = new ArrayList<>();
    public void addFilter(Function<Order, String> filter)
    {
        FILTERS.add(filter);
    }
    public String execute(Order order)
    {
        if (FILTERS.isEmpty()) return "RUNNING...";
        StringBuilder sb = new StringBuilder();
        Collections.reverse(FILTERS);
        for (Function<Order, String> filter : FILTERS)
        {
            sb.append(filter.apply(order));
        }
        return sb.toString();
    }
}

在上述代码中，我们使用`List<Function<Order, String>>`做一个管道，这是因为Stream对象能够做管道对象的任何事情。
管道里面流动的是`Order`对象，每一个管道的处理函数是`Function<Order, String>`，这是因为原始的处理函数`String execute(Order order);`只有一个输入参数和一个输出参数，刚好可以使用`Function`函数来处理。如果有多个输入参数，那么请重新定义一个函数式接口即可。
这样一来，增加过滤器类就编程了增加`Function`函数了：

    public void addFilter(Function<Order, String> filter)
    {
        FILTERS.add(filter);
    }
过滤器链的执行，也就是顺序执行`FILTERS`里面的元素即可：

    for (Function<Order, String> filter : FILTERS)
    {
        sb.append(filter.apply(order));
    }
然后是`FilterManager`类也会有少许改变：

    public class FilterManager {
	    private FilterChain filterChain;
	    public FilterManager(FilterChain filterChain) {
		    super();
		    this.filterChain = filterChain;
	    }
	    public void addFilter(Function<Order, String> filter)
	    {
		    filterChain.addFilter(filter);
	    }
	    public String filterRequest(Order order)
	    {
		    return filterChain.execute(order);
	    }
}

相对简单，这里就不多说了。
最后，我们来看如何使用这个过滤器模式。
首先是添加过滤器：

    FilterManager manager = new FilterManager(new FilterChain());
    manager.addFilter((o -> {
        if(o.getAddress() == null||o.getAddress().isEmpty())
        {
            return "Invalid address!";
        }
        return "";
    }));
    
    manager.addFilter((o) ->{
        if(o.getContactNumber() == null ||
                o.getContactNumber().isEmpty()||
                o.getContactNumber().matches(".*[^\\d]+.*")||
                o.getContactNumber().length() != 11)
        {
            return "Invalid contact number!";
        }
        else return "";
    });
    
    manager.addFilter((o) -> {
        if(o.getDepositNumber() == null || o.getDepositNumber().isEmpty())
        {
            return "invalid deposit number!";
        }
        else return "";
    });
    
    manager.addFilter((o) -> {
        if(o.getName() == null || o.getName().isEmpty() || o.getName().matches(".*[^\\w|\\s]+.*"))
        {
           return "Invalid name!";
        }
        else return "";
    });
    
    manager.addFilter((o) -> {
        if(o.getOrder() == null ||o.getOrder().isEmpty())
        {
            return "Invalid order!";
        }
        else return "";
    });
    
最后是执行过滤器：

    Order order = new Order("Tom","201244","Hangzhou Yuhang","203",null);
    System.out.println(manager.filterRequest(order));
怎么样？这就是一个使用了函数式编程的过滤器模式的完整实现过程，是不是使用了函数式编程，可以大大的减少了类的冗余？


----------
参考文献：[github上关于过滤器模式的例子][3]


  [1]: https://github.com/iluwatar/java-design-patterns/tree/master/intercepting-filter
  [2]: https://raw.githubusercontent.com/iluwatar/java-design-patterns/master/intercepting-filter/etc/intercepting-filter.png
  [3]: https://github.com/iluwatar/java-design-patterns/tree/master/intercepting-filter