# 函数式编程下的模版方法模式

标签（空格分隔）： Java8 函数式编程 模版方法模式

---

模版方法模式是我们非常常用的几种模式之一。
它定义一个算法中的操作框架，而将一些步骤延迟到子类中。使得子类可以不改变算法的结构即可重定义该算法的某些特定步骤。
解决的问题：
 1. 提供代码复用性
 将相同部分的代码放在抽象的父类中，而将不同的代码放入不同的子类中
 2. 实现了反向控制
 通过一个父类调用其子类的操作，通过对子类的具体实现扩展不同的行为，实现了反向控制 & 符合“开闭原则”
下面举出一个例子来说明该模式。
在TCP通讯中，我们通常设计一个通用的数据协议包，这个数据协议包基本上由公用的包头+包体+公用的包尾组成，每一个通讯指令只有包体是不同的，包头和包尾是相同的。而在TCP通讯中，我们需要对这种协议的数据包进行解析。
Java的TCP通讯，我们最常用的是netty框架，下面以netty框架为基础，说说我们如何通过使用模版方法模式来实现对TCP通讯的数据包进行解析。
首先是一个抽象的父类：

    @Slf4j
    public abstract class CashboxBaseEncoder {
        public final void encode(CashboxBaseData baseData, ByteBuf out)
        ｛
            encodeHeader(baseData, out);
            encodeBody(baseData, out);
            encodeTail(baseData, out);
        ｝
        //共10个字节，其中4个字节属于包头，其余type和时间属于包体
        private void encodeHeader(CashboxBaseData baseData, ByteBuf out)
        {
            log.info("开始写数据包头...");
            out.writeByte(baseData.getBeginDLE());
            out.writeByte(baseData.getStx());
            if (baseData.getType() != ENQ)
            {
                out.writeShortLE(baseData.getLength());
                out.writeByte(baseData.getType().getId());
                out.writeByte(baseData.getSendTime().getMonthValue());
                out.writeByte(baseData.getSendTime().getDayOfMonth());
                out.writeByte(baseData.getSendTime().getHour());
                out.writeByte(baseData.getSendTime().getMinute());
                out.writeByte(baseData.getSendTime().getSecond());
            }
        }
        //共7个字节
        private void encodeTail(CashboxBaseData baseData, ByteBuf out)
        {
            log.info("开始写数据包尾...");
            out.writeMediumLE(baseData.getId());
            if (baseData.getType() != ENQ) {
                out.writeByte(baseData.getEndDLE());
                out.writeByte(baseData.getEtx());
                out.writeShortLE(getCheckCode(out, baseData.getLength()));
            }
        }
        protected abstract void encodeBody(CashboxBaseData baseData, ByteBuf out);
    ｝

这就是一个典型的模版方法模式的父类。
我们有一个心跳协议的子类， 大概是这样实现的：

    @Slf4j
    public class CashboxHeartbeatEncoder extends CashboxBaseEncoder ｛
         @Override
         public void encodeBody(CashboxBaseData baseData, ByteBuf out)
         {
            log.info("进入心跳包写入...");
            log.debug("心跳包内容："+baseData.toString());
            out.writeByte(baseData.getSendTime().getMonthValue());
            out.writeByte(baseData.getSendTime().getDayOfMonth());
            out.writeByte(baseData.getSendTime().getHour());
            out.writeByte(baseData.getSendTime().getMinute());
            out.writeByte(baseData.getSendTime().getSecond());
         }
    ｝

同时，我们有一个版本查询协议的子类，是这样实现的：

    @Slf4j
    public class CashboxVersionCommandEncoder extends CashboxBaseEncoder ｛
        @Override
        public void encodeBody(CashboxBaseData baseData, ByteBuf out)
        {
            log.info("进入查看版本号命令写入...");
            CashboxVersionReadCommandData data = (CashboxVersionReadCommandData)baseData;
            out.writeByte(data.getModuleType());
            out.writeByte(data.getSubModuleType());
        ｝
    ｝

一个解析心跳的客户端代码：

    CashboxBaseEncoder encoder = new CashboxHeartbeatEncoder();
    encoder.encode(baseData, out);
    
上述代码的类图如下所示：
![一个实现了模版方法模式的数据协议包解析类图][1]
上述的功能，用函数式编程实现，显得更为简单和合理。
首先，实现一个函数式接口：

    @FunctionalInterface
    public interface BodyEncoder
    {
        public void encodeBody(CashboxBaseData baseData, ByteBuf out);
    }

上面的“CashboxBaseEncoder”将被改造成如下的代码：

    @Slf4j
    public class CashboxEncoder {
        public final void encode(CashboxBaseData baseData, ByteBuf out， BodyEncoder bodyEncoder)
        {
            encodeHeader(baseData, out);
            bodyEncoder.encodeBody(baseData, out);
            encodeTail(baseData, out);
        }
        //共10个字节，其中4个字节属于包头，其余type和时间属于包体
        private void encodeHeader(CashboxBaseData baseData, ByteBuf out)
        {
            log.info("开始写数据包头...");
            out.writeByte(baseData.getBeginDLE());
            out.writeByte(baseData.getStx());
            if (baseData.getType() != ENQ)
            {
                out.writeShortLE(baseData.getLength());
                out.writeByte(baseData.getType().getId());
                out.writeByte(baseData.getSendTime().getMonthValue());
                out.writeByte(baseData.getSendTime().getDayOfMonth());
                out.writeByte(baseData.getSendTime().getHour());
                out.writeByte(baseData.getSendTime().getMinute());
                out.writeByte(baseData.getSendTime().getSecond());
            }
        }
        //共7个字节
        private void encodeTail(CashboxBaseData baseData, ByteBuf out)
        {
            log.info("开始写数据包尾...");
            out.writeMediumLE(baseData.getId());
            if (baseData.getType() != ENQ) {
                out.writeByte(baseData.getEndDLE());
                out.writeByte(baseData.getEtx());
                out.writeShortLE(getCheckCode(out, baseData.getLength()));
            }
        }
    ｝

然后，就可以写客户端代码了。
一个解析心跳的客户端代码如下：

    CashboxEncoder encoder = new CashboxEncoder();
    encoder.encode(baseData1, out1, (baseData, out) -> {
        log.info("进入心跳包写入...");
        log.debug("心跳包内容："+baseData.toString());
        out.writeByte(baseData.getSendTime().getMonthValue());
        out.writeByte(baseData.getSendTime().getDayOfMonth());
        out.writeByte(baseData.getSendTime().getHour());
        out.writeByte(baseData.getSendTime().getMinute());
        out.writeByte(baseData.getSendTime().getSecond());
    });

可以看到，由函数式编程实现的模版方法模式，省了很多子类，节省了大量的代码，同时，也带来了相当大的灵活性。
值得使用函数式编程来改造这样的设计模式！


  [1]: https://thumbnail0.baidupcs.com/thumbnail/3b1223bf6b3236e8b454e08e58bb57c4?fid=2048635255-250528-91210563668197&time=1520838000&rt=sh&sign=FDTAER-DCb740ccc5511e5e8fedcff06b081203-4BamDtkB61GtTP1fwrgmRAwC8ec=&expires=8h&chkv=0&chkbd=0&chkpc=&dp-logid=1638165112128014097&dp-callid=0&size=c710_u400&quality=100&vuk=-&ft=video