##Android消息机制  
----
###不妨先瞅瞅这篇文章---[ 浅析Android中的消息机制](http://blog.csdn.net/liuhe688/article/details/6407225/ "CSDN")

Android中活动为UI主线程，不能执行耗时超出5秒的任务，并且所有View和ViewGroup都只能在UI主线程中运行。如果View或者ViewGroup在次线程中运行，将会抛出【Only the original thread that created a view hierarchy can touch its views】，那么android中会**通过消息机制来解决线程和线程之间的通信问题。**  

##线程与线程之间的通信--Android消息机制  
---
#####Android中消息机制由以下部分组成：
* Handler  
* Message  
* MessageQueue  
* Looper
![](/pic/android消息机制.png)  

##Handler  
---  
*      什么是handler？handler扮演了往MQ上添加消息和处理消息的角色（只处理由自己发出的消息），即通知MQ它要执行一个任务(sendMessage)，并在loop到自己的时候执行该任务(handleMessage)，整个过程是异步的。handler创建时会关联一个looper，默认的构造方法将关联当前线程的looper，不过这也是可以set的。
*   handler必须关联一个loop线程才能起作用，Android UI主线程以及关联了一个Loop线程，但是我们自定定义的线程必须开启Loop。  
##Handler拥有两个主要的特点:    


*  handler可以在任意线程发送消息，这些消息会被添加到关联的MQ上。  
![](/pic/handlerSend.png)
*  handler是在它关联的looper线程中处理消息的。  
![](/pic/handlerLoop.png)

####handler发送消息  
---
有了handler之后，我们就可以使用 post(Runnable), postAtTime(Runnable, long), postDelayed(Runnable, long), sendEmptyMessage(int), sendMessage(Message), sendMessageAtTime(Message, long)和sendMessageDelayed(Message, long)这些方法向MQ上发送消息了。光看这些API你可能会觉得handler能发两种消息，一种是Runnable对象，一种是message对象，这是直观的理解，但其实post发出的Runnable对象最后都被封装成message对象了。

###send方案发送消息【需要回调才能接收消息】  
---
* sendMessage() 立即发送Message到消息队列
* sendMessageAtFrontOfQueue() 立即发送Message到队列，而且是放在队列的最前面
* sendMessageAtTime() 设置时间，发送Message到队列
* sendMessageDelayed() 在延时若干毫秒后，发送Message到队列  

###post方案发送消息【直接绑定handler当前线程执行，需要Runnable对象】

---
* post() 立即发送Message到消息队列
* postAtFrontOfQueue() 立即发送Message到队列，而且是放在队列的最前面
* postAtTime() 设置时间，发送Message到队列
* postDelayed() 在延时若干毫秒后，发送Message到队列  

####send方案发送消息,不使用Message对象封装消息  
---  
```Java
 //Handler对象实例化
    Handler handler = new Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            Log.i("hzj", "收到消息："+msg.what);
        }
    };
    public void btn1(View view) {
        //发送消息，并没有显示的使用Message消息封装对象
        handler.sendEmptyMessage(1);
    }
```
####send方案发送消息,使用Message对象封装消息  
```Java
//Handler对象实例化
    Handler handler = new Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            Log.i("hzj", "收到消息："+msg.what);
            Log.i("hzj", "收到消息arg1："+msg.arg1);
            Log.i("hzj", "收到消息arg2："+msg.arg2);
        }
    };
    public void btn1(View view) {
        //发送消息，使用Message消息封装对象
        Message message=new Message();
        //消息类型
        message.what=1;
        //message可以携带两个简单的整数数据
        //消息数据1
        message.arg1=111;
        //消息数据2
        message.arg2=222;
        //发送消息
        handler.sendMessage(message);
    }
```
####send方案发送消息,使用Message对象封装消息，且发消息  
```Java

    //Handler对象实例化
    Handler handler = new Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            Log.i("hzj", "收到消息："+msg.what);
            Log.i("hzj", "收到消息arg1："+msg.arg1);
            Log.i("hzj", "收到消息arg2："+msg.arg2);
        }
    };
    public void btn1(View view) {
        //发送消息，使用Message对象发送消息
        //Message对象的获取为handler.obtainMessage();而不是直接new，这样效率会高一些，节约内存资源
        Message message=handler.obtainMessage();
        //消息类型
        message.what=1;
        //message可以携带两个简单的整数数据
        //消息数据1
        message.arg1=111;
        //消息数据2
        message.arg2=222;
        //发送消息,这里使用了message对象的sendToTarget();方法
        message.sendToTarget();
    }
```

####post方案发送消息  
```Java
public class F4 extends Fragment {
    Button button1;
    View view;
    public F4() {

    }
    //次线程
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            //由于该runnable对象使用handler.post(runnable);
            //又由于handler在UI主线程创建
            //因此runnable对象的run方法会在UI主线程执行
            Log.i("hzj", Thread.currentThread().getName() + "-----");
        }
    };
    //Handler对象实例化
    Handler handler = new Handler()
    {
        @Override
        public void handleMessage(Message msg) {
            //注意使用handler.post方式，这里获取不到消息
        }
    };
    public void btn1(View view) {
        //发送消息
        //post系列发送消息，直接就会在handler对象所在线程里面直接执行runnable对象的run方法
        //因此这里post的runnable对象里面的run方法实际上在UI主线程里面执行
        //这里也并不需要使用回调handleMessage方法获取消息对象，也获取不到
        handler.post(runnable);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        if (view == null) {
            view = inflater.inflate(R.layout.fragment_f4, container, false);
            button1 = (Button) view.findViewById(R.id.button1);
            button1.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    btn1(v);
                }
            });

        }
        return view;
    }
}
```
##Handler处理消息  
** 消息的处理是通过回调handleMessage(Message msg)完成，当然这里需要使用send方案发送消息，才能回调处理。 **    

```Java  
    //handler对象开始等待消息  
    Handler handler = new Handler() {  
        @Override  
        public void handleMessage(Message msg) {  
            //收到消息改变UI  
            textView1.setText(msg.obj.toString());
        }
    };
```

###次线程建立Handler对象  
```Java
import android.os.Handler;
import android.os.Looper;
public class Thread1 implements Runnable {
    Handler handler;
    @Override
    public void run() {
        //Looper准备
        Looper.prepare();
        //次线程实例化Handler对象，必须开启Looper
        //UI主线性实例化Handler对象不需要开启Looper,因为UI主线程以及开启了Looper
        handler = new Handler();
        //Looper循环
        Looper.loop();
    }
}
```

##更新到adt20的开发者们可能会在handler上发现这么一条警告：This Handler class should be static or leaks might occur 。
----

 首先在ADT 20 Changes我们可以找到这样一个变化：New Lint Checks: Look for handler leaks: This check makes sure that a handler inner class does not hold an implicit reference to its outer class.

  翻译过来就是，Lint会增加一个检查项目即：确保class内部的handler不含有外部类的隐式引用 。 同一个线程下的handler共享一个looper对象，消息中保留了对handler的引用，只要有消息在队列中，那么handler便无法被回收，如果handler不是static那么使用Handler的Service和Activity就也无法被回收。这就可能导致内存泄露。当然这通常不会发生，除非你发送了一个延时很长的消息。

  官方推荐将handler设为static类，并在里面使用弱引用WeakReference
  

 ##[案例]  

```Java  

import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import java.lang.ref.WeakReference;

public class F4 extends Fragment {
    Button button1;
    View view;
    public F4() {
    }
    //静态内部类Handler对象
    static class MyHandler extends Handler {
        //使用弱引用引用外部类
        WeakReference<F4> weakReference;

        //通过构造获取实例
        public MyHandler(WeakReference<F4> weakReference) {
            this.weakReference = weakReference;
        }
        @Override
        public void handleMessage(Message msg) {
            //更新主UI组件
            //使用弱引用对象的get方法获取外部类对象
            F4 temp = weakReference.get();
            temp.button1.setText("xxxx");
        }
    }
    //Handler对象实例化
    MyHandler handler = new MyHandler(new WeakReference<F4>(this));
    //发消息
    public void btn1(View view) {
        Message message = handler.obtainMessage();
        message.sendToTarget();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        if (view == null) {
            view = inflater.inflate(R.layout.fragment_f4, container, false);
            button1 = (Button) view.findViewById(R.id.button1);
            button1.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    btn1(v);
                }
            });
        }
        return view;
    }
}

```


##Message   
----  
 在整个消息处理机制中，message又叫task，封装了任务携带的信息和处理该任务的handler。message的用法比较简单。

* 尽管Message有public的默认构造方法，但是你应该通过Message.obtain()来从消息池中获得空消息对象，以节省资源。

* 如果你的message只需要携带简单的int信息，请优先使用Message.arg1和Message.arg2来传递信息，这比用Bundle更省内存

* 擅用message.what来标识信息，以便用不同方式处理message。  

##Looper  
---

 Looper的字面意思是“循环者”，它被设计用来使一个普通线程变成Looper线程。所谓Looper线程就是循环工作的线程。在程序开发中（尤其是GUI开发中），我们经常会需要一个线程不断循环，一旦有新任务则执行，执行完继续等待下一个任务，这就是Looper线程。使用Looper类创建Looper线程很简单：  

```Java  

public class LooperThread extends Thread {
    @Override
    public void run() {
        // 将当前线程初始化为Looper线程
        Looper.prepare();

        // ...其他处理，如实例化handler

        // 开始循环处理消息队列
        Looper.loop();
    }
}
```
---    
* 现在你的线程中有一个Looper对象，它的内部维护了一个消息队列MQ。注意，一个Thread只能有一个Looper对象

* prepare()其核心就是将looper对象定义为ThreadLocal。

* loop();调用loop方法后，Looper线程就开始真正工作了，它不断从自己的MQ中取出队头的消息(也叫任务)执行。


![](/pic/looper.png)  

----
####关于Looper总结几点：

* 每个线程有且最多只能有一个Looper对象，它是一个ThreadLocal
* Looper内部有一个消息队列，loop()方法调用后线程开始不断从队列中取出消息执行
* Looper使一个线程变成Looper线程。


---
##MessageQueue  

Message Queue（消息队列），但是MQ被封装到Looper里面了，我们不会直接与MQ打交道



















