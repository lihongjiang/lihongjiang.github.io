---
layout: post
title: 消息系统模型
category: 知识库
tags: handler机制
keywords: handler机制
description: handler机制
---


## 0 参考

[Android中消息系统模型和Handler Looper](http://www.cnblogs.com/kobe8/p/3989773.html)

[Android Handler机制](http://blog.csdn.net/stonecao/article/details/6417364)


## 1 理论知识

1 handler消息机制是用于线程通信的，不是更新UI的。
2 主线程属于消费者，子线程属于生产者
<li>当消息队列已满，停止生产（子线程阻塞)
<li>当消息队列被掏空，停止消费（主线程阻塞）   
3 消息队列不是无限制的。最大50，android源码。
4 同类唤醒异常
5 那个线程调用prepare，就会关联一个looper对象

6 [java并发编程--互斥锁, 读写锁及条件](http://coolxing.iteye.com/blog/1236909) 

7 线程数据隔离
  ThreadLocal
  [正确理解ThreadLocal](http://www.iteye.com/topic/103804)

## 2 模拟代码

Handler类：

    public class Handler {

    Looper mLopper;

    MessageQueue queue;

    public Handler() {
        mLopper = Looper.myLooper();
        queue = mLopper.messageQueue;
    }

    public   void sendMessage(Message msg) {
       msg.target=this;
        this.queue.enqueueMessage(msg);
    }

    public void dispatchMessage(Message msg) {
        handleMessage(msg);
    }

    public void handleMessage(Message msg) {

    }

Looper类：

    public class Looper {

    public static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<>();
    public  MessageQueue messageQueue;

    private Looper() {
        messageQueue = new MessageQueue();
    }

    public static Looper myLooper() {
        return sThreadLocal.get();
    }

    public static void prepare() throws Exception {
        if (sThreadLocal.get() != null) {
            throw new Exception();
        }
        sThreadLocal.set(new Looper());
    }

    public  static  void looper() throws Exception {
        Looper mylooper = myLooper();
        if (mylooper==null){
            throw  new Exception("");
        }
        MessageQueue  queue=mylooper.messageQueue;
        for (;;){
            Message  msg=queue.next();
            if (msg==null){
                continue;
            }
            msg.target.dispatchMessage(msg);
        }
    }
    }

MessageQueue类

	public class MessageQueue {

    Message[] items;
    int putIndex;
    int takeIndex;

    int count;

    Lock lock;//互斥锁，2个条件同步
    Condition noempoty;
    Condition nofull;

    public MessageQueue() {
        this.items = new Message[50];
        lock = new ReentrantLock();
        noempoty=lock.newCondition();
        nofull=lock.newCondition();

    }

    //生产大于消费会覆盖
    public void enqueueMessage(Message msg) {

        try {
            lock.lock();
            while (count == items.length) {
                //子线程阻塞
                try {
                    nofull.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            this.items[putIndex] = msg;
            putIndex = (++putIndex == items.length) ? 0 : putIndex;

            count++;

            noempoty.signalAll();
        } finally {
            lock.unlock();
        }

    }


    public Message next() {

        Message msg = null;
        try {
            lock.lock();
            while (count == 0) {
                //主线程
                try {
                    noempoty.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            msg = items[takeIndex];
            items[takeIndex] = null;
            takeIndex = (++takeIndex == items.length) ? 0 : takeIndex;
            count--;
            //消费通知生产
            nofull.signalAll();
        } finally {
            lock.unlock();
        }
        return msg;
    }

Message类

	public class Message {
    Handler  target;//发送消息handler,保护属性
    int  what;
    Object  obj;
    public Message() {
    }
    }

测试类
	public class Test {

    public  static void main(String[] args) throws Exception {

        System.out.println("stat");
        Looper.prepare();

        final Handler handler=new Handler(){
            @Override
            public void handleMessage(Message msg) {
              System.out.println(msg.obj.toString());
            }
        };

        for (int i=0;i<10;i++){
            Thread  thread=new Thread(new Runnable() {
                @Override
                public void run() {

                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    Message  msg=new Message();
                    msg.what=10;
                    synchronized (UUID.class){
                        msg.obj=Thread.currentThread().getName()+
		UUID.randomUUID().toString();
                    }
                    handler.sendMessage(msg);

                }
            });
            thread.setName("thread"+i+"---->>");
            thread.start();
        }

        Looper.looper();

    }



