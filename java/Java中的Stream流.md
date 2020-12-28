# Java中的Stream流

Stream流在早在Java8中就已推出，但一直保持着听说过，没用过的状态。现在根据掘金的文章  [ 一文带你玩转 Java8 Stream 流，从此操作集合 So Easy](https://juejin.cn/post/6844903830254010381)
对Java中的Stream流做一个系统的学习。  

## 一、Stream流是怎么工作的
流包含着一系列元素的集合，我们可以对其做不同类型的操作，使用这些元素来执行计算，用代码表示就是  
```java
List<String> myList = Arrays.asList("a1", "a2", "b1", "b2", "c3");
myList
    .stream()                           // (1)创建一个流
    .filter(s -> s.startsWith("a"))     // (2)使用Lambda语法筛选出所有以a开头的元素
    .map(String::toUpperCase)           // (3)转换成大写
    .sorted()                           // (4)进行排序
    .forEach(System.out::println);      // (5)for循环打印
```
在上文的代码中，(2)(3)(4)属于**中间操作**，而(5)属于**终端操作**。  
* 中间操作：中间操作会再次返回一个流，因此可以链接多个中间操作，不用加分号可以直接进行相连。
* 终端操作：对流操作的一个结束动作，一般返回一个void或者一个非流的结果，foreach便是一个终结操作。  

流类似于流水线式操作并且大部分操作都支持lambda表达式作为参数。

## 二、不同类型的Stream流
可以从各种数据源中创建Stream流，其中以Collection集合最为常见，如List和Set等。  
流又分顺序流与并行流，先从顺序流开始：  
```java
Arrays.asList("a1","a2","a3")  
                .stream()                           // 创建流       
                .findFirst()                        // 找到第一个元素
                .ifPresent(System.out::println);    // 若是存在，则输出
```
使用asList()方法创建一个集合，这时不用再刻意创建一个新的集合来接收，在原有集合的基础上可以直接使用stream()方法来获取创建一个流。也可以使用Stream.of()来从一堆对象中直接创建Stream流。
```java
Stream.of("a1","a2","c3")
                .findFirst()
                .ifPresent(System.out::println);
```  
除了常规对象流之外，Java8还附带了一些流用于处理原始数据类型的int，long和double，在其中，IntStreams.range()方法还可以取代常规的fori循环：
```java
IntStream.range(1,4)
                .forEach(System.out::println);  // 相当于for (int i = 1, i < 4; i++) {}
```
若是有将常规对象转换为原始类型流的需求，可以使用mapToInt(),mapToLong()和mapToDouble()
```java
Stream.of("a1","a2","a3")
                .map(s -> s.substring(1))           // 对每个字符串元素从下标1开始截取
                .mapToInt(Integer::parseInt)        // 转为Int类型
                .max()                              // 取最大值
                .ifPresent(System.out::println);    // 若存在则输出
// 3
```  
若是要将原始数据类型转换为对象流，可以使用mapToObj()来实现。
```java
IntStream.range(1,4)
                .mapToObj(i -> "a" + i)
                .forEach(System.out::println);              
// a1
// a2
// a3
```  

## 三、Stream流的处理顺序
中间操作有个重要的特性——**延迟性**。
```java
Stream.of("c1","c3","b4","g5")
                .filter(s -> {
                    System.out.println("filter: "+ s);
                    return true;
                });
```  
以上代码并不会依次打印流中所有的元素，因为当且仅当**存在终端操作时**，中间操作才会执行。  
对上述代码进行修改：
```java
Stream.of("c1","c3","b4","g5")
        .filter(s -> {
            System.out.println("filter: "+ s);
            return true;
        })
        .forEach(s -> System.out.println("forEach: "+ s));
```  
输出
```java
filter: c1
forEach: c1
filter: c3
forEach: c3
filter: b4
forEach: b4
filter: g5
forEach: g5
```
以上的输出显示，流并不是在filter中所有元素打印，再在forEach中打印所有元素，而是随着链条垂直移动，当Stream处理第一个"c1"元素时，会顺着流的顺序执行完所有操作，才会继续执行第二个元素"c3"。  
  
这主要是出于性能方面的考虑，这样设计可以减少对每个元素的实际操作数，查看以下代码：
```java
Stream.of("e6","a1","a2","b3","c4","d5")
                .map(s -> {
                    System.out.println("map: "+ s);
                    return s.toUpperCase();
                })
                .anyMatch(s -> {
                    System.out.println("anyMatch: "+ s);
                    return s.startsWith("A");
                });
// map: e6
// anyMatch: E6
// map: a1
// anyMatch: A1
```  
代码中anyMatch()表示一旦遇到以'A'为前缀的元素就停止循环，从"e6"开始垂直执行一遍，循环到"a1"，返回为true，结束代码。  
因为数据流的链式调用时垂直执行的，map在这里只需要执行两次，若是水平执行，map会先将所有元素都遍历一遍，降低了效率。
  
## 四、中间操作的顺序设计
以下代码由中间操作map和filter，终端操作forEach祖成：
```java
Stream.of("d2","a2","b1","b3","c")
                .map(s -> {
                    System.out.println("map: "+s);
                    return s.toUpperCase();
                })
                .filter(s -> {
                    System.out.println("filter: "+s);
                    return s.startsWith("A");
                })
                .forEach(s -> System.out.println("forEach: "+ s));
// map: d2
// filter: D2
// map: a2
// filter: A2
// forEach: A2
// map: b1
// filter: B1
// map: b3
// filter: B3
// map: c
// filter: C
```  
在上述代码中，map和filter都会先执行5次，forEach只会调用一次，因为只有"a2"满足条件。  
若改变map与filter的顺序，可以大大减少执行的次数：
```java
Stream.of("d2","a2","b1","b3","c")
                .filter(s -> {
                    System.out.println("filter: "+s);
                    return s.startsWith("a");
                })
                .map(s -> {
                    System.out.println("map: "+s);
                    return s.toUpperCase();
                })
                .forEach(s -> System.out.println("forEach: "+ s));
// filter: d2
// filter: a2
// map: a2
// forEach: A2
// filter: b1
// filter: b3
// filter: c      
```  
filter()筛选出以'a'开头的元素，若不符合，则不会进行后续操作，改变了Stream()流方法的执行顺序，大大减少了不必要的代码执行。         

## 五、数据流复用问题
Stream流不能被复用，一旦执行了任何终端操作，流就会被关闭。
```java
Stream<String> stream = Stream.of("d2", "d3", "b1", "c4")
                .filter(s -> s.startsWith("d"));
stream.anyMatch(s -> true);     // 可以执行
stream.noneMatch(s -> true);    // 报错
```  
为了克服这个限制，可以为我们想要执行的每个终端操作创建一个新的流链，例如，可以使用Supplier来包装流，通过get()方法来构建一个新的Strean流。
```java
Supplier<Stream<String>> streamSupplier = () -> Stream.of("d2", "d3", "b1", "c4")
                .filter(s -> s.startsWith("d"));
streamSupplier.get().anyMatch(s -> true);
streamSupplier.get().noneMatch(s -> true);
```  
通过构造一个新的流，可以避开流不能被复用的限制。

## 六、Stream流高级操作
Stream不仅有filter，map等中间操作，还支持一些更复杂的例如collect，flatMap和reduce操作。  
以下代码均使用List< Person >进行演示：
```java
class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return name;
    }
}

// 构建一个 Person 集合
List<Person> persons = Arrays.asList(
        new Person("Max", 18),
        new Person("Peter", 23),
        new Person("Pamela", 23),
        new Person("David", 12)
);
```  
### **6.1 Collect**
**collect**将流中的元素转变成另一个不同的对象，例如一个List、Set或Map。collect接受入参为Collector（收集器），有四个不同的操作组成：供应器（supplier）、累加器（accumulator）、组合器（combiner）和终止器（finisher）。  
  
但这些大多数情况下不需要自己去实现收集器，Java通过Collectors类内置了各种常用的收集器，可以直接拿来用。
```java
List<Person> p1 = persons.stream()                      // 创建一个流
                .filter(p -> p.name.startsWith("P"))    // 筛选出以"P"开头的元素
                .collect(Collectors.toList());          // 创建一个新的List
System.out.println(p1);

// [Peter, Pamela]
```  
上述代码使用Collectors中的toList()方法从原有的List中根据过滤方法创建一个新的List，若是Set对象，则使用Collectors。toSet()方法。  
  
* Collectors.groupingBy()可以对流对象进行分组。  
```java
Map<Integer, List<Person>> personByAge = persons.stream()
        .collect(Collectors.groupingBy(p -> p.age));        // 根据age进行分组并创建一个新的对象

personByAge.forEach((age,p) ->
        System.out.format("age %s: %s\n",age, p)
);

// age 18: [Max]
// age 23: [Peter, Pamela]
// age 12: [David]
```  
* 计算所有人平均年龄  
```java
Double averageAge = persons
    .stream()
    .collect(Collectors.averagingInt(p -> p.age)); // 聚合出平均年龄

System.out.println(averageAge);     // 19.0
```  
* 计算最小年龄、最大年龄、平均年龄、总和以及总数量。  
```java
IntSummaryStatistics ageSummary =
    persons
        .stream()
        .collect(Collectors.summarizingInt(p -> p.age)); // 生成摘要统计

System.out.println(ageSummary);
// IntSummaryStatistics{count=4, sum=76, min=12, average=19.000000, max=23}
```
* 将所有人名连接成一个字符串。
```java
String phrase = persons
    .stream()
    .filter(p -> p.age >= 18) // 过滤出年龄大于等于18的
    .map(p -> p.name) // 提取名字
    .collect(Collectors.joining(" and ", "In Germany ", " are of legal age.")); // 以 In Germany 开头，and 连接各元素，再以 are of legal age. 结束

System.out.println(phrase);
// In Germany Max and Peter and Pamela are of legal age.
```  
对于如何将流转换为 Map集合，必须指定 Map 的键和值。这里需要注意，Map 的键必须是唯一的，否则会抛出IllegalStateException 异常。  
  
可以选择传递一个合并函数作为额外的参数来避免发生这个异常:
```java
Map<Integer, String> map = persons
    .stream()
    .collect(Collectors.toMap(
        p -> p.age,
        p -> p.name,
        (name1, name2) -> name1 + ";" + name2)); // 对于同样 key 的，将值拼接

System.out.println(map);
// {18=Max, 23=Peter;Pamela, 12=David}
```  
### **6.2 FlatMap**
Map只能将对象映射到单个对象，若想将一个对象转换为多个其他对象或者根本不想做转换操作时，可以使用flatMap。  
FlatMap能够将流的每个元素转换为其他对象的类，因此，每个对象可以被转换为零至多个其他对象，并以流的方式返回，这些流的内容会放入flatMap返回的流中。
```java
// 具体例子后续补充
```  
### **6.3 Reducce**
规约操作可以将流的所有元素组合成一个结果，Java8支持三种不同的reduce方法。
1. 将流中的元素规约成流中的一个元素
```java
persons
    .stream()
    .reduce((p1, p2) -> p1.age > p2.age ? p1 : p2)
    .ifPresent(System.out::println);    // Pamela
```  
以上代码筛选出年龄最大的那个人。  
reduce方法接受BinaryOperator积累函数。该函数实际上是两个操作数类型相同的BiFunction。BiFunction功能和Function一样，但是它接受两个参数。示例代码中，我们比较两个人的年龄，来返回年龄较大的人。  
  
2. reduce方法接受标识值和BinaryOperator累加器。此方法可用于构造一个新的 Person，其中包含来自流中所有其他人的聚合名称和年龄：
```java
Person result =
    persons
        .stream()
        .reduce(new Person("", 0), (p1, p2) -> {
            p1.age += p2.age;
            p1.name += p2.name;
            return p1;
        });

System.out.format("name=%s; age=%s", result.name, result.age);
// name=MaxPeterPamelaDavid; age=76
```  
3. reduce方法接受三个参数：标识值，BiFunction累加器和类型的组合器函数BinaryOperator。由于初始值的类型不一定为Person，可以使用这个归约函数来计算所有人的年龄总和：
```java
Integer ageSum = persons
    .stream()
    .reduce(0, (sum, p) -> sum += p.age, (sum1, sum2) -> sum1 + sum2);

System.out.println(ageSum);  // 76
```  

## 七、并行流
流是可以并行执行的，当流中存在大量元素时，可以显著提升性能。  
  
并行流底层使用的ForkJoinPool, 它由ForkJoinPool.commonPool()方法提供。底层线程池的大小最多为五个 - 具体取决于 CPU 可用核心数：
```java
ForkJoinPool commonPool = ForkJoinPool.commonPool();
System.out.println(commonPool.getParallelism());    // 3
```  
集合支持parallelStream()方法来创建元素的并行流。或者你可以在已存在的数据流上调用中间方法parallel()，将串行流转换为并行流，这也是可以的。
```java
Arrays.asList("a1", "a2", "b1", "c2", "c1")
    .parallelStream()
    .filter(s -> {
        System.out.format("filter: %s [%s]\n",
            s, Thread.currentThread().getName());
        return true;
    })
    .map(s -> {
        System.out.format("map: %s [%s]\n",
            s, Thread.currentThread().getName());
        return s.toUpperCase();
    })
    .forEach(s -> System.out.format("forEach: %s [%s]\n",
        s, Thread.currentThread().getName()));

// filter:  b1 [main]
// filter:  a2 [ForkJoinPool.commonPool-worker-1]
// map:     a2 [ForkJoinPool.commonPool-worker-1]
// filter:  c2 [ForkJoinPool.commonPool-worker-3]
// map:     c2 [ForkJoinPool.commonPool-worker-3]
// filter:  c1 [ForkJoinPool.commonPool-worker-2]
// map:     c1 [ForkJoinPool.commonPool-worker-2]
// forEach: C2 [ForkJoinPool.commonPool-worker-3]
// forEach: A2 [ForkJoinPool.commonPool-worker-1]
// map:     b1 [main]
// forEach: B1 [main]
// filter:  a1 [ForkJoinPool.commonPool-worker-3]
// map:     a1 [ForkJoinPool.commonPool-worker-3]
// forEach: A1 [ForkJoinPool.commonPool-worker-3]
// forEach: C1 [ForkJoinPool.commonPool-worker-2]
```  
并行流使用了所有的ForkJoinPool中的可用线程来执行流式操作。在持续的运行中，输出结果可能有所不同，因为所使用的特定线程是非特定的。  
  
并行流对含有大量元素的数据流提升性能极大。但是也需要记住并行流的一些操作，例如reduce和collect操作，需要额外的计算（如组合操作），这在串行执行时是并不需要。


