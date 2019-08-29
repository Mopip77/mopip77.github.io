---
layout: post
title: Collector详解
tags:
- java
- jdk8
---

## Collector
collector是一个接口,它是一个可变的汇聚操作,将输入元素累积到一个可变的结果容器中;它会在所有元素都处理完毕后,将累积的结果转换为一个最终的表示(这是一个可选操作);它支持串行与并行两种方式执行。
**Collectors**提供了常见的汇聚操作,毫无疑问可以自己实现collector

Collector需要满足两个特性:identity(同一性) 和 associativity(结合性)
a. identity: 容器A与一个空容器合并,结果还是A
b. associativity: 分割的并行计算和串行的计算结果等价(类似mapreduce)

同时Collector还需要满足如下要求:
Collector源码 line:109~136

collector<T, A, R>(T传入类型, A中间类型, R结果类型)接受参数:
1. supplier<A>
2. accumulator<A, T>
3. combiner<A, A>
4. finisher<A, R> (重载版本可以没有)
5. Characteristics  

其中前三个和collect()接受的参数相同, 但是由于通过中间类型运算(常使用<?> often hidden as an implementation detail),所以可能需要finisher做一个类型转换

```java
enum Characteristics {
        CONCURRENT, // 支持并行操作同一个中间结果容器,在《~stream详解》中详细说明
        UNORDERED, //无需保证结果和输入流原有的顺序
        IDENTITY_FINISH  //无需从中间结果转换成最终结果即<A,R> => A == R
        }
```

### 实现自己的Collector
```java
public class MySetCollector<T> implements Collector<T, Set<T>, Set<T>> {
    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked");
        return HashSet::new;
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked");
        // 不能是HashSet::add
        return (set, e) -> {
            System.out.println("set:" + set + " e:" + e);
            set.add(e);
        };
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked");
        return (left, right) -> {
            left.addAll(right);
            return left;
        };
    }

    @Override
    public Function<Set<T>, Set<T>> finisher() {
        System.out.println("finisher invoked");
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked");
        return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH, Characteristics.UNORDERED));
    }
}
```

调用代码和结果如下
```java
List<String> words = Arrays.asList("fsdf", "cvxzcvd", "adsa", "wccd");
        Set<String> collect = words.stream().collect(new MySetCollector<>());
        System.out.println(collect);
    
//结果
supplier invoked
accumulator invoked
combiner invoked
characteristics invoked
set:[] e:fsdf
set:[fsdf] e:cvxzcvd
set:[cvxzcvd, fsdf] e:adsa
set:[cvxzcvd, adsa, fsdf] e:wccd
characteristics invoked
[cvxzcvd, wccd, adsa, fsdf]
```
这里有几个值的注意的点:
1. 执行顺序
首先在ReduceOps#makeRef中获取supplier,accumulator,combiner,以及检查characteristics中是否有UNORDERED
然后分别执行supplier,accumulator,combiner
最后ReferencePipeline#collect检查characteristics中是否有IDENTITY_FINISH
```java
return collector.characteristics().contains(Collector.Characteristics.IDENTITY_FINISH)
               ? (R) container
               : collector.finisher().apply(container);
```
可见如果有IDENTITY_FINISH会直接把中间类型强转成结果类型,所以代码需要**保证两者类型一致**

2. 在accumulator中有一行注释,不能使用HashSet<T>::add
理论上可以直接使用实现了Set的HashSet的方法,但是由于supplier也可能返回一个其他实现了Set的类(例如TreeSet),所以不能直接使用HashSet<T>::add

### Collectors几个特殊的收集器

| method | 功能 |
| --- | --- |
| mapping | 在accumulator前先对元素mapping一次 |
| collectingAndThen | 对collector的结果再使用一个finisher|
| summingInt | 里面使用的累加元素为int[1],而不是直接一个int,这是因为int类型为值传递传入accumulator无法变更值本身,而int[1]为引用类型, 很多其他地方也是这样使用|
| partitioningBy | 这个实现了一个Partition的静态内部类,继承的map|
| groupingBy | 是最复杂的实现,可以参考一下 |

1. groupingByConcurrent在并发下性能更好(对于groupingBy而言), 同时如果downstream不支持CONCURRENT特性函数内部也做了synchroized