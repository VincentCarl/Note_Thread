#Java线程池【ExecutorService】
---
###线程池的作用：
* **线程池的作用就是限制系统中执行线程的数量。**

根据系统的环境情况，可以自动或手动设置线程数量，达到运行的最佳效果；少了浪费了系统资源，多了造成系统拥挤效率不高。用线程池控制线程数量，其他线程排队等候。一个任务执行完毕，再从队列的中取最前面的任务开始执行。若队列中没有等待进程，线程池的这一资源处于等待。当一个新任务需要运行时，如果线程池中有等待的工作线程，就可以开始运行了；否则进入等待队列。  

--- 

###为什么要用线程池：
* 减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务。
* 可以根据系统的承受能力，调整线程池中工作线程的数目。防止因为消耗过多的内存，而把服务器累趴下（每个线程需要大概1M的内存，线程开的越多，消耗的内存也就越大，最后死机）。

##Java里面的线程池
---
![](file:///C:/Users/旭军/Desktop/threadChi.png)  

  Java里面线程池的顶级接口是Executor,但严格意义上讲Exector并不是一个线程池，而只是一个执行线程的工具。真正的线程池是**ExecutorService**.  

#####****Java通过Executors提供四种线程池，分别是：****  
*  *****newCachedThreadPool创建一个可缓存的线程池。*****如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

* *****newFixedThreadPool 创建固定大小的线程池。*****每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

* *****newScheduledThreadPool 创建一个大小无限的线程池。*****此线程池支持定时以及周期性执行任务的需求。  


* *****newSingleThreadExecutor 创建一个单线程的线程池。*****这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。    

---
###创建线程池  
* Executor接口  
* ExecutorService接口  
* Executors类静态方法配置线程池  

---

##【案例】  
```Java
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;
import java.lang.ref.WeakReference;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
public class MainActivity extends AppCompatActivity {

    TextView textView1;
    TextView textView2;
    TextView textView3;
    TextView textView4;
    TextView textView5;
    TextView textView6;
    TextView textView7;
    TextView textView8;
    TextView textView9;
    TextView textView10;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView1 = (TextView) findViewById(R.id.textView1);
        textView2 = (TextView) findViewById(R.id.textView2);
        textView3 = (TextView) findViewById(R.id.textView3);
        textView4 = (TextView) findViewById(R.id.textView4);
        textView5 = (TextView) findViewById(R.id.textView5);
        textView6 = (TextView) findViewById(R.id.textView6);
        textView7 = (TextView) findViewById(R.id.textView7);
        textView8 = (TextView) findViewById(R.id.textView8);
        textView9 = (TextView) findViewById(R.id.textView9);
        textView10 = (TextView) findViewById(R.id.textView10);
    }

    //消息机制
    MyHandler handler = new MyHandler(new WeakReference<MainActivity>(MainActivity.this));
    static class MyHandler extends Handler {
        WeakReference<MainActivity> weakReference;

        public MyHandler(WeakReference<MainActivity> weakReference) {
            this.weakReference = weakReference;
        }

        @Override
        public void handleMessage(Message msg) {
            MainActivity temp = weakReference.get();

            switch (msg.what) {
                case 1:
                    temp.textView1.setText(msg.obj.toString());
                    break;
                case 2:
                    temp.textView2.setText(msg.obj.toString());
                    break;
                case 3:
                    temp.textView3.setText(msg.obj.toString());
                    break;
                case 4:
                    temp.textView4.setText(msg.obj.toString());
                    break;
                case 5:
                    temp.textView5.setText(msg.obj.toString());
                    break;
                case 6:
                    temp.textView6.setText(msg.obj.toString());
                    break;
                case 7:
                    temp.textView7.setText(msg.obj.toString());
                    break;
                case 8:
                    temp.textView8.setText(msg.obj.toString());
                    break;
                case 9:
                    temp.textView9.setText(msg.obj.toString());
                    break;
                case 10:
                    temp.textView10.setText(msg.obj.toString());
                    break;
            }
        }
    }

    //线程业务
    public void myServer(int what) {
        for (int i = 0; i < 10; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Message message = handler.obtainMessage();
            message.what = what;
            message.obj = i;
            message.sendToTarget();
        }
    }

    //1号线程
    Runnable runnable1 = new Runnable() {
        @Override
        public void run() {
            myServer(1);
        }
    };
    //2号线程
    Runnable runnable2 = new Runnable() {
        @Override
        public void run() {

            myServer(2);
        }
    };
    //3号线程
    Runnable runnable3 = new Runnable() {
        @Override
        public void run() {
            myServer(3);
        }
    };

    //4号线程
    Runnable runnable4 = new Runnable() {
        @Override
        public void run() {
            myServer(4);
        }
    };

    //5号线程
    Runnable runnable5 = new Runnable() {
        @Override
        public void run() {
            myServer(5);
        }
    };
    //6号线程
    Runnable runnable6 = new Runnable() {
        @Override
        public void run() {
            myServer(6);
        }
    };
    //7号线程
    Runnable runnable7 = new Runnable() {
        @Override
        public void run() {
            myServer(7);
        }
    };
    //8号线程
    Runnable runnable8 = new Runnable() {
        @Override
        public void run() {
            myServer(8);
        }
    };
    //9号线程
    Runnable runnable9 = new Runnable() {
        @Override
        public void run() {
            myServer(9);
        }
    };
    //10号线程
    Runnable runnable10 = new Runnable() {
        @Override
        public void run() {
            myServer(10);
        }
    };

    //普通线程开启方案
    public void btn1(View view) {
        new Thread(runnable1).start();
        new Thread(runnable2).start();
        new Thread(runnable3).start();
        new Thread(runnable4).start();
        new Thread(runnable5).start();
        new Thread(runnable6).start();
        new Thread(runnable7).start();
        new Thread(runnable8).start();
        new Thread(runnable9).start();
        new Thread(runnable10).start();
    }

    //创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。
    public void btn2(View view) {
        threadExecute(Executors.newSingleThreadExecutor());
    }

    //创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。
    public void btn3(View view) {
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        threadExecute(executorService);
    }
    //创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们
    public void btn4(View view) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        threadExecute(executorService);
    }

    //创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。
    public void btn5(View view) {
        ScheduledExecutorService executorService = Executors.newScheduledThreadPool(5);
        executorService.schedule(runnable1, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable2, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable3, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable4, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable5, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable6, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable7, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable8, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable9, 1000, TimeUnit.MILLISECONDS);
        executorService.schedule(runnable10, 1000, TimeUnit.MILLISECONDS);
    }


    //执行线程池
    public void threadExecute(ExecutorService executorService) {
        executorService.execute(runnable1);
        executorService.execute(runnable2);
        executorService.execute(runnable3);
        executorService.execute(runnable4);
        executorService.execute(runnable5);
        executorService.execute(runnable6);
        executorService.execute(runnable7);
        executorService.execute(runnable8);
        executorService.execute(runnable9);
        executorService.execute(runnable10);
    }
}

```
