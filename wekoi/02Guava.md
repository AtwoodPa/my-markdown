# Guava

## 1、基本工具【Basic Utils】
对条件做前置判断，经常用在方法的最前面，来对参数进行校验，不符合则抛出异常
```java
@Test
public void testPreconditions(){
    User user = new User();
    Preconditions.checkArgument(user != null,"user null error");// user为null则抛出IllegalArgumentException

    Preconditions.checkNotNull(user); //user为null则抛出NullPointerException
}
```
使用Optional 降低代码抛出NullPointerException的风险
```java
@Test
public void testOptional(){
    String str = "...";
    Optional<String> optional = Optional.fromNullable(str);
    if(optional.isPresent()){
        String tmp = optional.get();
    }
    str = optional.or("default string");
    str = optional.or(new Supplier<String>() {
        @Override
        public String get() {
            return "default string";
        }
    });
    str = optional.orNull();

    // 对optional中的value做转换操作
    optional.transform(new Function<String, Object>() {
        @Override
        public Object apply(String input) {
            return "transformed string";
        }
    });
}
```
对于String的操作有：repeat、joiner、split
```java
@Test
public void testStringsJoinerSplitterCaseFormat(){
    String str = new String("hello");
    String str1 = null;
    System.out.println(Strings.nullToEmpty(str1));//如果字符串为null，则转换为空字符串
    String repeat = Strings.repeat(str, 3);// 重复字符串为新的字符串
    System.out.println("repeat = " + repeat);
    Joiner joiner = Joiner.on("; ").skipNulls();
    String join = joiner.join("1", null, "2", "3", "4");// 使用；拼接字符串
    System.out.println("joiner = " + join);

    Iterable<String> split = Splitter.on(";")
            .trimResults()
            .omitEmptyStrings()
            .split(join);
    // 遍历split
    Iterator<String> iterator = split.iterator();
    while (iterator.hasNext()) {
        System.out.println(iterator.next());
    }
    String s = CaseFormat.LOWER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "user_name");// 将字符串从low underscore命名格式转换为low camel命名。
    System.out.println("s = " + s);

}
```

## 2、集合【Collections】
- 不可变集合
  - 创建方式
    - of
      - List
        - ImmutableList.of("a", "b", "c")
      - Set
        - ImmutableSet.of("a", "b", "c")
      - Map
        - ImmutableMap.of("a", 1, "b", 2, "c", 3)
    - Builder
      - List
        - ImmutableList.<String>builder().add("a").add("b").add("c").build();
      - Set
        - ImmutableSet.<String>builder().add("a").add("b").add("c").build();
      - Map
        - ImmutableMap.<String, Integer>builder().put("a", 1).put("b", 2).put("c", 3).build();
    - copyOf
      - List
        - ImmutableList.copyOf(list);
      - Set
        - ImmutableSet.copyOf(set);
      - Map
        - ImmutableMap.copyOf(map);

```java
@Test
public void testImmutableList() throws Exception {
    ImmutableList<String> list = ImmutableList.of("1","2","3");
    for (int i = 0; i < list.size(); i++) {
        System.out.println(i+ " => " + list.get(i));
    }
    // Multiset可以多次添加相等的元素,主要统计给定元素的个数。
    HashMultiset<String> multiset = HashMultiset.create();
    multiset.add("1");
    multiset.add("1");
    System.out.println(multiset.count("1"));// 统计元素“1”的个数
    // Multimap是一个key映射多值的Map，类似于Map<K, List<V>>、Map<K, Set<V>>
    Multimap<String,String> multimap = ArrayListMultimap.create();
    multimap.put("test","1");
    multimap.put("test","2");
    System.out.println(multimap.get("test"));
}
// Builder创建不可变集合
public static final ImmutableSet<Color> GOOGLE_COLORS =
            ImmutableSet.<Color>builder()
                    .addAll(WEBSAFE_COLORS)
                    .add(new Color(0, 191, 255))
                    .build();
@Test
public void testListsMaps(){
    ArrayList<String> strings = Lists.newArrayList("1", "2", "3");
    strings.stream()
            .forEach(System.out::print);
    ArrayList<String> list = Lists.newArrayListWithCapacity(100);

}
```
   
2、新集合类型
- Multiset
  - Multiset是一个key映射多值的Map，类似于Map<K, List<V>>、Map<K, Set<V>>
  - Multiset可以多次添加相等的元素,主要统计给定元素的个数。
  - Multiset提供了如下方法：
    - elementSet()：所有不重复元素的集合，类型为Set<E>；
    - entrySet()：所有元素的集合，类型为Set<Multiset.Entry<E>>，Multiset.Entry是Multiset的内部接口，包含了元素与计数的映射；
    - setCount(E, int)：设置某一个元素的重复次数；
    - count(E)：返回给定元素的计数；
    - add(E, int)：增加给定元素的计数；
    - remove(E, int)：减少给定元素的计数；
    - remove(E)：清除给定元素的计数；
    - removeAll(E)：清除所有给定元素的计数；
    - retainAll(E)：保留给定元素的计数，清除其他元素的计数；
    - size()：所有元素计数的总和；
    - isEmpty()：与size()是否为0等价；
    - iterator()：返回所有元素的迭代器，元素重复次数等同于它们在Multiset中的计数；
    - equals(Object)：如果给定对象和Multiset中的元素计数都相等，则返回true；
    - hashCode()：返回Multiset的哈希码，定义为entrySet().hashCode()；
    - toString()：返回Multiset的字符串表示，格式为[element1 x count1, element2 x count2, ...]。

```java
/**
     * 场景：统计一个名单中每个名字出现的次数
     */
    @Test
    public void testHashMultiset(){
        List<String> nameList = Arrays.asList("张三", "李四", "王五", "乔二娃", "张三", "李四", "Tom");
        Map<String ,Integer> nameCountMap = new HashMap<>();

        for (String name : nameList){
            Integer count = nameCountMap.get(name);
            System.out.println(name + " ==" + count);
            nameCountMap.put(name,count != null ? ++count : 1);
        }
        System.out.println(nameCountMap.get("张三"));

        // 使用Multiset
        Multiset<String> multiset = HashMultiset.create();
        multiset.addAll(nameList);
        int count = multiset.count("张三");
        System.out.println("张三 count = " + count);
        
    }

@Test
    public void testMultiset(){
        // 原始统计一个词在文档中出现的次数
//        Map<String, Integer> counts = new HashMap<>();
//        String[] words = {"1", "2", "1", "4", "5"};
//        for (String word: words){
//            Integer count = counts.get(word);
//            if (count == null) {
//                counts.put(word,1);
//            }else {
//                counts.put(word,count + 1);
//            }
//        }
        Multiset<String> counts = HashMultiset.create();
        counts.add("a", 1);
        counts.add("a", 2);
        counts.add("a", 3);
        counts.add("b",1);
        counts.add("b",2);
        counts.add("b",3);
        counts.add("b",4);
        System.out.println(counts.count("a"));// 统计a元素的个数
        counts.remove("a",2);// 减少2个a元素
        System.out.println(counts.count("a"));// 统计a元素的个数
        counts.setCount("a",99);// 指定a元素的个数为99个
        System.out.println(counts.count("a"));// 统计a元素的个数
        Set<Multiset.Entry<String>> entries = counts.entrySet();
        Iterator<Multiset.Entry<String>> iterator = entries.iterator();
        while (iterator.hasNext()){// 通过迭代器遍历元素
            System.out.println(iterator.next());
        }
        System.out.println(counts.size());// 返回集合元素的总个数
    }
```
3、新map类型
- Multimap
  - Multimap是一个key映射多值的Map，类似于Map<K, List<V>>、Map<K, Set<V>>
  - Multimap提供了如下方法：
    - put(K, V)：往Multimap中添加键值对；
    - putAll(K, Iterable<V>)：往Multimap中添加键值集合；
    - remove(K, V)：从Multimap中移除键值对，如果有这样的键值对存在的话；
    - removeAll(K)：移除键所对应的所有值；
    - replaceValues(K, Iterable<V>)：替换某个键对应的所有值；
    - entries()：返回Multimap中所有”键-单个值映射”——包括重复键；
    - keys()：返回Multimap中所有键的集合——包括重复键；
    - values()：返回Multimap中所有”单个值”所构成的集合——重复值被视为单个元素；
    - size()：返回所有”键-单个值映射”的个数——也就是不同键的总数（重复键视为同一个键）；
    - isEmpty()：判断Multimap是否为空，等价于size() == 0；
    - containsKey(K)：判断Multimap是否包含指定的键；
    - containsValue(V)：判断Multimap是否包含指定的值；
    - containsEntry(K, V)：判断Multimap是否包含指定的键值对；
    - get(K)：获取指定”键”的值集合；
    - removeAll(K)：清除指定键的所有映射值；
    - keySet()：返回Multimap中所有不同的键的集合，返回集合不包含重复键；
    - asMap()：返回Multimap的Map<K, Collection<V>>形式；
    - equals(Object)：如果指定的对象和Multimap相等，则返回true；
    - hashCode()：返回Multimap的哈希码值。
  - 实现类
    - ArrayListMultimap
    - HashMultimap
    - LinkedHashMultimap
    - LinkedListMultimap
```java
@Test
public void testMultimap() {
    // 需求：有一批学生名单，需要按学生姓名进行分类
    // 使用HashMap实现Multimap
    List<Student> studentList = createStudentList();
    Map<String, List<Student>> studentMap = new HashMap<>();
    for (Student student : studentList) {
        List<Student> students = studentMap.get(student.getName());
        if (students == null) {
            students = new ArrayList<>();
            studentMap.put(student.getName(), students);
        }
        students.add(student);
    }
    // 遍历studentMap
    studentMap.forEach((s, students) -> {
        for (Student student : students) {
            System.out.println("s = " + s);
            System.out.println("student = " + student);
        }
    });
    // Lambda优化
    List<Student> studentListLambda = createStudentList();
    Map<String, List<Student>> studentMapLambda = new HashMap<>();
    for (Student student: studentListLambda) {
        List<Student> students = studentMapLambda.computeIfAbsent(student.getName(), k -> new ArrayList<>());
        students.add(student);
    }
    // 使用Multimap实现
    // 获取key为String，value为List的Multimap实例
    Multimap<String,Student> multiMap = ArrayListMultimap.create();
    List<Student> students = createStudentList();
    // 初始化multimap
    students.forEach(student -> {
        // 映射结果（姓名，学生）
        multiMap.put(student.getName(),student);
    });
    // 遍历multiMap
    multiMap.entries()
            .stream()
            .forEach(stringStudentEntry
                        -> System.out.println(
                                stringStudentEntry.getKey() + "="
                                        + stringStudentEntry.getValue()));
}
```
4、BiMap
- 对键值对保证强制唯一性的Map
- BiMap提供了如下方法：
  - put(K, V)：往BiMap中添加键值对，如果已存在相同键或相同值，则抛出IllegalArgumentException异常；
  - forcePut(K, V)：往BiMap中添加键值对，如果已存在相同键或相同值，则覆盖原有的键值对；
  - putAll(Map<? extends K, ? extends V>)：往BiMap中添加另一个Map的键值对，如果已存在相同键或相同值，则抛出IllegalArgumentException异常；
  - putAll(BiMap<? extends K, ? extends V>)：往BiMap中添加另一个BiMap的键值对，如果已存在相同键或相同值，则抛出IllegalArgumentException异常；
  - forcePutAll(Map<? extends K, ? extends V>)：往BiMap中添加另一个Map的键值对，如果已存在相同键或相同值，则覆盖原有的键值对；
  - forcePutAll(BiMap<? extends K, ? extends V>)：往BiMap中添加另一个BiMap的键值对，如果已存在相同键或相同值，则覆盖原有的键值对；
  - remove(Object)：移除指定键对应的键值对；
  - removeValue(Object)：移除指定值对应的键值对；
  - inverse()：返回BiMap的反转视图，即键值对反转后的BiMap，反转后的BiMap的键为原BiMap的值，值为原BiMap的键；
  - values()：返回BiMap中所有值的集合，返回集合不包含重复值；
  - inverse()：返回BiMap的反转视图，即键值对反转后的BiMap，反转后的BiMap的键为原BiMap的值，值为原BiMap的键；
  - values()：返回BiMap中所有值的集合，返回集合不包含重复值；
  - inverse()：返回BiMap的反转视图，即键值对反转后的BiMap，反转后的BiMap
- 实现类
  - HashBiMap
  - ImmutableBiMap
  - EnumBiMap
```java
/**
 * BiMap ： 保证  键和值  都是唯一的
 * BiMap强制其value的唯一性
 */
@Test
public void testBiMap(){
    BiMap<String ,String> upperToSmall = HashBiMap.create();
    upperToSmall.put("A","a");
    upperToSmall.put("B", "b");
    upperToSmall.put("C", "c");
    // upperToSmall.put("D", "c");// 不允许插入已经存在的value
    System.out.println(upperToSmall.get("A"));

}
```

5、Table
- Table是一个”行-列-值”的数据结构，和我们熟知的Map非常相似。Table有两个支持所有类型的键：”行”和”列”。因此，Table支持两种查找方式：rowMap()和columnMap()。
- Table提供了如下方法：
  - boolean contains(Object rowKey, Object columnKey)：判断Table中是否包含指定的键值对；
  - boolean containsRow(Object rowKey)：判断Table中是否包含指定的行；
  - boolean containsColumn(Object columnKey)：判断Table中是否包含指定的列；
  - boolean containsValue(Object value)：判断Table中是否包含指定的值；
  - V get(Object rowKey, Object columnKey)：获取Table中指定键对应的值，如果没有对应的键值对，则返回null；
  - boolean isEmpty()：判断Table是否为空，等价于size() == 0；
  - int size()：返回Table中所有”单元格”的数量，等价于所有行的所有列的数量之和；
  - void clear()：清除Table中所有的键值对；
  - V put(R rowKey, C columnKey, V value)：往Table中添加键值对，如果已存在相同的键值对，则覆盖原有的键值对；
  - void putAll(Table<? extends R, ? extends C, ? extends V>)：往Table中添加另一个Table的键值对；
  - V remove(Object rowKey, Object columnKey)：移除Table中指定的键值对；
  - Map<C, V> row(R rowKey)：返回Table中指定行对应的所有列构成的Map，对该Map的操作也将影响到原Table；
  - Map<R, V> column(C columnKey)：返回Table中指定列对应的所有行构成的Map，对该Map的操作也将影响到原Table；
  - Set<Cell<R, C, V>> cellSet()：返回Table中所有”单元格”的集合，Cell类似于Map.Entry，也包含rowKey、columnKey、value等成员变量；
  - Map<R, Map<C, V>> rowMap()：返回”行-列-值”形式的Map，对该Map的操作也将影响到原Table；
  - Map<C, Map<R, V>> columnMap()：返回”列 - 行 - 值”形式的Map，对该Map的操作也将影响到原Table。
- 实现类
  - HashBasedTable
  - TreeBasedTable
  - ImmutableTable
```java
@Test
public void testTable(){
    Table<String,String,Integer> table = HashBasedTable.create();
    table.put("张三","语文",80);
    table.put("张三","数学",90);
    table.put("张三","英语",70);
    table.put("李四","语文",60);
    table.put("李四","数学",100);
    table.put("李四","英语",80);
    // 获取张三的成绩
    Map<String, Integer> zhangsan = table.row("张三");
    System.out.println("zhangsan = " + zhangsan);
    // 获取语文成绩
    Map<String, Integer> yuwen = table.column("语文");
    System.out.println("yuwen = " + yuwen);
    // 获取所有成绩
    Set<Table.Cell<String, String, Integer>> cells = table.cellSet();
    System.out.println("cells = " + cells);
    // 获取所有学生
    Set<String> students = table.rowKeySet();
    System.out.println("students = " + students);
    // 获取所有课程
    Set<String> courses = table.columnKeySet();
    System.out.println("courses = " + courses);
    // 获取所有成绩
    Collection<Integer> scores = table.values();
    System.out.println("scores = " + scores);
    // 获取所有成绩
    Map<String, Map<String, Integer>> rowMap = table.rowMap();
    System.out.println("rowMap = " + rowMap);
    // 获取所有成绩
    Map<String, Map<String, Integer>> columnMap = table.columnMap();
    System.out.println("columnMap = " + columnMap);
}
```

6、ClassToInstanceMap
- ClassToInstanceMap是一种特殊的Map：它的键是类型，而值是符合键所指类型的对象。
- ClassToInstanceMap提供了如下方法：
  - V getInstance(Class<T>)：返回给定类型对应的值，如果这个类型没有对应的值，则返回null；
  - V putInstance(Class<T>, V)：为指定的类型设置值，如果这个类型之前已经有值存在，则覆盖原有的值。
- 实现类
  - MutableClassToInstanceMap
  - ImmutableClassToInstanceMap
```java
@Test
public void testClassToInstanceMap(){
    ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
    numberDefaults.putInstance(Integer.class, Integer.valueOf(0));
    numberDefaults.putInstance(Long.class, Long.valueOf(0));
    numberDefaults.putInstance(Double.class, Double.valueOf(0));
    System.out.println(numberDefaults.get(Integer.class));
    System.out.println(numberDefaults.get(Long.class));
    System.out.println(numberDefaults.get(Double.class));
}
```
7、RangeSet
- RangeSet描述了一组不相连的、非空的区间。
- RangeSet提供了如下方法：
  - add(Range<C>)：向RangeSet中添加一个区间；
  - addAll(RangeSet<C>)：向RangeSet中添加一组区间；
  - contains(C)：判断RangeSet中是否包含指定的元素；
  - containsAll(RangeSet<C>)：判断RangeSet中是否包含指定的RangeSet；
  - encloses(Range<C>)：判断RangeSet中是否包含指定的区间；
  - enclosesAll(RangeSet<C>)：判断RangeSet中是否包含指定的RangeSet中的所有区间；
  - span()：返回RangeSet的区间的并集；
  - remove(Range<C>)：从RangeSet中移除一个区间；
  - removeAll(RangeSet<C>)：从RangeSet中移除一组区间；
  - complement()：返回RangeSet中不包含的区间；
  - asRanges()：返回RangeSet中包含的区间的视图，返回类型为Set<Range<C>>；
  - asDescendingSetOfRanges()：返回RangeSet中包含的区间的视图，返回类型为Set<Range<C>>，并且按照区间的上界降序排列；
  - iterator()：返回RangeSet中包含的区间的迭代器，迭代器中的元素按照区间的上界升序排列；
  - subRangeSet(Range<C>)：返回RangeSet中包含的区间与指定区间的交集；
  - toString()：返回RangeSet的字符串表示。
- 实现类
  - TreeRangeSet
  - ImmutableRangeSet
```java
@Test
public void testRangeSet(){
    RangeSet<Integer> rangeSet = TreeRangeSet.create();
    rangeSet.add(Range.closed(1,10));
    rangeSet.add(Range.closedOpen(11,15));
    rangeSet.add(Range.open(15,20));
    rangeSet.add(Range.openClosed(0,0));
    System.out.println(rangeSet);
    System.out.println(rangeSet.contains(10));
    System.out.println(rangeSet.contains(15));
    System.out.println(rangeSet.contains(21));
    System.out.println(rangeSet.rangeContaining(5));
    System.out.println(rangeSet.rangeContaining(15));
    System.out.println(rangeSet.rangeContaining(21));
    System.out.println(rangeSet.encloses(Range.closed(5,10)));
    System.out.println(rangeSet.encloses(Range.closed(15,20)));
    System.out.println(rangeSet.encloses(Range.closed(0,0)));
    System.out.println(rangeSet.span());
    System.out.println(rangeSet.complement());
    System.out.println(rangeSet.asRanges());
    System.out.println(rangeSet.asDescendingSetOfRanges());
    System.out.println(rangeSet.subRangeSet(Range.closed(5,15)));
}
```
8、RangeMap
- RangeMap描述了”不相交的、非空的区间”到特定值的映射。
- RangeMap提供了如下方法：
  - put(Range<K>, V)：向RangeMap中添加映射关系；
  - putAll(RangeMap<K, V>)：向RangeMap中添加一组映射关系；
  - get(K)：获取RangeMap中指定键对应的值；
  - remove(K)：移除RangeMap中指定键对应的映射关系；
  - asMapOfRanges()：返回RangeMap的视图，返回类型为Map<Range<K>, V>；
  - subRangeMap(Range<K>)：返回RangeMap的子集，子集中的映射关系的键必须是指定区间的子集。
- 实现类
  - TreeRangeMap
  - ImmutableRangeMap
```java
@Test
public void testRangeMap(){
    RangeMap<Integer,String> rangeMap = TreeRangeMap.create();
    rangeMap.put(Range.closed(1,10),"foo");
    rangeMap.put(Range.closed(3,6),"bar");
    rangeMap.put(Range.closed(8,10),"foo");
    rangeMap.put(Range.closed(12,16),"foo");
    System.out.println(rangeMap);
    System.out.println(rangeMap.get(5));
    System.out.println(rangeMap.get(11));
    System.out.println(rangeMap.get(15));
    System.out.println(rangeMap.asMapOfRanges());
    System.out.println(rangeMap.subRangeMap(Range.closed(5,15)));
}
```
## 3、缓存【Caches】
- Guava Cache缓存的适用场景
  - 用内存空间来提升速度
    - 本地缓存
  - 某些键会被查询一次以上
  - 确保缓存中的数量总量适中，避免内存溢出（不会超出内存大小）
- Guava Cache缓存的特性
  - 自动加载缓存
  - 缓存回收
  - 缓存刷新
  - 缓存预加载
- Guava Cache常用方法
  - get(K, Callable<V>)：获取缓存中指定键对应的值，如果键不存在，则使用Callable进行加载；
  - getAllPresent(Iterable<K>)：获取缓存中指定键集合对应的值集合，如果键不存在，则对应的值为null；
  - put(K, V)：向缓存中添加一个键值对；
  - putAll(Map<? extends K, ? extends V>)：向缓存中添加一组键值对；
  - invalidate(Object)：从缓存中移除指定的键值对；
  - invalidateAll(Iterable<?>)：从缓存中移除指定的键值对集合；
  - invalidateAll()：从缓存中移除所有的键值对；
  - size()：返回缓存中键值对的数量；
  - stats()：返回缓存的统计数据；
  - asMap()：返回缓存中所有键值对的视图，返回类型为ConcurrentMap<K, V>；
  - cleanUp()：清除缓存中的无效键值对。
  - refresh(K)：刷新缓存中指定键对应的值，该方法是异步执行的，如果多个线程同时调用该方法，则只有一个线程会执行该方法，其他线程会等待执行结果。
  - getIfPresent(K)：获取缓存中指定键对应的值，如果键不存在，则返回null。
- 缓存清除策略
  - expireAfterWrite 写缓存后多久过期 
  - expireAfterAccess 读写缓存后多久过期 
  - refreshAfterWrite 写入数据后多久过期,只阻塞当前数据加载线程,其他线程返回旧值

```java
/**
 * TODO Guava Cache
 */
public class GuavaCacheService {

    public void setCache() {
        LoadingCache<Integer, String> cache = CacheBuilder.newBuilder()
                //设置并发级别为8，并发级别是指可以同时写缓存的线程数
                .concurrencyLevel(8)
                //设置缓存容器的初始容量为10
                .initialCapacity(10)
                //设置缓存最大容量为100，超过100之后就会按照LRU最近虽少使用算法来移除缓存项
                .maximumSize(100)
                //是否需要统计缓存情况,该操作消耗一定的性能,生产环境应该去除
                .recordStats()
                //设置写缓存后n秒钟过期
                .expireAfterWrite(60, TimeUnit.SECONDS)
                //设置读写缓存后n秒钟过期,实际很少用到,类似于expireAfterWrite
                //.expireAfterAccess(17, TimeUnit.SECONDS)
                //只阻塞当前数据加载线程，其他线程返回旧值
                //.refreshAfterWrite(13, TimeUnit.SECONDS)
                //设置缓存的移除通知
                .removalListener(notification -> {
                    System.out.println(notification.getKey() + " " + notification.getValue() + " 被移除,原因:" + notification.getCause());
                })
                //build方法中可以指定CacheLoader，在缓存不存在时通过CacheLoader的实现自动加载缓存
                .build(new DemoCacheLoader());

        //模拟线程并发
        new Thread(() -> {
            //非线程安全的时间格式化工具
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("HH:mm:ss");
            try {
                for (int i = 0; i < 10; i++) {
                    String value = cache.get(1);
                    System.out.println(Thread.currentThread().getName() + " " + simpleDateFormat.format(new Date()) + " " + value);
                    TimeUnit.SECONDS.sleep(3);
                }
            } catch (Exception ignored) {
            }
        }).start();

        new Thread(() -> {
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("HH:mm:ss");
            try {
                for (int i = 0; i < 10; i++) {
                    String value = cache.get(1);
                    System.out.println(Thread.currentThread().getName() + " " + simpleDateFormat.format(new Date()) + " " + value);
                    TimeUnit.SECONDS.sleep(5);
                }
            } catch (Exception ignored) {
            }
        }).start();
        //缓存状态查看
        System.out.println(cache.stats().toString());

    }

    /**
     * 随机缓存加载,实际使用时应实现业务的缓存加载逻辑,例如从数据库获取数据
     */
    public static class DemoCacheLoader extends CacheLoader<Integer, String> {
        @Override
        public String load(Integer key) throws Exception {
            System.out.println(Thread.currentThread().getName() + " 加载数据开始");
            TimeUnit.SECONDS.sleep(8);
            Random random = new Random();
            System.out.println(Thread.currentThread().getName() + " 加载数据结束");
            CacheBuilder.newBuilder()
                    // 设置并发级别为cpu核心数
                    .concurrencyLevel(Runtime.getRuntime().availableProcessors())
                    .build();

            return "value:" + random.nextInt(10000);
        }
    }

    /**
     * 移除操作监听器
     */
    public void clearListener(){
        RemovalListener<String, String> listener = notification -> System.out.println("[" + notification.getKey() + ":" + notification.getValue() + "] is removed!");
        Cache<String,String> cache = CacheBuilder.newBuilder()
                .maximumSize(5)
                .removalListener(listener)
                .build();
    }
}
```
## 4、并发【Concurrency】
Guava使用ListenableFuture代替Future，ListenableFuture是Future的扩展，可以注册回调函数，当计算完成时会自动触发回调函数。
- ListenableFuture的优点
  - 异步执行
  - 支持回调
  - 支持链式调用
- ListenableFuture的缺点
  - 无法获取任务执行结果
  - 无法控制任务执行起始时间
  - 无法取消任务
- ListenableFuture常见方法
  - addListener(Runnable, Executor)：为ListenableFuture添加回调函数，当计算完成时会自动触发回调函数；
  - get()：获取计算结果，如果计算没有完成，则会阻塞当前线程；
  - get(long, TimeUnit)：获取计算结果，如果计算没有完成，则会阻塞当前线程，直到超时；
  - isDone()：判断计算是否完成；
  - isCancelled()：判断计算是否被取消；
  - cancel(boolean)：取消计算，如果计算已经完成或者已经被取消，则返回false，否则返回true。
  - transformAsync(Function<? super V, ListenableFuture<B>>, Executor)：将ListenableFuture的计算结果转换为另一个ListenableFuture；
## 5、字符串处理【Strings】
- Strings
  - Strings.isNullOrEmpty(String)：判断字符串是否为空或者长度为0；
  - Strings.nullToEmpty(String)：如果字符串为null，则转换为空字符串；
  - Strings.emptyToNull(String)：如果字符串为空字符串，则转换为null；
  - Strings.commonPrefix(CharSequence, CharSequence)：返回两个字符串的相同前缀；
  - Strings.commonSuffix(CharSequence, CharSequence)：返回两个字符串的相同后缀；
  - Strings.repeat(String, int)：重复字符串，第二个参数为重复的次数；
  - Strings.padStart(String, int, char)：在字符串前面填充指定的字符，使得字符串长度达到指定长度；
  - Strings.padEnd(String, int, char)：在字符串后面填充指定的字符，使得字符串长度达到指定长度；


## 6、原生类型【Primitives】
- Primitives
  - Primitives.asList(T...)：将原生类型数组转换为对应的包装类型数组；
  - Primitives.toByteArray(int)：将int转换为byte数组；
  - Primitives.fromByteArray(byte[])：将byte数组转换为int；
  - Primitives.join(数组, 分隔符)：将数组用分隔符连接成字符串；
  - Primitives.asList(数组)：将数组转换为List；
  - Primitives.toArray(Collection, 类型)：将集合转换为数组；
  - Primitives.compare(原生类型, 原生类型)：比较两个原生类型的大小；
  - Primitives.contains(原生类型[], 原生类型)：判断原生类型数组是否包含指定的原生类型；
  - Primitives.indexOf(原生类型[], 原生类型)：返回原生类型在原生类型数组中的索引，如果不存在，则返回-1；
  - Primitives.lastIndexOf(原生类型[], 原生类型)：返回原生类型在原生类型数组中的最后一个索引，如果不存在，则返回-1；
  - Primitives.asList(原生类型[])：将原生类型数组转换为List；
  - Primitives.toArray(Collection, 原生类型)：将集合转换为原生类型数组；
  - Primitives.constrainToRange(原生类型, 原生类型, 原生类型)：将原生类型限制在指定的范围内；
  - Primitives.saturatedCast(long)：将long转换为int，如果long值超出int的范围，则返回Integer.MAX_VALUE或Integer.MIN_VALUE；
  - Primitives.checkedCast(long)：将long转换为int，如果long值超出int的范围，则抛出IllegalArgumentException异常；
  - Primitives.compareUnsigned(原生类型, 原生类型)：比较两个无符号原生类型的大小；
  - Primitives.divide(原生类型, 原生类型)：计算两个原生类型的商；
  - Primitives.pow(原生类型, 原生类型)：计算原生类型的幂；
## 7、排序【Ordering】
- Ordering
  - Ordering.natural()：对可排序类型做自然排序，如数字按大小，日期按先后排序；
  - Ordering.usingToString()：按对象的字符串形式做字典排序[lexicographical ordering]；
  - Ordering.from(Comparator)：把给定的Comparator转化为排序器；
  - compound(Comparator)：合成另一个比较器，以处理当前排序器中的相等情况；
  - reverse()：获取语义相反的排序器；
  - nullsFirst()：使用当前排序器，但额外把null值排到最前面；
  - nullsLast()：使用当前排序器，但额外把null值排到最后面；
  - onResultOf(Function)：对集合中元素调用Function，再按返回值用当前排序器排序；
  - lexicographical()：返回一个新的排序器，使用当前排序器，但比较的是两个可迭代对象的元素，按元素逐个比较，直到找到一个非零值，此时返回非零值，若到某个序列的结尾则视为长度更短的序列更小；
  - isOrdered(Iterable)：判断可迭代对象是否已按排序器排序：允许有排序值相等的元素；
  - isStrictlyOrdered(Iterable)：判断可迭代对象是否已严格按排序器排序：不允许排序值相等的元素；
  - sortedCopy(Iterable)：判断可迭代对象是否已按排序器排序，如果已排序则返回该对象，否则抛出IllegalArgumentException异常；
  - min(E, E)：返回两个参数中最小的那个。如果相等，则返回第一个参数；
  - min(E, E, E, E...)：返回多个参数中最小的那个。如果有超过一个参数都最小，则返回第一个最小的参数；
  - min(Iterable)：返回迭代器中最小的元素。如果可迭代

## 8、事件总线【EventBus】
- EventBus
  - EventBus是Guava提供的一个事件发布-订阅模型的实现，主要用于组件之间的解耦。
  - EventBus的使用
    - 定义事件
    - 定义事件监听器
    - 注册事件监听器
    - 发布事件
  - EventBus的优点
    - 事件发布者和事件监听器之间没有依赖关系，可以独立开发，解耦合；
    - 事件的发布者不需要知道谁来处理该事件，事件的监听器也不需要知道事件何时产生；
    - 事件的发布者和事件的监听器之间通过事件总线进行通信，事件总线是事件发布者和事件监听器的中介，事件发布者和事件监听器之间不会直接进行通信，降低了耦合度。
  - EventBus的缺点
    - 事件的发布者和事件的监听器之间没有直接关系，不容易定位问题；
    - 事件的发布者和事件的监听器之间通过事件总线进行通信，事件总线是事件发布者和事件监听器的中介，增加了事件处理的复杂度。
  - EventBus的使用场景
    - 事件发布者和事件监听器之间没有依赖关系，可以独立开发，解耦合；
    - 事件的发布者不需要知道谁来处理该事件，事件的监听器也不需要知道事件何时产生；
    - 事件的发布者和事件的监听器之间通过事件总线进行通信，事件总线是事件发布者和事件监听器的中介，事件发布者和事件监听器之间不会直接进行通信，降低了耦合度。
  - EventBus的缺点
    - 事件的发布者和事件的监听器之间没有直接关系，不容易定位问题；
    - 事件的发布者和事件的监听器之间通过事件总线进行通信，事件总线是事件发布者和

## 9、数学运算【Math】
- Math
  - Math.max(int, int)：返回两个int值中较大的那个；
  - Math.max(long, long)：返回两个long值中较大的那个；
  - Math.max(float, float)：返回两个float值中较大的那个；
  - Math.max(double, double)：返回两个double值中较大的那个；
  - Math.min(int, int)：返回两个int值中较小的那个；
  - Math.min(long, long)：返回两个long值中较小的那个；
  - Math.min(float, float)：返回两个float值中较小的那个；
  - Math.min(double, double)：返回两个double值中较小的那个；
  - Math.sqrt(double)：返回double值的平方根；
  - Math.pow(double, double)：返回第一个double值的第二个double值次方的值；
  - Math.log(double)：返回double值的自然对数（底数为e）；
  - Math.log10(double)：返回double值的以10为底的对数；
  - Math.exp(double)：返回double值的e的次方的值；
  - Math.abs(int)：返回int值的绝对值；
  - Math.abs(long)：返回long值的绝对值；
  - Math.abs(float)：返回float值的绝对值；
  - Math.abs(double)：返回double值的绝对值；
  - Math.ceil(double)：返回大于等于参数的最小的整数；
  - Math.floor(double)：返回小于等于参数的最大的整数；
  - Math.rint(double)：返回与参数最接近的整数。如果有两个整数与参数同样接近，则返回其中的偶数；
  - Math.round(float)：返回与参数最接近的整数。如果有两个整数与参数同样接近，则返回其中的偶数；
  - Math.round(double)：返回与参数最接近的整数。如果有两个整数与参数同样接近

## 10、I/O
- Files
  - Files.exists(Path, LinkOption...)：判断文件是否存在；
  - Files.createDirectory(Path, FileAttribute<?>...)：创建目录；
  - Files.createDirectories(Path, FileAttribute<?>...)：创建目录，包括不存在的父目录；
  - Files.createFile(Path, FileAttribute<?>...)：创建文件；
  - Files.createTempDirectory(Path, String, FileAttribute<?>...)：创建临时目录；
  - Files.createTempFile(Path, String, String, FileAttribute<?>...)：创建临时文件；
  - Files.copy(Path, Path, CopyOption...)：复制文件或目录；
    - copy(from,to)
  - Files.move(Path, Path, CopyOption...)：移动文件或目录；
    - move(from,to)
  - Files.delete(Path)：删除文件或目录；
  - Files.deleteIfExists(Path)：删除文件或目录，如果文件或目录不存在，则不抛出异常；
  - Files.readAllBytes(Path)：读取文件的所有字节；
  - Files.readAllLines(Path)：读取文件的所有行；
  - Files.size(Path)：获取文件的大小；
  - Files.getAttribute(Path, String, LinkOption...)：获取文件的属性；
  - Files.setAttribute(Path, String, Object, LinkOption...)：设置文件的属性；
  - Files.getFileStore(Path)：获取文件所在的文件系统；
  - Files.isSymbolicLink(Path)：判断文件是否为符号链接；
  - Files.isReadable(Path)：判断文件是否可读；
  - Files.isWritable(Path)：判断文件是否可写；
  - Files.isExecutable(Path)：判断文件是否可执行；
  - Files.isHidden(Path)：判断文件是否隐藏；
  - Files.probeContentType(Path)：获取文件的MIME类型；
  - Files.walk(Path, FileVisitOption...)：遍历目录及其子目录；
  - Files.walkFileTree(Path, FileVisitor<? super Path>)：遍历目录及其子目录，可以自定义遍历规则；
  - Files.lines(Path)：读取文件的所有行，返回类型为Stream<String>；
  - Files.lines(Path, Charset)：读取文件的所有行，返回类型为Stream<String>；
  - Files.newBufferedReader(Path)：