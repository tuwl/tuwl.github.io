---
title: 多线程下的 DateFormat
date: 2018-03-21 20:20:48
tags:
---

[原文链接][1]


  [1]: http://fahdshariff.blogspot.com/2010/08/dateformat-with-multiple-threads.html

DateFormat 类不是线程安全的。官方文档声明道：“时间格式化不是同步的，推荐为每一个线程创建分离的格式化实例。如果多线程同时访问一个格式化实例，它必须在外部已被同步。”
下面的代码将展示一般在单线程的情况下是如何使用 DateFormat 去转化一个 String 变成一个 Date 的。通过声明 DateFormat 为一个实例变量并多次使用它是一种高效的做法，那样系统就不需要多次获取本地语言和国家公约的信息。
   

     public class DateFormatTest {
          private final DateFormat format = new SimpleDateFormat("yyyyMMdd");
            public Date convert(String source) throws ParseException{
                Date d = format.parse(source);
                return d;
            }
     }

这段代码不是线程安全的。我们可以用多线程调用这个方法来测试，在下面的调用代码中，我创建了一个有两个线程的线程池并提交给它5个 Date 转化任务。然后我检查结果。

    final DateFormatTest t = new DateFormatTest();
    Callable<Date> task = new Callable<Date>(){
        public Date call() throws Exception {
            return t.convert("20100811");
        }
    };
 
    //lets try 2 threads only
    ExecutorService exec = Executors.newFixedThreadPool(2);
    List<Future<Date>> results = new ArrayList<Future<Date>>();
 
    //perform 5 date conversions
    for(int i = 0 ; i < 5 ; i++){
        results.add(exec.submit(task));
    }
    exec.shutdown();
 
    //look at the results
    for(Future<Date> result : results){
        System.out.println(result.get());
    }

程序运行的结果是不可预料的，有时候他会输出正确的日期，有时候有错误的（e.g. Sat Jul 31 00:00:00 BST 2012!）其他时候它会抛出一个异常 NumberFormatException 。
这有几个例子：

    Caused by: java.lang.NumberFormatException: For input string: ""
        at java.lang.NumberFormatException.forInputString(NumberFormatException.java:48)
        at java.lang.Long.parseLong(Long.java:431)
        at java.lang.Long.parseLong(Long.java:468)
        at java.text.DigitList.getLong(DigitList.java:177)
        at java.text.DecimalFormat.parse(DecimalFormat.java:1298)
        at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1589)

    Caused by: java.lang.NumberFormatException: For input string: ".10201E.102014E4"
        at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1224)
        at java.lang.Double.parseDouble(Double.java:510)
        at java.text.DigitList.getDouble(DigitList.java:151)
        at java.text.DecimalFormat.parse(DecimalFormat.java:1303)
        at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1589)

    Caused by: java.lang.NumberFormatException: multiple points
        at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:1084)
        at java.lang.Double.parseDouble(Double.java:510)
        at java.text.DigitList.getDouble(DigitList.java:151)
        at java.text.DecimalFormat.parse(DecimalFormat.java:1303)
        at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1936)
        at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1312)

错误结果

    Sat Oct 22 00:00:00 BST 2011
    Thu Jan 22 00:00:00 GMT 1970
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Thu Oct 22 00:00:00 GMT 1970
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010

正确结果

    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010
    Fri Oct 22 00:00:00 BST 2010

如何同时使用 DateFormat ？
有多种方法可以以线程安全的方式使用 DateFormat ：
1. Synchronization
使这段代码线程安全的最简单的方法就是在解析 Date 字符串之前在 DateFormat 对象上取得一个锁。这种方法一次只有一个线程可以访问对象，其他线程必须等待。

        public Date convert(String source)
                        throws ParseException{
          synchronized (format) {
            Date d = format.parse(source);
            return d;
          }
        }
    
2. ThreadLocals
另一种方法是使用 ThreadLocal 变量去保持 DateFormat 对象，这意味着每一个线程将拥有自己的 copy 不需要等待其他的线程释放。这通常比前一种方法更加高效。

        public class DateFormatTest {
          private static final ThreadLocal<DateFormat> df
                         = new ThreadLocal<DateFormat>(){
            @Override
            protected DateFormat initialValue() {
                return new SimpleDateFormat("yyyyMMdd");
            }
          };
 
          public Date convert(String source)
                             throws ParseException{
            Date d = df.get().parse(source);
            return d;
          }
        }

3. Joda-Time
Joda-Time 是一个伟大的，开源的， JDK 的 Date 和 Calendar API 的替代品。它的 DateTimeFormat 是线程安全和不可变的。

        import org.joda.time.DateTime;
        import org.joda.time.format.DateTimeFormat;
        import org.joda.time.format.DateTimeFormatter;
        import java.util.Date;
 
        public class DateFormatTest {
 
          private final DateTimeFormatter fmt =
               DateTimeFormat.forPattern("yyyyMMdd");
 
          public Date convert(String source){
            DateTime d = fmt.parseDateTime(source);
            return d.toDate();
          }
        }

