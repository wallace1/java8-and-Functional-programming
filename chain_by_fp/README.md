# 函数式编程下的责任链模式

标签（空格分隔）： Java 函数式编程 责任链模式

---

责任链模式是一种设计模式。在责任链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任。
比如，一个请假申请就可以很好的描述这种模式：
![基于请假审批的责任链描述][1]
在“iluwatar/java-design-patterns”上有一个很好的例子：
[责任链模式][2]
在这个例子中，我们来看看是怎么传递责任的。
首先是责任链条的某一个链：

    public class OrcOfficer extends RequestHandler {
        public OrcOfficer(RequestHandler handler) {
	       super(handler);
	    }
	   @Override
	   public void handleRequest(Request req) {
	       if (req.getRequestType().equals(RequestType.TORTURE_PRISONER)) {
	           printHandling(req);
	           req.markHandled();
	       } else {
	           super.handleRequest(req);
	       ｝
	   ｝
	   @Override
	   public String toString() {
	       return "Orc officer";
	   }
    ｝
在这个类里，如果要初始化这个类，必须在构造器里给定一个下家`public OrcOfficer(RequestHandler handler)`，这就形成了一个链条了。
其次，在处理请求的时候，首先是做判断，如果该它处理的，就处理了，责任链的传递也就结束了；如果不该它处理，则交由父类的方法处理。
那么，我们来看看，父类的`handleRequest`方法里干了什么呢？

    public void handleRequest(Request req) {
	    if (next != null) {
	      next.handleRequest(req);
	    }
	  }
	  
可以看到，该方法首先是问还有没有下家，如果有，则交给下家处理；如果没有，则结束。
完整的例子，请参考[责任链模式][3]。
责任链模式解决问题的思想是十分优雅的，有很好的扩展性，把一个复杂的问题做了简单的分解，使得我们实现起来相当的简单。
不好的地方，是面向对象语言的弱点，必须实现很多的子类来做责任链，造成了类的众多和代码的重复。比如上述例子中，就有`OrcCommander`、`OrcOfficer`和`OrcSoldier`三个子类，这些子类都存在不同程度的代码重复。
我们现在来用函数式编程来解决面向对象的弱点。
首先，`RequestType`和`Request`可以保留：

    public enum RequestType {
        DEFEND_CASTLE, TORTURE_PRISONER, COLLECT_TAX
    ｝
    
    public class Request {
        private final RequestType requestType;
        private final String requestDescription;
        private boolean handled;
        public Request(final RequestType requestType, final String requestDescription) {
            this.requestType = Objects.requireNonNull(requestType);
            this.requestDescription = Objects.requireNonNull(requestDescription);
        ｝
        public String getRequestDescription() {
	        return requestDescription;
	    }
	    public RequestType getRequestType() {
	        return requestType;
	    }
	    public void markHandled() {
	        this.handled = true;
	    }
	    public boolean isHandled() {
	        return this.handled;
	    }
	    @Override
	    public String toString() {
	        return getRequestDescription();
	    }
	｝

RequestHandler类需要做相当的改造：

    public class RequestHandler {
        private static final Logger LOGGER = LoggerFactory.getLogger(RequestHandler.class);
        private RequestHandler next = null;
        public void setNext(RequestHandler next) {
            this.next = next;
        }
    

上面是下家的代码和设置下家的代码。

    private Predicate<Request> predicate;
    public void withPredication(Predicate<Request> predicate)
    {
        this.predicate = predicate;
    }
判断条件，判断是否该本节点处理。

    private Consumer<Request> action;
    public void withAction(Consumer<Request> action)
    {
        this.action = action;
    }
处理方法。
最终对请求的处理：

    public void handleRequest(Request req) throws Exception{
        if (this.predicate == null || this.action == null) throw new Exception("必须先通过withPredication和withAction给定判定条件和执行方法！");
        if (this.predicate.test(req)) this.action.accept(req);
    

如果该本节点处理，则处理。否则：

        else if (next != null) {
            next.handleRequest(req);
        }
    ｝
扔给下家处理。
下面，我们来看看`OrcKing`类如何构建责任链：

    public class OrcKingKing {
        private static final Logger LOGGER = LoggerFactory.getLogger(OrcKingKing.class);
        RequestHandler chain;
        public OrcKingKing() {
	        buildChain();
	    }
	    private void buildChain() {
	        RequestHandler orcCommander = new RequestHandler();
	        orcCommander.withPredication(comparedWith.apply(DEFEND_CASTLE));
	        orcCommander.withAction(actionWith.apply("OrcCommander"));
	        RequestHandler orcOfficer = new RequestHandler();
	        orcOfficer.withPredication(comparedWith.apply(TORTURE_PRISONER));
	        orcOfficer.withAction(actionWith.apply("OrcOffice"));
	        orcCommander.setNext(orcOfficer);
	        RequestHandler orcSoldier = new RequestHandler();
	        orcSoldier.withPredication(comparedWith.apply(COLLECT_TAX));
	        orcSoldier.withAction(actionWith.apply("OrcSoldier"));
	        orcOfficer.setNext(orcSoldier);
	        
	        this.chain = orcCommander;
	   ｝
	   public void makeRequest(Request req) throws Exception{
	       chain.handleRequest(req);
	   ｝
	   private Function<RequestType, Predicate<Request>> comparedWith = type -> req -> req.getRequestType().equals(type);
	   private Function<String, Consumer<Request>> actionWith = objectType -> req -> {
          LOGGER.info("{} handling request \"{}\"", objectType, req);
          req.markHandled();
      };
  ｝

这样，就完成了责任链的构建。
在函数式编程里，函数是可以作为参数传递的。我们通过把方法传递给某一个对象，达到了多态的目的，就不需要做子类继承了。

    RequestHandler orcCommander = new RequestHandler();
orcCommander是一个对象，通过下面的代码传递了两个运行时的函数：

    orcCommander.withPredication(comparedWith.apply(DEFEND_CASTLE));
	orcCommander.withAction(actionWith.apply("OrcCommander"));
	
通过函数式编程，函数可以不断的抽象，就像下面的代码：

    private Function<RequestType, Predicate<Request>> comparedWith = type -> req -> req.getRequestType().equals(type);
    private Function<String, Consumer<Request>> actionWith = objectType -> req -> {
          LOGGER.info("{} handling request \"{}\"", objectType, req);
          req.markHandled();
      };
  

通过这种方式传递了函数，就避免了子类众多的问题，因为在面向对象的语言中，函数必须依附在类身上，不能独立存在。而函数式编程中，函数是可以独立存在的，它也可以像对象一样，在运行时进行传递。
最后，我们就可以使用这个责任链了：

    OrcKingKing king = new OrcKingKing();
    king.makeRequest(new Request(RequestType.DEFEND_CASTLE, "defend castle"));
    king.makeRequest(new Request(RequestType.TORTURE_PRISONER, "torture prisoner"));
    king.makeRequest(new Request(RequestType.COLLECT_TAX, "collect tax"));

看看，函数式编程的思路，是不是很有意思！


----------
参考文献：[chain][4]


  [1]: https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=89a978df9fcad1c8c4b6f4751e570c6c/a8014c086e061d95b39c5fc871f40ad163d9ca6c.jpg
  [2]: https://github.com/iluwatar/java-design-patterns/tree/master/chain
  [3]: https://github.com/iluwatar/java-design-patterns/tree/master/chain
  [4]: https://github.com/iluwatar/java-design-patterns/tree/master/chain