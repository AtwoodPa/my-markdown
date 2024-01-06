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