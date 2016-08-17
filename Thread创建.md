**在Java中一般线程的创建有两种方式:**  


- '继承'Thread类，重写run方法  
- 实现Runnable接口，重写run方法  
- run()方法内部为线程的生命周期，run()方法结束那么线程生命周期结束  
- 启动线程一律使用Thread对象的start()方法

