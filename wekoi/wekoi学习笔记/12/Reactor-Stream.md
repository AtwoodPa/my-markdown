# Reactor-Stream

*   官方文档
    
    *   https://www.reactive-streams.org/
        
    *   https://projectreactor.io/
        
*   Reactive-Stream
    
    *   是JVM面向流的库的标准和规范
        
        *   处理可能无限数量的元素
            
        *   有序
            
        *   在组件之间异步传递元素
            
        *   强制性**非阻塞，背压模式**
            

**如何构造响应式系统**

*   依靠**消息**来进行构建
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7zGv8yMOako/img/26e8abda-0153-423a-8822-81229e99b16a.png)

*   响应式编程/声明式编程
    
    *   声明流、说清楚要干什么、最终结果是怎么样的
        
*   响应式编程模型
    

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7zGv8yMOako/img/e1fbd9bc-abaf-450d-af9a-b2e2c2f61a3e.png)

*   响应式编程的特点
    
    *   异步
        
    *   基于事件
        
    *   基于流
        
    *   响应式编程的核心是流
        

方法调用流程：

*   传递参数
    
*   业务处理
    
*   返回结果
    

> 万物皆是数据 Data is everything

## 发布订阅模式

*   **Publisher（发布者）**：
    
    *   发布者发布数据，发布的数据将被放置缓冲区内等待订阅者订阅
        
    *   发布的所有数据，都被存到了内部的buffer区域
        
    *   Publisher 是用于产生事件流的接口，可以发出一系列的数据元素、错误信息或者完成信号。
        
    *   Publisher 接口定义了 subscribe(Subscriber<? super T> subscriber) 方法，用于订阅 Subscriber。
        
*   **Subscriber（订阅者）**：
    
    *   订阅者订阅发布者的数据，订阅者接收数据并进行处理
        
    *   Subscriber 是订阅 Publisher 的对象，它负责处理从 Publisher 发出的事件。
        
    *   Subscriber 接口定义了四个方法：
        
        *   void onSubscribe(Subscription subscription): 当订阅成功时调用，传递一个 Subscription 对象，用于管理订阅关系。
            
        *   void onNext(T item): 处理从 Publisher 收到的下一个元素。
            
        *   void onError(Throwable throwable): 处理错误信息。
            
        *   void onComplete(): 处理完成信号。
            
*   **Subscription（订阅关系）**：
    
    *   Subscription 对象用于表示订阅关系，并允许订阅者在需要时请求或取消元素。
        
    *   Subscriber 在 onSubscribe(Subscription subscription) 方法中收到 Subscription 对象，通过该对象可以管理订阅关系。
        
*   **Flow 类**：
    
    *   Flow 类是 Java Flow API 的工具类，提供了一些静态方法用于创建发布者和订阅者，以及一些辅助方法。
        
*   **SubmissionPublisher 类**：
    
    *   SubmissionPublisher 是 Publisher 的默认实现，它实现了流的发布功能，并且可以通过调用 subscribe(Subscriber<? super T> subscriber) 方法进行订阅。
        

**处理数据流程如下：**

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7zGv8yMOako/img/754a2b55-a115-46ce-8042-b30be7a5af3d.png)

### 发布&订阅

以下是一个简单的发布订阅模式的实现

    /**
     * 发布者：源源不断的发送数据
     * 订阅者：源源不断的接受数据
     *
     * 响应式编程（发布订阅） = 异步 + 缓冲区处理
     *
     * @author supanpan
     * @date 2024/01/04
     */
    public class FlowDemo {
    
        public static void main(String[ ] args) {
    
            // 1、定义一个发布者； 发布数据
            var publisher = new SubmissionPublisher<String>();
    
    
            // 2、定义一个订阅者； 接收数据
            Subscriber<Object> subscriber = new Subscriber<>() {
                private Subscription subscription;
    
                @Override
                public void onSubscribe(Subscription subscription) {
                    System.out.println(Thread.currentThread() + "订阅者订阅了 => " + subscription);
                    // 订阅者和发布者建立关系的时候，调用此方法
                    // subscription.request(10); // 一次性请求10个数据
                    // subscription.request(Long.MAX_VALUE); // 请求所有数据
                    this.subscription = subscription;
                    // 请求一个数据
                    // this.subscription.request(Long.MAX_VALUE);// 一开始将所有数据都请求下来
                    this.subscription.request(1);// 一开始只请求一个数据
                }
    
                @Override
                public void onNext(Object item) {
                    System.out.println(Thread.currentThread() + "订阅者接收到数据：" + item);
    
                    // 如果数据是p - 5，取消订阅
                    if (item.equals("p - 5")) {
                        // 取消订阅
                        subscription.cancel();
                    }else {
                        // 一直不断的请求数据
                        SleepUtils.sleep(1);
                        subscription.request(1);
                    }
                }
                @Override
                public void onError(Throwable throwable) {
                    System.out.println(Thread.currentThread() + "订阅者接收数据出现异常：" + throwable);
                }
                @Override
                public void onComplete() {
                    System.out.println(Thread.currentThread() + "订阅者接收数据完成");
                }
            };
    
            // 3、绑定发布者和订阅者【必须先进行绑定，才可以发布数据】
            publisher.subscribe(subscriber);
            // 4、发布者发布数据
            for (int i = 0; i < 10; i++) {
                if (i >= 9) {
                    // publisher.closeExceptionally(new RuntimeException("！！！有异常，关闭发布者，不干了！！！"));
                } else {
                    publisher.submit("p - " + i);
                }
                // publisher 发布的所有数据，都被存到了内部的buffer区域
            }
            // 5、结束后，关闭发布者
            publisher.close();
            SleepUtils.sleep(20);// 主线程若结束，发布者和订阅者都会结束
        }
    }

### 处理器

**加入处理器之后的处理流程**

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7zGv8yMOako/img/7562e719-05d2-40d6-89a8-786fa982a929.png)

1、实现Processor接口，自定义处理器

    /**
     * 自定义处理器
     * 1、实现Processor接口
     * 2、定义输入和输出的数据类型，即泛型
     * 3、定义一个发布者，发布数据
     * 4、定义一个订阅者，接收数据
     * 5、建立订阅关系，发布者&处理器&订阅者 - 职责链模式
     * 6、发布数据
     */
    
    public class ProcessorDemo {
      static class MyProcessor extends SubmissionPublisher<String> implements Flow.Processor<String, String> {
        private Flow.Subscription subscription;
        @Override
        public void onSubscribe(Flow.Subscription subscription) {
          System.out.println("processor订阅绑定完成......");
          this.subscription = subscription;
          // 向上游请求数据
          subscription.request(1);
        }
    
        /**
         * 数据到达时，会调用此方法
         * @param item the item
         */
        @Override
        public void onNext(String item) {
          System.out.println("拿到数据 => " + item);
          // 对流中的数据进行处理
          item += " - New!New!New!";
          // 处理完成后再向下游重新发布
          submit(item);// 此时流中的数据是进行处理之后的数据
          // 向上游再次请求数据
          subscription.request(1);
        }
    
        @Override
        public void onError(Throwable throwable) {
    
        }
    
        @Override
        public void onComplete() {
    
        }
      }
    
    
    
      public static void main(String[ ] args) {
    
        SubmissionPublisher<String> publisher = new SubmissionPublisher<String>();
    
        Flow.Subscriber<String> subscriber = new Flow.Subscriber<String>() {
          private Flow.Subscription subscription;
    
          @Override
          public void onSubscribe(Flow.Subscription subscription) {
            System.out.println(Thread.currentThread() + "订阅者订阅了 => " + subscription);
            // 订阅者和发布者建立关系的时候，调用此方法
            // subscription.request(10); // 一次性请求10个数据
            // subscription.request(Long.MAX_VALUE); // 请求所有数据
            this.subscription = subscription;
            // 请求一个数据
            // this.subscription.request(Long.MAX_VALUE);// 一开始将所有数据都请求下来
            this.subscription.request(1);// 一开始只请求一个数据
          }
    
          @Override
          public void onNext(String item) {
            System.out.println(Thread.currentThread() + "订阅者接收到数据：" + item);
    
            // 如果数据是p - 5，取消订阅
            if (item.equals("p - 5")) {
              // 取消订阅
              subscription.cancel();
            }else {
              // 一直不断的请求数据
              SleepUtils.sleep(1);
              subscription.request(1);
            }
          }
          @Override
          public void onError(Throwable throwable) {
            System.out.println(Thread.currentThread() + "订阅者接收数据出现异常：" + throwable);
          }
          @Override
          public void onComplete() {
            System.out.println(Thread.currentThread() + "订阅者接收数据完成");
          }
        };
        MyProcessor processor1 = new MyProcessor();
        MyProcessor processor2 = new MyProcessor();
        MyProcessor processor3 = new MyProcessor();
    
        // 加了中间操作之后的绑定流程：publisher -> processor（存在多个处理器，按照先后顺序进行绑定） -> subscriber
        publisher.subscribe(processor1);// 此时的处理器相当于 订阅者
        processor1.subscribe(processor2);// 此时的处理器相当于 发布者
        processor2.subscribe(processor3);//
        processor3.subscribe(subscriber);// 链表关系绑定责任链
        for (int i = 0; i < 10; i++) {
          if (i >= 9) {
            // publisher.closeExceptionally(new RuntimeException("！！！有异常，关闭发布者，不干了！！！"));
          } else {
            publisher.submit("p - " + i);
          }
          // publisher 发布的所有数据，都被存到了内部的buffer区域
        }
        publisher.close();
      }
    
    }
    

### 响应式编程总结

*   底层
    
    *   基于数据缓冲队列 + 消息驱动模型 + 异步回调机制
        
*   编码
    
    *   流式编程 + 链式调用 + 声明式API
        
*   效果
    
    *   优雅全异步 + 消息实时处理 + 高吞吐量 + 占用少量内存