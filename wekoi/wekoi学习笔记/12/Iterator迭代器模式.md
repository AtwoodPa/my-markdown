# Iterator迭代器模式

## Iterator迭代器模式

**介绍：**

*   迭代器模式是一种行为型设计模式，它允许你在不暴露集合底层表示（并不知道集合底层使用何种方式对数据尽心存储）的情况下遍历集合中的元素。
    
*   这种模式提供了一种方法，可以顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。
    
*   迭代器模式通常包括两个角色：迭代器和聚合对象。
    
    *   迭代器（==Iterator==,**读取元素**）
        
        *   负责定义访问和遍历元素的接口
            
        *   实现了该接口的类，拥有访问聚合对象中元素的能力
            
    *   聚合对象（==Aggregate==,**存储元素**）
        
        *   负责定义创建相应迭代器对象的接口，该接口返回一个适当的迭代器实例
            
        *   实现了该接口的类将成为一个可以保存多个元素的集合，就像数组一样
            

以下是一个简单的迭代器实例，演示了如何使用迭代器模式遍历一个集合：

### 1、抽象实例

**创建迭代器接口：**

    /**
     * 迭代器接口
     *
     * @author supanpan
     * @date 2023/12/21
     */
    public interface Iterator<T>{
      /**
       * 判断是否存在下一个元素
       * @return: 循环终止条件
       *  true：集合中存在下一个元素
       *  false：集合中不存在下一个元素
       */
      boolean hasNext();
    
      /**
       * 获取下一个元素，并且将迭代器指针位置移动到下一个元素
       */
      T next();
    }

实现迭代器接口，具体迭代器实现：

    /**
     * 具体迭代器实现
     *
     * @author supanpan
     * @date 2023/12/21
     */
    public class ConcreteIterator<T> implements Iterator<T> {
      private List<T> list;// 存放元素的列表
      private int position;// 遍历元素的下标
    
      /**
       * 初始化迭代器
       *
       */
      public ConcreteIterator(List<T> list) {
        this.list = list;
        this.position = 0;
      }
    
      /**
       * 判断是否到集合的末尾
       *
       */
      @Override
      public boolean hasNext() {
        return position < list.size();
      }
    
      /**
       * 遍历（迭代）集合取出相应元素
       *
       */
      @Override
      public T next() {
        if (!hasNext()){
          throw new NoSuchElementException("没有元素了......");
        }
        T item = list.get(position);
        position++;
        return item;
      }
    }

**创建聚合对象：**

    /**
     * 聚合对象
     *
     * @author supanpan
     * @date 2023/12/21
     */
    public interface Aggregate<T> {
        /**
         * 生成一个用于遍历集合的迭代器
         *
         */
        Iterator<T> createIterator();
    }

**实现聚合对象接口，创建具体聚合对象：**

    /**
     * 具体聚合对象
     *
     * @author supanpan
     * @date 2023/12/21
     */
    public class ConcreteAggregate<T> implements Aggregate<T>{
        private List<T> items;
    
        public ConcreteAggregate() {
            this.items = new ArrayList<>();
        }
    
        /**
         * 向集合中添加元素
         * @param item 待添加元素
         */
        public void addItem(T item){
            items.add(item);
        }
    
        /**
         * 创建迭代器对象
         *
         * @return 具体的迭代器
         */
        @Override
        public Iterator<T> createIterator() {
    //         items.sort(null);// 在创建迭代器之前对集合中的元素进行排序操作
            return new ConcreteIterator<>(items);// 将聚合对象中的集合传递给具体迭代器对象，以供访问
        }
    }

**测试Main:**

    public class Main {
    
        public static void main(String[ ] args) {
    
            // 创建聚合对象
            ConcreteAggregate<String> aggregate = new ConcreteAggregate<>();
            // 向聚合对象中添加元素
            aggregate.addItem("Item 1");
            aggregate.addItem("Item 3");
            aggregate.addItem("Item 2");
            aggregate.addItem("Item 4");
    
            // 获取迭代器对象
            Iterator<String> iterator = aggregate.createIterator();
            // 使用迭代器模式进行遍历
            while (iterator.hasNext()){
                // 获取迭代器中的元素
                String item = iterator.next();
                System.out.println(item);
            }
        }
    }

### 2、书架实例

**创建聚合对象，用于将具体实现类中的集合通过迭代器进行返回**

    public interface Aggregate<T> {
        Iterator<T> iterator();
    }

**创建迭代器对象，用于访问聚合对象中的数组数据**

    public interface Iterator<T> {
        boolean hasNext();
        T next();
    }

**创建Book对象，Book对象用于获取书籍的名称**

    public class Book {
        private String name;// 书籍名称
    
        public Book(String name) {
            this.name = name;
        }
    
        public String getName() {
            return name;
        }
    }

**创建BookShelf对象，这个对象实现了Aggregate接口，拥有聚合元素的能力**

    public class BookShelf implements Aggregate{
        private ArrayList<Book> books;// 对比抽象实例中的数组，这里改成了存放书籍的列表
    
        /**
         * 初始化集合对象
         *
         * @param initialSize 集合大小
         */
        public BookShelf(int initialSize){
            this.books = new ArrayList<>(initialSize);
        }
        public Book getBookAt(int index) {
            return (Book) books.get(index);
        }
        public void appendBook(Book book) {
            books.add(book);
        }
        public int getLength() {
            return books.size();
        }
    
        /**
         * 将书籍数组返回给具体迭代器实现类，提供访问书籍数组的能力
         * 
         */
        public Iterator iterator() {
            return new BookShelfIterator(this);
        }
    }

**创建BookShelfIterator对象，这个对象实现了Iterator接口，拥有访问聚合对象中的数据能力**

    public class BookShelfIterator implements Iterator{
      private BookShelf bookShelf;
      private int index;
    
      /**
       * 初始化迭代器
       * @param bookShelf 聚合对象集合数据
       */
      public BookShelfIterator(BookShelf bookShelf) {
        this.bookShelf = bookShelf;
        this.index = 0;
      }
    
      @Override
      public boolean hasNext() {
        return index < bookShelf.getLength();
      }
    
      @Override
      public Object next() {
        Book item = bookShelf.getBookAt(index);
        index++;
        return item;
      }
    }

**测试类：**

    public class Main {
    
        public static void main(String[ ] args) {
    
            BookShelf bookShelf = new BookShelf(4);
            bookShelf.appendBook(new Book("时间简史"));
            bookShelf.appendBook(new Book("活着"));
            bookShelf.appendBook(new Book("百年孤独"));
            bookShelf.appendBook(new Book("1984"));
            bookShelf.appendBook(new Book("三体"));
            bookShelf.appendBook(new Book("围城"));
            bookShelf.appendBook(new Book("小王子"));
            bookShelf.appendBook(new Book("云边的小卖部"));
    
            Iterator it = bookShelf.iterator();
            while (it.hasNext()) {
                Book book = (Book)it.next();
                System.out.println(book.getName());
            }
        }
    }

### 3、通讯录案例

以下案例源自《Design Patterns》- Alexander Shvets 中的迭代器案例

**创建迭代器接口：**

    public interface ProfileIterator {
        /**
         * 判断是否还有下个元素
         *
         */
        boolean hasNext();
    
        /**
         * 获取下个元素
         * @return 档案信息
         */
        Profile getNext();
    
        /**
         * 重置操作
         */
        void reset();
    }
    

**创建聚合对象接口：**

    /**
     * 通用聚合对象：让迭代器对象与集合实体之间产生链接
     */
    public interface SocialNetwork {
        /**
         * 创建好友迭代器
         */
        ProfileIterator createFriendsIterator(String profileEmail);
    
        /**
         * 创建同事迭代器
         */
        ProfileIterator createCoworkersIterator(String profileEmail);
    }

**创建档案实体：**

    @Getter
    public class Profile {
        private String name;// 姓名
        private String email;// 邮箱
        private Map<String, List<String>> contacts = new HashMap<>();// 通讯录
        public Profile(String email, String name, String... contacts) {
            this.email = email;
            this.name = name;
    
            // Parse contact list from a set of "friend:email@gmail.com" pairs.
            for (String contact : contacts) {
    
                String[ ] parts = contact.split(":");
    
                String contactType = "friend", contactEmail;
                if (parts.length == 1) {
                    contactEmail = parts[0];
                }
                else {
                    contactType = parts[0];
                    contactEmail = parts[1];
                }
                if (!this.contacts.containsKey(contactType)) {
                    this.contacts.put(contactType, new ArrayList<>());
                }
                this.contacts.get(contactType).add(contactEmail);
            }
        }
    
        public List<String> getContacts(String contactType) {
            if (!this.contacts.containsKey(contactType)) {
                this.contacts.put(contactType, new ArrayList<>());
            }
            return contacts.get(contactType);
        }
    
    
    }

**创建迭代器实现类：**

    /**
     * Facebook迭代器实现类
     *
     * @author supanpan
     * @date 2023/12/22
     */
    public class FacebookIterator implements ProfileIterator{
        private Facebook facebook;
        private String type;
        private String email;// 用户邮箱
        private int currentPosition = 0;// 遍历指针下标
        private List<String> emails = new ArrayList<>();// 邮箱列表
        private List<Profile> profiles = new ArrayList<>();// 档案列表
    
    
        /**
         * 初始化构造方法
         * @param facebook
         * @param type
         * @param email
         */
        public FacebookIterator(Facebook facebook, String type, String email) {
            this.facebook = facebook;
            this.type = type;
            this.email = email;
        }
    
        /**
         * 模拟网络延迟
         */
        private void lazyLoad() {
            if (emails.isEmpty()) {
                List<String> profiles = facebook.requestProfileFriendsFromFacebook(this.email, this.type);
                for (String profile : profiles) {
                    this.emails.add(profile);
                    this.profiles.add(null);
                }
            }
        }
    
        /**
         * 判断是否还有下一个元素
         *
         */
        @Override
        public boolean hasNext() {
            // 模拟延迟获取数据
            lazyLoad();
            return currentPosition < emails.size();
        }
    
        /**
         * 获取集合内的元素
         *
         *
         */
        @Override
        public Profile getNext() {
            if (!hasNext()){
                return null;// 遍历结束了...
            }
            // 获取当前邮箱
            String friendEmail = emails.get(currentPosition);
            // 获取当前档案信息
            Profile friendProfile = profiles.get(currentPosition);
            if (friendProfile == null) {
                friendProfile = facebook.requestProfileFromFacebook(friendEmail);
                profiles.set(currentPosition, friendProfile);
            }
            currentPosition++;// 将遍历内部集合的指针元素向后移动
            return friendProfile;
        }
    
        /**
         * 将遍历内部集合的指针元素归零
         */
        @Override
        public void reset() {
            currentPosition = 0;
        }
    }
    
    
    public class LinkedInIterator implements ProfileIterator {
      private LinkedIn linkedIn;
      private String type;
      private String email;
      private int currentPosition = 0;
      private List<String> emails = new ArrayList<>();
      private List<Profile> contacts = new ArrayList<>();
    
      public LinkedInIterator(LinkedIn linkedIn, String type, String email) {
        this.linkedIn = linkedIn;
        this.type = type;
        this.email = email;
      }
    
      private void lazyLoad() {
        if (emails.size() == 0) {
          List<String> profiles = linkedIn.requestRelatedContactsFromLinkedInAPI(this.email, this.type);
          for (String profile : profiles) {
            this.emails.add(profile);
            this.contacts.add(null);
          }
        }
      }
    
      @Override
      public boolean hasNext() {
        lazyLoad();
        return currentPosition < emails.size();
      }
    
      @Override
      public Profile getNext() {
        if (!hasNext()) {
          return null;
        }
    
        String friendEmail = emails.get(currentPosition);
        Profile friendContact = contacts.get(currentPosition);
        if (friendContact == null) {
          friendContact = linkedIn.requestContactInfoFromLinkedInAPI(friendEmail);
          contacts.set(currentPosition, friendContact);
        }
        currentPosition++;
        return friendContact;
      }
    
      @Override
      public void reset() {
        currentPosition = 0;
      }
    }

**创建聚合实现类：**

    public class Facebook implements SocialNetwork {
        private List<Profile> profiles;// 档案列表，这个集合将来会通过迭代器暴露给外部使用
    
        /**
         * 初始化构造方法
         *
         */
        public Facebook(List<Profile> cache) {
            if (cache != null){
                this.profiles = cache;
            }else {
                this.profiles = new ArrayList<>();// 滞空操作
            }
        }
    
        /**
         * 通过邮箱获取档案信息
         */
        public Profile requestProfileFromFacebook(String profileEmail) {
            // 模拟网络延迟
            simulateNetworkLatency();
            System.out.println("Facebook: Loading profile '" + profileEmail + "' over the network...");
            // 最终返回数据
            return findProfile(profileEmail);
        }
    
        /**
         * 通过邮箱获取档案的好友列表
         */
        public List<String> requestProfileFriendsFromFacebook(String profileEmail, String contactType) {
            /// 模拟网络延迟
            simulateNetworkLatency();
            System.out.println("Facebook: Loading '" + contactType + "' list of '" + profileEmail + "' over the network...");
    
            // 最终返回数据
            Profile profile = findProfile(profileEmail);
            if (profile != null) {
                return profile.getContacts(contactType);
            }
            return null;
        }
    
        /**
         * 在内部封装遍历集合的操作
         */
        private Profile findProfile(String profileEmail){
            for (Profile profile : profiles){
                if (profile.getEmail().equals(profileEmail)){
                    return profile;
                }
            }
            return null;
        }
        /**
         * 线程睡眠2s
         */
        private void simulateNetworkLatency(){
            try{
                Thread.sleep(2000);
            }catch (InterruptedException ex){
                ex.printStackTrace();
            }
        }
        @Override
        public ProfileIterator createFriendsIterator(String profileEmail) {
            return new FacebookIterator(this, "friends", profileEmail);
        }
    
        @Override
        public ProfileIterator createCoworkersIterator(String profileEmail) {
            return new FacebookIterator(this, "coworkers", profileEmail);
        }
    }
    
    
    public class LinkedIn implements SocialNetwork {
      private List<Profile> contacts;
    
      public LinkedIn(List<Profile> cache) {
        if (cache != null) {
          this.contacts = cache;
        } else {
          this.contacts = new ArrayList<>();
        }
      }
    
      public Profile requestContactInfoFromLinkedInAPI(String profileEmail) {
        // 模拟网络请求
        simulateNetworkLatency();
        System.out.println("LinkedIn: Loading profile '" + profileEmail + "' over the network...");
    
        // 返回数据
        return findContact(profileEmail);
      }
    
      public List<String> requestRelatedContactsFromLinkedInAPI(String profileEmail, String contactType) {
        // 模拟网路请求
        simulateNetworkLatency();
        System.out.println("LinkedIn: Loading '" + contactType + "' list of '" + profileEmail + "' over the network...");
    
        // 返回数据
        Profile profile = findContact(profileEmail);
        if (profile != null) {
          return profile.getContacts(contactType);
        }
        return null;
      }
    
      private Profile findContact(String profileEmail) {
        for (Profile profile : contacts) {
          if (profile.getEmail().equals(profileEmail)) {
            return profile;
          }
        }
        return null;
      }
    
      private void simulateNetworkLatency() {
        try {
          Thread.sleep(2500);
        } catch (InterruptedException ex) {
          ex.printStackTrace();
        }
      }
    
      @Override
      public ProfileIterator createFriendsIterator(String profileEmail) {
        return new LinkedInIterator(this, "friends", profileEmail);
      }
    
      @Override
      public ProfileIterator createCoworkersIterator(String profileEmail) {
        return new LinkedInIterator(this, "coworkers", profileEmail);
      }
    }

通过组合的形式，将迭代器和聚合对象整合到同一应用中

    public class SocialSpammer {
        // 将聚合对象和迭代器对象组合到当前应用中
        public SocialNetwork network;
        public ProfileIterator iterator;
    
        public SocialSpammer(SocialNetwork network) {
            this.network = network;
        }
    
    
        public void sendSpamToFriends(String profileEmail, String message) {
            System.out.println("\nIterating over friends...\n");
            iterator = network.createFriendsIterator(profileEmail);
            while (iterator.hasNext()) {
                Profile profile = iterator.getNext();
                sendMessage(profile.getEmail(), message);
            }
        }
    
        public void sendSpamToCoworkers(String profileEmail, String message) {
            System.out.println("\nIterating over coworkers...\n");
            iterator = network.createCoworkersIterator(profileEmail);
            while (iterator.hasNext()) {
                Profile profile = iterator.getNext();
                sendMessage(profile.getEmail(), message);
            }
        }
    
        public void sendMessage(String email, String message) {
            System.out.println("Sent message to: '" + email + "'. Message body: '" + message + "'");
        }
    }

创建Client代码：

    public class Client {
        public static Scanner scanner = new Scanner(System.in);
    
    
        public static void main(String[ ] args) {
    
            System.out.println("Please specify social network to target spam tool (default:Facebook):");
            System.out.println("1. Facebook");
            System.out.println("2. LinkedIn");
            String choice = scanner.nextLine();
    
            SocialNetwork network;
            if (choice.equals("2")) {
                network = new LinkedIn(createTestProfiles());
            }
            else {
                network = new Facebook(createTestProfiles());
            }
    
            SocialSpammer spammer = new SocialSpammer(network);
            spammer.sendSpamToFriends("anna.smith@bing.com",
                    "Hey! This is Anna's friend Josh. Can you do me a favor and like this post [link]?");
            spammer.sendSpamToCoworkers("anna.smith@bing.com",
                    "Hey! This is Anna's boss Jason. Anna told me you would be interested in [link].");
        }
    
        public static List<Profile> createTestProfiles() {
            List<Profile> data = new ArrayList<Profile>();
            data.add(new Profile("anna.smith@bing.com", "Anna Smith", "friends:mad_max@ya.com", "friends:catwoman@yahoo.com", "coworkers:sam@amazon.com"));
            data.add(new Profile("mad_max@ya.com", "Maximilian", "friends:anna.smith@bing.com", "coworkers:sam@amazon.com"));
            data.add(new Profile("bill@microsoft.eu", "Billie", "coworkers:avanger@ukr.net"));
            data.add(new Profile("avanger@ukr.net", "John Day", "coworkers:bill@microsoft.eu"));
            data.add(new Profile("sam@amazon.com", "Sam Kitting", "coworkers:anna.smith@bing.com", "coworkers:mad_max@ya.com", "friends:catwoman@yahoo.com"));
            data.add(new Profile("catwoman@yahoo.com", "Liza", "friends:anna.smith@bing.com", "friends:sam@amazon.com"));
            return data;
        }
    }

### 4、Iterator总结

通过以上两个实例，在每个实例中，都出现了相对重要的接口和实现类，这四个关键角色，分别是Iterator（迭代器）、ConcreteIterator（具体的迭代器）、Aggregate（聚合对象）和ConcreteAggregate（具体的聚合对象）。

1.  ==**Iterator（迭代器）**==：
    
    *   Iterator是一个接口，它定义了在集合对象上进行迭代的方法
        
        *   hasNext()用于检查是否还有下一个元素
            
        *   next()用于获取下一个元素。
            
2.  ==**ConcreteIterator（具体的迭代器）**==：
    
    *   ConcreteIterator是Iterator接口的具体实现，它持有对应的集合对象，并且在内部实现了迭代逻辑。
        
    *   具体的迭代器类通常会包含一个指向当前元素的游标，并且实现了Iterator接口中定义的方法。
        
3.  ==**Aggregate（聚合对象）**==：
    
    *   Aggregate是一个接口，它定义了创建迭代器对象的方法，例如createIterator()
        
    *   聚合对象是包含一组元素的对象，它通常会提供一种方式来获取迭代器对象，使得外部客户端可以通过迭代器遍历聚合对象中的元素。
        
4.  ==**ConcreteAggregate（具体的聚合对象）**==：
    
    *   ConcreteAggregate是Aggregate接口的具体实现，它实现了创建迭代器对象的方法，并且通常会包含一个内部集合来存储元素。
        
    *   具体的聚合对象类会将迭代器对象的创建委托给具体的迭代器类。
        

下面是Iterator实现的类图：

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLw4G0Xpmqpz2/img/0e418260-dd24-46da-b0ed-0c2b15e85bcd.png)