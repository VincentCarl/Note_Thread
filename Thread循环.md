#线程的循环
---
在某些情况下，需要让线程不用停止下来，或者延迟多少时间再运行线程。所以这样的场景可以使用java提供的Timer和TimeTask结合处理。

#Timer和TimeTask
---
* Timer是jdk中提供的一个定时器工具，使用的时候会在主线程之外起一个单独的线程执行指定的计划任务，可以指定执行一次或者反复执行多次。Timer是一个线程，使用schedule方法完成对TimerTask的调度，Timer对象调用一次schedule方法就是创建了一个线程，使用Timer的cancel()停止操作。Timer执行一次cancel()方法后，Timer线程都被终止。

**Timer的schedule方法有两种模式**

多少毫秒后执行一次  
void schedule(TimerTask, Date)  
void schedule(TimerTask, long)

每过多少毫秒执行一次  
void schedule(TimerTask, long, long)   
void schedule(TimerTask, Date, long)
  

**TimerTask是一个实现了Runnable接口的抽象类，代表一个可以被Timer执行的任务。**

#Time对象创建
---
```Java
Timer timer;
//直接实例化
timer = new Timer();
```

#TimeTask对象创建
```Java
//TimerTask是抽象类，需要继承TimerTask并且实现TimerTask的run方法
    class MyTimerTask extends TimerTask {
        @Override
        public void run() {

        //这里为一个新线程

        }
    }
```

#Timer对象执行TimeTask任务
---
```Java
    timer = new Timer();
    //延迟0秒，并且每过1000毫秒执行一次
    timer.schedule(new MyTimerTask(), 0, 1000);


    timer = new Timer();
    //延迟1000秒后执行
    timer.schedule(new MyTimerTask(), 1000);
```
##Timer线程结束【如果执行了每过N毫秒执行一次，需要调用cancel()方法结束线程】
---
```Java
   //timer对象线程结束
    timer.cancel();
    timer = null;
```

##案例（结合了android的handler）
---
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

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;


public class F extends Fragment{
  //碎片视图
  View view;
  //碎片中的文本对象
  TextView textView;
  //开始按钮
  Button button1;
  //结束按钮
  Button button2;
  //计时器对象
  Timer timer;

  //TimerTask是抽象类，需要继承TimerTask并实现TimerTask的run方法
  class MyTimerTask extends TimerTask{
    @Override
    public void run(){
      //执行getMyResult()方法获取当前时间
      String temp = getMyResult();
      //发送消息给UI线程的handler对象
       Message message = handler.obtainMessage();
      //设置消息数据
      message.obj = temp;
      //设置消息类型
      message.what = 1;
      //消息发送
      message.sendToTarget();
   }
  }
 //handler对象开始等待消息
   Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            //收到消息改变UI
            textView1.setText(msg.obj.toString());
        }
    };
  //业务逻辑
    public String getMyResult() {
        //获取当前日期
        Date date = new Date();
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String result = simpleDateFormat.format(date);
        return result;
    }

     public void btn1(View view) {
        //实例化timer对象
        timer = new Timer();
        //每过一秒钟执行一次MyTimerTask任务
        timer.schedule(new MyTimerTask(), 0, 1000);
        button1.setEnabled(false);
    }

    public void btn2(View view) {
        //timer对象线程结束
        timer.cancel();
        timer = null;
        button1.setEnabled(true);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        if (view == null) {
            view = inflater.inflate(R.layout.fragment_f2, container, false);
            textView1 = (TextView) view.findViewById(R.id.textView1);
            button1 = (Button) view.findViewById(R.id.button1);
            button1.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    btn1(v);
                }
            });
            button2 = (Button) view.findViewById(R.id.button2);
            button2.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    btn2(v);
                }
            });
        }
        return view;
    }
}
```




















