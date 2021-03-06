[Class HashMap<K,V>](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html)
[Class ConcurrentHashMap<K,V>](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html)
Hash table based implementation of the Map interface. This implementation provides all of the optional map operations, and permits null values and the null key. (The HashMap class is roughly equivalent to Hashtable, except that it is unsynchronized and permits nulls.) This class makes no guarantees as to the order of the map; in particular, it does not guarantee that the order will remain constant over time.

> 基于哈希表的Map接口实现。该接口实现提供了所有可选的Map操作，而且允许空value(null value)和空key(null key).(HashMap类大致相当于HashTable，除了它是不同步的和允许为空。)这个类不保证Map的顺序，特别是，它不保证顺序随着时间的推移保持不变。

This implementation provides constant-time performance for the basic operations (get and put), assuming the hash function disperses the elements properly among the buckets. Iteration over collection views requires time proportional to the "capacity" of the HashMap instance (the number of buckets) plus its size (the number of key-value mappings). Thus, it's very important not to set the initial capacity too high (or the load factor too low) if iteration performance is important.

> 假定散列函数将元素适当的分散在桶（buckets）之间，则该实现为其基本操作（get和put）提供了恒定时间的性能。迭代集合视图所需要的时间与HashMap实例的容量（"capacity" ，桶的数量）加上其大小（size，映射键值对的数量）成比例关系。因此，若迭代性能很重要，则不要将初始容量（initial capacity）设置过高（或者负载因子[load factor]设置过低）是非常重要的。

An instance of HashMap has two parameters that affect its performance: initial capacity and load factor. The capacity is the number of buckets in the hash table, and the initial capacity is simply the capacity at the time the hash table is created. The load factor is a measure of how full the hash table is allowed to get before its capacity is automatically increased. When the number of entries in the hash table exceeds the product of the load factor and the current capacity, the hash table is rehashed (that is, internal data structures are rebuilt) so that the hash table has approximately twice the number of buckets.

> 一个HashMap的实例有两个影响其性能的参数：初始容量（initial capacity） 和负载因子（load factor）。容量是指哈希表中的桶的数量，初始容量只是创建哈希表时的容量。负载因子是指在自动增加容量之前允许哈希表填充的程度的衡量标准（译注：即衡量哈希表的空/满程度，以便其增加容量）。当哈希表中的条目超过负载因子与当前容量的乘积时，哈希表将被重哈希（rehashed，即，重建内部数据结构）以便哈希表拥有大约两倍的桶数（译注：即自动扩容为大致原来容量的2倍）。

As a general rule, the default load factor (.75) offers a good tradeoff between time and space costs. Higher values decrease the space overhead but increase the lookup cost (reflected in most of the operations of the HashMap class, including get and put). The expected number of entries in the map and its load factor should be taken into account when setting its initial capacity, so as to minimize the number of rehash operations. If the initial capacity is greater than the maximum number of entries divided by the load factor, no rehash operations will ever occur.

> 通常，默认负载因子（0.75）在时间与空间成本之间提供了良好的权衡。较高的值会减少空间成本，但会增加查找成本（反映在HashMap类的大部分操作中，包含get和put）。在设置其初始容量时，应考虑map中的预期条目数及其负载因子，以便最小化重哈希操作的数量。如果初始容量大于最大条目数除以负载因子，则不会发成rehash操作。

If many mappings are to be stored in a HashMap instance, creating it with a sufficiently large capacity will allow the mappings to be stored more efficiently than letting it perform automatic rehashing as needed to grow the table. Note that using many keys with the same hashCode() is a sure way to slow down performance of any hash table. To ameliorate impact, when keys are Comparable, this class may use comparison order among keys to help break ties.
> 如果很多映射关系（mappings）需要存储在一个HashMao实例中，则相对于根据需要执行rehash操作扩展表的容量来说，使用足够大的初始容量创建它将使映射关系更有效地存储。注意许多keys使用同一个hashCode()在任何哈希表中都会减慢性能。


**Note that this implementation is not synchronized.** If multiple threads access a hash map concurrently, and at least one of the threads modifies the map structurally, it must be synchronized externally. (A structural modification is any operation that adds or deletes one or more mappings; merely changing the value associated with a key that an instance already contains is not a structural modification.) This is typically accomplished by synchronizing on some object that naturally encapsulates the map. If no such object exists, the map should be "wrapped" using the Collections.synchronizedMap method. This is best done at creation time, to prevent accidental unsynchronized access to the map:

> 注意这个实现是不同步的。如果多个线程同时访问一个哈希映射，并且至少有一个线程在结构上修改了此映射，则它必须保持外部同步。（结构修改是指添加或删除一个或多个映射的任何操作，仅更改与实例已包含的key关联的值不是结构修改。）这通常通过对自然封装该map的某个对象进行同步来实现。如果不存在此类对象，则应使用Collections.synchronizedMap方法“包装”该map。
这最好在创建时完成，以防止意外地不同步访问该map：

```
   Map m = Collections.synchronizedMap(new HashMap(...));
```

The iterators returned by all of this class's "collection view methods" are fail-fast: if the map is structurally modified at any time after the iterator is created, in any way except through the iterator's own remove method, the iterator will throw a ConcurrentModificationException. Thus, in the face of concurrent modification, the iterator fails quickly and cleanly, rather than risking arbitrary, non-deterministic behavior at an undetermined time in the future.

> 由所有此类的“集合视图方法”返回的迭代器都是快速失败的：如果在迭代器创建之后的任何时候对map进行结构修改，除非通过迭代器自己的remove方法，其他任何方式迭代器都将抛出ConcurrentModificationException。因此，在并发修改的情况下，迭代器快速而干净地失败，而不是在未来的未确定时间冒任意，非确定性行为的风险。

Note that the fail-fast behavior of an iterator cannot be guaranteed as it is, generally speaking, impossible to make any hard guarantees in the presence of unsynchronized concurrent modification. Fail-fast iterators throw ConcurrentModificationException on a best-effort basis. Therefore, it would be wrong to write a program that depended on this exception for its correctness: the fail-fast behavior of iterators should be used only to detect bugs.

> 请注意，迭代器的快速失败行为无法得到保证，因为一般来说，在存在不同步的并发修改时，不可能做出任何硬性保证。失败快速迭代器会尽最大努力抛出ConcurrentModificationException。
因此，编写依赖于此异常的程序以确保其正确性是错误的：迭代器的快速失败行为应该仅用于检测错误。

This class is a member of the Java Collections Framework.

> 该类是Java Collections Framework中的一员。

## 实现注意事项（Implementation notes)

This map usually acts as a binned (bucketed) hash table, but when bins get too large, they are transformed into bins of TreeNodes, each structured similarly to those in java.util.TreeMap. Most methods try to use normal bins, but relay to TreeNode methods when applicable (simply by checking instanceof a node). Bins of TreeNodes may be traversed and used like any others, but additionally support faster lookup when overpopulated. However, since the vast majority of bins in normal use are not overpopulated, checking for existence of tree bins may be delayed in the course of table methods.

> 该Map通常充当binned(bucketed)哈希表，但是当容器变得太大时，它们就会转成TreeNodes的bins，每个bins的结构都与java.util.TreeMap类型。大多数方法尝试使用普通bins，但在适用的情况下中继到TreeNode（简单的通过检测节点的实例）。TreeNodes的Bins可以像其他任何Bins一样遍历和使用，而且当填充过多时，还可以支持更快的查找。但是，由于在正常使用中绝大多数的bin并没有被填满，在使用表方法的过程中，对树容器存在性的检测可能会被延迟。

Tree bins (i.e., bins whose elements are all TreeNodes) are ordered primarily by hashCode, but in the case of ties, if two elements are of the same "class C implements Comparable<C>", type then their compareTo method is used for ordering. (We conservatively check generic types via reflection to validate this -- see method comparableClassFor).  The added complexity of tree bins is worthwhile in providing worst-case O(log n) operations when keys either have distinct hashes or are orderable, Thus, performance degrades gracefully under accidental or malicious usages in which hashCode() methods return values that are poorly distributed, as well as those in which many keys share a hashCode, so long as they are also Comparable. (If neither of these apply, we may waste about a factor of two in time and space compared to taking no precautions. But the only known cases stem from poor user programming practices that are already so slow that this makes little difference.)

> 树容器（即，其元素都是TreeNodes的容器）主要由hashCode排序，但在ties的情况下，如果两个元素具有相同的“class C implements Comparable<C>”（即实现了Comparable接口），则对他们使用compareTo方法排序。 （我们保守地通过反射来检查泛型类型以验证这一点 - 请参阅方法comparableClassFor）。 当keys具有不同的散列或可排序时，增加树容器的复杂性对于提供最坏情况```O(log n)```的操作是值得的。因此，在偶然或恶意的使用hashCode（）方法分布不佳的返回值，以及许多keys共享hashCode的值（只要它们具有可比性）时，性能会优雅地降级。 （如果这两者都不适用，与不采取任何预防措施相比，我们可能在时间和空间上浪费大约两倍。但是，唯一已知的案例源于糟糕的用户编程实践，这些实践已经非常缓慢，这几乎没有什么区别。）
 
Because TreeNodes are about twice the size of regular nodes, we use them only when bins contain enough nodes to warrant use (see TREEIFY_THRESHOLD). And when they become too small (due to removal or resizing) they are converted back to plain bins. In usages with well-distributed user hashCodes, tree bins are rarely used.  Ideally, under random hashCodes, the frequency of nodes in bins follows a Poisson distribution (http://en.wikipedia.org/wiki/Poisson_distribution) with a parameter of about 0.5 on average for the default resizing threshold of 0.75, although with a large variance because of resizing granularity. Ignoring variance, the expected occurrences of list size k are (exp(-0.5) * pow(0.5, k) / factorial(k)). The first values are:

> 由于TreeNodes的大小约为常规节点的两倍，因此只有当bin包含足够的节点以保证使用时，我们才使用它们（请参阅TREEIFY_THRESHOLD）。 当它们变得太小（由于移除或调整大小）时，它们会转换回普通bins。 在具有良好分布的用户hashCodes的用法中，很少使用树容器。 理想情况下，在随机hashCodes下，bin中节点的频率遵循泊松分布（Poisson distribution,http://en.wikipedia.org/wiki/Poisson_distribution），对于0.75的默认调整阈值，参数平均值约为0.5，尽管由于调整粒度而存在很大的差异。 忽略方差，列表大小k的预期出现是```(exp(-0.5) * pow(0.5, k) / factorial(k))```。 第一个值是

```
0:    0.60653066
1:    0.30326533
2:    0.07581633
3:    0.01263606
4:    0.00157952
5:    0.00015795
6:    0.00001316
7:    0.00000094
8:    0.00000006
```

more: less than 1 in ten million

> 更多：小于千万分之一。


The root of a tree bin is normally its first node. However, sometimes (currently only upon Iterator.remove), the root might be elsewhere, but can be recovered following parent links (method TreeNode.root()).

> 树bin的根通常是它的第一个节点。 但是，有时（目前仅在Iterator.remove上），根可能在其他地方，但可以在父链接之后恢复（方法TreeNode.root（））。
  
All applicable internal methods accept a hash code as an argument (as normally supplied from a public method), allowing them to call each other without recomputing user hashCodes. Most internal methods also accept a "tab" argument, that is normally the current table, but may be a new or old one when resizing or converting.

> 所有适用的内部方法都接受哈希码作为参数（通常从公共方法提供），允许它们相互调用而无需重新计算用户hashCodes。 大多数内部方法也接受“tab”参数，通常是当前表，但在调整大小或转换时可能是新的或旧的。


When bin lists are treeified, split, or untreeified, we keep them in the same relative access/traversal order (i.e., field Node.next) to better preserve locality, and to slightly simplify handling of splits and traversals that invoke iterator.remove. When using comparators on insertion, to keep a total ordering (or as close as is required here) across rebalancings, we compare classes and identityHashCodes as tie-breakers.

> 当bin列表被树化，拆分或未解析时，我们将它们保持在相同的相对访问/遍历顺序（即，字段Node.next）中以更好地保留局部性，并略微简化对调用iterator.remove的拆分和遍历的处理。 当在插入中使用比较器时，为了保持整个重新排序的总排序（或者在这里需要尽可能接近），我们比较类和identityHashCodes作为tie-breakers。

The use and transitions among plain vs tree modes is complicated by the existence of subclass LinkedHashMap. See below for hook methods defined to be invoked upon insertion, removal and access that allow LinkedHashMap internals to otherwise remain independent of these mechanics. (This also requires that a map instance be passed to some utility methods that may create new nodes.)

> 由于LinkedHashMap子类的存在，普通模式和树模式之间的使用和转换变得复杂。请参阅下面的定义在插入，删除和访问时调用的钩子方法，这些钩子方法允许LinkedHashMap内部保持独立于这些机制。(这还需要将映射实例传递给一些可能创建新节点的实用工具方法。)

The concurrent-programming-like SSA-based coding style helps avoid aliasing errors amid all of the twisty pointer operations.
> 类似于基于sa的编码风格的并行编程的有助于避免在所有扭曲指针操作中出现混叠错误。