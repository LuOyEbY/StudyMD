# Google Guava工具集的学习

##### Guava简介

```txt
		Guava工程包含了若干被Google的Java项目广泛并依赖的核心库。例如：集合，缓存，原生类型支持，并发库，通用注解，字符串处理，I/O等等。
```

##### 使用和避免null

```java
1.JAVA8中的Optional<T>表明可能为null的T类型引用。
2.Optional实例可能包含非null的引用（引用存在），也可能什么都不存在。

public class TestOptional {

    @Test
    public void testOptional() throws  RuntimeException{
        /**
         * 三种创建Optional对象的方式
         */
        //1.创建空的Optional对象
        Optional<Object> empty = Optional.empty();

        //2.创建任意类型的Optional对象非null值
        Optional<Object> o = Optional.of(1);

        //3.使用任意值包括null值
        Optional<Object> o1 = Optional.ofNullable(2);

        /**
         * 判断引用是否缺失的方法
         * 底层实现为 != ，不建议使用
         */
        boolean present = o1.isPresent();

        /**
         * 当Optional引用存在是执行括号中的方法
         * 类似的方法 map  filter  flatMap
         */
        o1.ifPresent(System.out::println);

        /**
         * 当Optional引用缺失时执行括号中的方法
         *
         */
        o1.orElse("引用缺失");
        o1.orElseGet(() -> {
            //自定义引用缺失的返回值
            return "自定义引用缺失";
        });
        o1.orElseThrow(() -> {
            throw new RuntimeException("引用缺失异常");
        });
    }
		//使用Optional和stream连用
    public void streamTest(List<String> list){
        Optional.ofNullable(list)
                .map(List::stream)
                .orElseGet(Stream::empty)
                .forEach(System.out::println);
    }

}
```

##### 不可变对象

```java
创建对象的不可变拷贝是一项很好的防御性编程技巧。
Guava为所有的JDK标准集合类型和Guava心机和类型都提供了简单易用的不可变版本。

不可变对象的优点
1.当对象被不可信的库调用时，不可变形式是安全的
2.不可变对象被多个线程调用时，不存竞态条件问题
3.不可变集合不需要考虑变化，因此可以节省时间和空间
4.不可变对象因为固定不变，可以作为常量来安全使用
	/**
   * JAVA8本身带有的创建不可变集合的方法
   */
   List<Integer> list2 = Collections.unmodifiableList(list);
  /**
   * Guava创造不可变集合的三种方式
   */
  //通过已经存在的集合创建
  ImmutableSet<Integer> integers = ImmutableSet.copyOf(list);
	//通过初始化值的方式创建
	ImmutableSet<Integer> of = ImmutableSet.of(1, 2, 3, 3);
	//以builder的方式进行创建
	ImmutableSet<Object> build = ImmutableSet.builder()
                                          .add(1)
                                          .addAll(Sets.newHashSet(2, 3))
                                          .add(5)
                                          .build();

```

##### 新的集合类型

```java
Guava引入了很多JDK没有，但是明显有用的新集合类型。这些新类型是为了和JDK集合框架共存，而没有往JDK集合抽象中硬塞其他概念。
1.multiset（无序可重复的集合）融合了set和list的特性。
2.多种实现类（HashMultiset,TreeMultiset,LinkedHashMultiset,ConcurrentHashMultiset,ImmutableMultiset）
3.方法：
	a:add(E)==添加元素
	b:iterator()==迭代器，包含所有的元素
	c:size()==返回所有元素的总个数(包括重复元素)
	d:count(object)==返回给定元素的计数
	e:entrySet()==返回Set<Multiset.Entry<E>>和Map.entrySet类似
	f:elementSet()==返回所有不重复元素的Set<E>和Map.keySet类似

4.例子统计一片文章中文字出现次数的功能
	public class MultisetTest {

    private static String text = "黄金殿里，烛影双龙戏。劝得官家真个醉，进酒犹呼万岁。" +
                                 "折旋舞彻《伊州》。君恩与整搔头。一夜御前宣住，六宫多少人愁。";

    @Test
    public void multisetTest(){

        Multiset<Character> multiset = HashMultiset.create();

        char[] chars = text.toCharArray();

        Chars.asList(chars).stream().forEach((item) ->
            multiset.add(item)
        );

        System.out.println(multiset.size());
        System.out.println(multiset.count('少'));
    }

}
```

##### 集合工具类(Sets)

```java
Sets工具类，可以计算两个set的并集，交集，差集，相对差集，子集，笛卡尔积。
public class SetsTest {


    private static Set set1 = Sets.newHashSet(1,2);

    private static Set set2 = Sets.newHashSet(4,5);

    //获取两个集合的并集
    @Test
    public void union(){
        Sets.SetView union = Sets.union(set1, set2);
        System.out.println( union);
    }

    //获取两个集合的差集元素属于A不属于B
    @Test
    public void difference(){
        Sets.SetView difference = Sets.difference(set1, set2);
        System.out.println(difference);
    }

    //获取两个集合的相对差集，属于A不属于B，属于B不属于A
    @Test
    public void symetricDifference(){
        Sets.SetView setView = Sets.symmetricDifference(set1, set2);
        System.out.println(setView);
    }

    //获取单个集合所有子集
    @Test
    public void powerSet(){
        Set<Set<Integer>> set = Sets.powerSet(set1);
        System.out.println(JSON.toJSONString(set ,true));

    }

    //获取两个集合的笛卡尔积
    @Test
    public void createsianProduct(){
        Set<List<Integer>> set = Sets.cartesianProduct(set1,set2);
        System.out.println(set);
    }

}


```

##### 集合工具类(Lists)

```java
Lists工具类，可以进行反转，拆分。
public class ListsTest {

    private static List  list = Lists.newArrayList(1,2,3,4,5,6,7);

    //将一个集合按指定数量进行分区
    @Test
    public void partition(){
        List<List<Integer>> partition = Lists.partition(list, 3);
        System.out.println(JSON.toJSONString(partition));

    }

    //将一个集合进行反转
    @Test
    public void reverse(){
        List reverse = Lists.reverse(list);
        System.out.println(JSON.toJSONString(reverse));
    }

}

```



##### IO流（Source）和汇（Sink）

```java
1.对字节流和字符流提供的方法
 	a:ByteStreams==> 提供对InputStream/OutputStream的操作
 	b:CharStreams==> 提供对Reader/Writer的操作
2.对源和汇的抽象
  a:源是可读的==ByteSource/CharSource
  b:汇是可写的==ByteSink/CharSink
    
3.例子：使用源和汇进行文件操作
    public class IOTest {

    @Test
    public void copyFile(){

        /**
         * 创建对应的源和汇
         *
         */

        CharSource charSource = Files.asCharSource(new File("/Users/Emotibot/Desktop/StudyDaily/src/main/java/com/luoye/studydaily/codingFirst/SD_four/IOTest.java"), Charsets.UTF_8);
        CharSink charSink = Files.asCharSink(new File("src/test/test.txt"), Charsets.UTF_8);

        try {
            //拷贝文件
            charSource.copyTo(charSink);
            
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
 }   
```

