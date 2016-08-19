##线程同步【线程安全】
---   
* 线程同步，指的是多线程共享资源的时候，程序世界会出现错误，这种错误就是**线程不安全**。因此需要对线程进行同步处理。线程同步处理的手段是**使用锁机制**。
* 在Java中使用**synchronized关键字**，达到代码级别或者方法级别的同步。
在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问量。  

####synchronized方法级别，那么同步锁为当前方法所在的对象

```Java    
 public synchronized void server1(){

} 
   
```

####synchronized代码级别，那么同步锁为自己定义的对象  

```Java   
   public void server1(){
        //代码段级别同步，锁对象为任意对象
        //但是需要保证所有调用该代码段的线程 都看到是同一对象
        synchronized(object){
         //同步范围
        }
    
  }
   
```
---
##线程安全与线程不安全
---
####**首先明白线程的工作原理**：  
jvm有一个main   memory，而每个线程有自己的working   memory，一个线程对一个variable进行操作时，都要在自己的working   memory里面建立一个copy，操作完之后再写入main   memory。多个线程同时操作同一个variable，就可能会出现不可预知的结果。根据上面的解释，很容易想出相应的scenario。   
而用synchronized的关键是建立一个monitor，这个monitor可以是要修改的variable也可以其他你认为合适的object比如method，然后通过给这个monitor加锁来实现线程安全，每个线程在获得这个锁之后，要执行完   load到workingmemory   －>   use&assign   －>   store到mainmemory   的过程，才会释放它得到的锁。这样就**实现了所谓的线程安全**。

####**什么是线程安全以及线程安全是怎么工作的**：  
如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是**线程安全**的。即是，线程安全就是说多线程访问同一代码，不会产生不确定的结果。  

编写线程安全的代码是依靠线程同步。线程安全一般都涉及到synchronized 就是一段代码同时只能有一个线程来操作 不然中间过程可能会产生不可预制的结果。

####**线程不安全案例：**  
比如一个 ArrayList 类，在添加一个元素的时候，它可能会有两步来完成：1. 在 Items[Size] 的位置存放此元素；2. 增大 Size 的值。

在单线程运行的情况下，如果 Size = 0，添加一个元素后，此元素在位置 0，而且 Size=1； 而如果是在多线程情况下，比如有两个线程，线程 A 先将元素存放在位置 0。但是此时 CPU 调度线程A暂停，线程 B 得到运行的机会。线程B也向此 ArrayList 添加元素，因为此时 Size 仍然等于 0 （注意哦，我们假设的是添加一个元素是要两个步骤哦，而线程A仅仅完成了步骤1），所以线程B也将元素存放在位置0。然后线程A和线程B都继续运行，都增加 Size 的值。 那好，现在我们来看看 ArrayList 的情况，元素实际上只有一个，存放在位置 0，而 Size 却等于 2。这就是“线程不安全”了。  

##案例  
```Java  

import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;
import android.widget.TextView;
import java.util.Random;
import java.util.Timer;
import java.util.TimerTask;

public class F3 extends Fragment {
    //共享资源
    int count = 1000;
    //同步锁对象
    Object object = new Object();

    //四个变量记录四个线程所获得的资源
    int r1 = 0;
    int r2 = 0;
    int r3 = 0;
    int r4 = 0;


    //UI
    View view;
    TextView textView1;
    TextView textView2;
    TextView textView3;
    TextView textView4;
    TextView textView5;
    Button button1;
    //消息对象
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                    textView1.setText(msg.obj.toString());
                    r1 = Integer.parseInt(msg.obj.toString());
                    break;
                case 2:
                    textView2.setText(msg.obj.toString());
                    r2 = Integer.parseInt(msg.obj.toString());
                    break;
                case 3:
                    textView3.setText(msg.obj.toString());
                    r3 = Integer.parseInt(msg.obj.toString());
                    break;
                case 4:
                    textView4.setText(msg.obj.toString());
                    r4 = Integer.parseInt(msg.obj.toString());
                    break;
            }


            textView5.setText(count + "");
            //消息对象观察资源，如果资源为0那么初始化
            if (count == 0) {
                timer1.cancel();
                timer2.cancel();
                timer3.cancel();
                timer4.cancel();
                button1.setEnabled(true);
                r1 = 0;
                r2 = 0;
                r3 = 0;
                r4 = 0;
                //重新补充资源
                count = 1000;
            }
        }
    };


    //四个线程
    Timer timer1;
    Timer timer2;
    Timer timer3;
    Timer timer4;

    //执行任务
    class MyTimeTask extends TimerTask {
        //消息类型值
        int what;
        //传递进来的每个任务的资源数量
        int rValue;

        public MyTimeTask(int what, int rValue) {
            this.what = what;
            this.rValue = rValue;

        }

        @Override
        public void run() {
            //同步任务
            synchronized (object) {

                //判断公共资源
                if (count > 0) {
                    //添加资源
                    rValue++;
                    //减少公共资源
                    count--;
                    //发送消息
                    Message message = handler.obtainMessage();
                    message.what = what;
                    message.obj = rValue;
                    message.sendToTarget();
                }
            }
        }
    }


    public void btn1(View view) {

        //实例化线程
        timer1 = new Timer();
        timer2 = new Timer();
        timer3 = new Timer();
        timer4 = new Timer();
        //随机数实例化，造成每个线程每次执行的毫秒数不一样
        Random random = new Random();
        int random1 = random.nextInt(100) + 100;
        int random2 = random.nextInt(100) + 100;
        int random3 = random.nextInt(100) + 100;
        int random4 = random.nextInt(100) + 100;

        //线程开始执行任务
        timer1.schedule(new MyTimeTask(1, r1), 0, random1);
        timer2.schedule(new MyTimeTask(2, r2), 0, random2);
        timer3.schedule(new MyTimeTask(3, r3), 0, random3);
        timer4.schedule(new MyTimeTask(4, r4), 0, random4);

        button1.setEnabled(false);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {

        if (view == null) {
            view = inflater.inflate(R.layout.fragment_f3, container, false);
            textView1 = (TextView) view.findViewById(R.id.textView1);
            textView2 = (TextView) view.findViewById(R.id.textView2);
            textView3 = (TextView) view.findViewById(R.id.textView3);
            textView4 = (TextView) view.findViewById(R.id.textView4);
            textView5 = (TextView) view.findViewById(R.id.textView5);
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

