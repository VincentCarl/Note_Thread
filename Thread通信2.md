##Handler主线程延时执行方案  
---
这里使用每个APP都会有的APP界面产品过场案例，在说明主线程延迟执行，却又不会阻塞主线程，主要使用的就是handler的消息机制中的post发送消息方式。   
####[案例]
```Java
public class WelcomeActivity extends AppCompatActivity {

    //在主线程创建Handler对象
    Handler handler=new Handler();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //使用postDelayed方法延迟5000执行，这样就不会阻塞主线程
        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                //这里依然是主线程
                //跳转其它Activity
                Intent intent=new Intent("com.hzj163.myhandler.MainActivity");
                startActivity(intent);
                finish();
            }
        },5000);
    }
}

```   


##Handler线程和线程之间的通信 

----   
在次线程中实例化Handler必须保证次线程开启了Looper否则会触发Can't create handler inside thread that has not called Looper.prepare()。  
	
```Java
import android.os.Handler;
import android.os.Looper;
import android.os.Message;
import android.support.v4.app.INotificationSideChannel;
import android.support.v7.app.ActionBar;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.SurfaceView;
import android.view.View;
import android.widget.TextView;

import java.util.Timer;
import java.util.TimerTask;
public class MainActivity extends AppCompatActivity {
    TextView textView2;
    int a = 900;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        textView2 = (TextView) findViewById(R.id.textView2);
        //启动次级线程1，准备等待接收消息
        new Thread(runnable1).start();
    }

    //主线程Handler对象
    Handler handlerMain = new Handler();
    //次线程Handler对象
    Handler handlerThread;
    //次线程1，负责接收消息
    //在次线程1，run方法内部实例化了Handler对象handlerThread
    Runnable runnable1 = new Runnable() {
        @Override
        public void run() {
            //Looper准备
            Looper.prepare();
            //实例化handlerThread，只有在run方法内部实例化的handlerThread才有效
            handlerThread = new Handler() {
                //回调handleMessage准备接受消息
                @Override
                public void handleMessage(Message msg) {
                    Log.i("hzj", "开始收消息thread:" + Thread.currentThread().getName());
                    final int temp = msg.arg1;
                    //使用主线程的handlerMain对象执行更新主线程UI
                    handlerMain.post(new Runnable() {
                        @Override
                        public void run() {
                            //更新UI
                            textView2.setText("a:=" + temp);
                        }
                    });
                }
            };
            //Looper开始循环
            Looper.loop();
        }
    };
    //线程2：主要负责发送消息，给线程1发送消息，因此需要使用handlerThread对象来发消息
    Runnable runnable2 = new Runnable() {
        @Override
        public void run() {
            Log.i("hzj", "开发发消息：thread:" + Thread.currentThread().getName());
            a=1000;
            //handlerThread创建消息对象，因为handlerThread在次线程1中
            Message message = handlerThread.obtainMessage();
            message.arg1 = a;
            //发送消息，因此线程1中的handlerThread能够收到消息，达到了线程2和线程1通信
            message.sendToTarget();
        }
    };

    public void btn2(View view) {
        //线程和线程之间的交互
        //开启线程2，开始向线程1发送消息
        new Thread(runnable2).start();
    }

    @Override
    protected void onDestroy() {

        //注意需要关闭次线程1的Looper
        if (handlerThread != null) {
            //关闭次线程1的Looper
            handlerThread.getLooper().quit();
        }
        super.onDestroy();
    }
}
```