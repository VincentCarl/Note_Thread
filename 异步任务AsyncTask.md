##AsyncTask
---
**官方文档：**  
```Java
public abstract class AsyncTask    
extends Object 

java.lang.Object   
android.os.AsyncTask<Params, Progress, Result>
```
---
AsyncTask enables proper and easy use of the UI thread. This class allows to perform background operations and publish results on the UI thread without having to manipulate threads and/or handlers.

AsyncTask is designed to be a helper class around Thread and Handler and does not constitute a generic threading framework. AsyncTasks should ideally be used for short operations (a few seconds at the most.) If you need to keep threads running for long periods of time, it is highly recommended you use the various APIs provided by the java.util.concurrent package such as Executor, ThreadPoolExecutor and FutureTask.

An asynchronous task is defined by a computation that runs on a background thread and whose result is published on the UI thread. An asynchronous task is defined by 3 generic types, called Params, Progress and Result, and 4 steps, called onPreExecute, doInBackground, onProgressUpdate and onPostExecute.

大意:    

*   异步任务能够恰当、容易的使用UI线程，这个类允许执行后台的操作并且在UI线程发布结果，而不必操作thread或者handler。  
*   异步任务被设计成一个Thread和Handler的帮助类，而不是构成一个类线程框架。异步任务应该完美的被使用于简短的操作（最多几秒）!如果你需要保持线程长时间的运行，应该强烈推荐使用各种Java提供的各种API，比如线程池。   
*   一个异步任务是由运行在后台的计算和发布在UI线程上的结果定义的，异步任务由3个参数组成:Params,Progress,Result。4个步骤:onPreExecute, doInBackground, onProgressUpdate and onPostExecute.


----  
   AsyncTask是Android 1.5 Cubake加入的用于实现异步操作的一个类，在此之前只能用Java SE库中的Thread来实现多线程异步，AsyncTask是Android平台自己的异步工具，融入了Android平台的特性，让异步操作更加的安全，方便和实用。实质上它也是对Java SE库中Thread的一个封装，加上了平台相关的特性，所以对于所有的多线程异步都强烈推荐使用AsyncTask，因为它考虑，也融入了Android平台的特性，更加的安全和高效。

AsyncTask可以方便的执行异步操作（doInBackground)，又能方便的与主线程进行通信，它本身又有良好的封装性，可以进行取消操作（cancel())。

----
##[案例]  
```Java 
 import android.os.AsyncTask;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.ProgressBar;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    ProgressBar progressBar1;
    TextView textView1;
    MyAsyncTask myAsyncTask;
    //AsyncTask<Params, Progress, Result>
    //Params启动任务执行的输入参数，比如HTTP请求的URL
    //Progress 后台任务执行的百分比。
    //Result 后台执行任务最终返回的结果，比如String。
    class MyAsyncTask extends AsyncTask<Integer, Integer, String> {
        //开始执行前【主线程】【初始工作，因为在UI主线程】
        //只会执行一次
        @Override
        protected void onPreExecute() {
            textView1.setText("0%");
            progressBar1.setMax(100);
            progressBar1.setProgress(0);
        }

        //处理结果的地方【主线程】，一般用于更新UI
        //参数：为执行结果
        //只会执行一次
        @Override
        protected void onPostExecute(String description) {
            Toast.makeText(MainActivity.this, description, Toast.LENGTH_LONG).show();
            textView1.setText("0%");
            progressBar1.setMax(100);
            progressBar1.setProgress(0);
        }

        //后台执行，比较耗时的操作都可以放在这里。
        //这里是【次线程】
        //参数:线程开始执行时候的参数，对应于【AsyncTask<Params, Progress, Result>中的Params】
        //返回：对应于【AsyncTask<Params, Progress, Result>中的Result,主要提供给onPostExecute使用】
        @Override
        protected String doInBackground(Integer... params) {
            for (int i = 0; i < params[0]; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //publicProgress(Progress…)来更新任务的进度。
                publishProgress(i);
            }
            return "执行完毕!!";
        }

        //可以使用进度条增加用户体验度。 此方法在主线程执行，用于显示任务执行的进度。
        //这里是【主线程】
        ////参数:线程开始执行时候的参数，对应于【AsyncTask<Params, Progress, Result>中的Params】
        @Override
        protected void onProgressUpdate(Integer... values) {
            textView1.setText(values[0] + "%");
            progressBar1.setProgress(values[0]);
        }

        //用户调用取消时，要做的操作【主线程】
        //只会执行一次
        @Override
        protected void onCancelled(String description) {
        }

        //用户调用取消时，要做的操作【主线程】
        //只会执行一次
        @Override
        protected void onCancelled() {
        }
    }

    public void btn1(View view) {
        //实例化异步任务
        myAsyncTask = new MyAsyncTask();
        //开始执行异步任务
        myAsyncTask.execute(100);

    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        progressBar1 = (ProgressBar) findViewById(R.id.progressBar1);
        textView1 = (TextView) findViewById(R.id.textView1);
    }
}
```
