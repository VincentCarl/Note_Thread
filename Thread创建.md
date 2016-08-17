**在Java中一般线程的创建有两种方式:**  


- '继承'Thread类，重写run方法  
- 实现Runnable接口，重写run方法  
- run()方法内部为线程的生命周期，run()方法结束那么线程生命周期结束  
- 启动线程一律使用Thread对象的start()方法

####案例继承Thread  
```Java
class MyThread extends Thread{
  @Override
  public void run(){
   for(int i=0;i<10;i++){
    try{
       Thread.sleep();
    }
    catch(InterruptedException e){
       e.printStackTrace();
    }
   }
  }
}
  public void button(View view){
    MyThread mt = new MyThread();
    mt.start();
 }
```
####案例实现Runable
```Java
class MyThewad implements Runable{
  @Override
  public void run(){
   for(int i=0;i<10;i++){
    try{
       Thread.sleep();
    }
    catch(InterruptedException e){
       e.printStackTrace();
    }
   }
  }
}
public void button2(View view){
  MyThread mt = new MyThread();
  Thread thread = new Thread(mt);
  mt.start();
}
```


---
__在Java中线程对象为Thread,因此Thread有很多方法便于我们管理线程__
- Thraed.sleep(毫秒)阻塞且可以自动唤醒
- Thread.currentThread()获得当前线程对象



