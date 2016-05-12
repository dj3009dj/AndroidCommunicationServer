# AndroidCommunicationServer
<h>大纲</h>
 <ol>
 <li>学习Android如何与服务器连接，本文是个很好的例子。
 <li>使用ServerSocket建立Client-Server连接，实现简单的聊天室功能。
 <li>流程图如下所示：<br/>
  ![alt tag](https://github.com/dj3009dj/ImageRepository/blob/master/androidToServer/c-s.png)
 </ol>
 <h><b>服务器端</b></h><br/>
服务器端首先利用端口号开启ServerSocket。<br/>
<pre>ServerSocket ss = new ServerSocket(30000);</pre>
使用while循环为每一个client生成一个socket。每个socket开启一个新线程。<br/>
<pre>Socket s=ss.accept();
socketList.add(s);
new Thread(new MyServerThread(s)).start();</pre>
线程的作用是：获取client向服务器发的信息(getInputStream)，再向每一个客户端发送这条信息(getOutputStream)。<br/>
<pre>
//获取client发的信息
BufferedReader br=new BufferedReader(new InputStreamReader(s.getInputStream(),"utf-8"));
//写在run中,表示新线程执行的语句
  public void run(){
  String content;
  while((content=br.readLine())!=null)//循环方式不断从Socket中读取客户端发送的内容
     for(Iterator it=arrayList.iterator;it.hasNext();)//向每一个client发送这条信息
     {
       Socket socket=it.next();
       OutputStream os=socket.getOutputStream();
       os.write((content+"\n").getBytes("utf-8"));
     }
  }</pre>
至此，服务器的内容基本完善了，使用android studio似乎不能运行，这里使用了IDEA。</br>
 <h><b>客户端</b></h><br/>
 客户端启动新线程，建立网络连接，并且将信息发送到server端。先看网络连接：<br/>
 <pre>
 public void run{
 Socket ss=new Socket("192.168.1.1",30000);//随便写了个IP地址
 BufferedReader br=new BufferedReader(new InputStreamReader(ss.getInputStream()));//获取了服务器传来的消息
 new Thread(){//在这里又重新启动一个子线程
  public void run(){
    while((content=br.readLine())!=null){
    Message msg=new Message();
    msg.what=0x123;
    msg.obj=content;//这里将服务器传来的消息，讲给mHandler处理
    mHandler.sendMessage(msg);//mHandler是主线程中，用来改变聊天窗口的
    }
  }
 }.start();
 ....
 }</pre>
 让我们看看主进程的mHandler:</br>
 <pre>
 Handler mHandler=new Handler(){
 public void handleMessage(Message msg){
 if(msg==0x123){
 editText.append(msg.obj.toString);//将服务器传来的消息追加到聊天窗口中
 }
  }
 }</pre>
 其次是当点击发送按钮之后，将client端的信息发送到server端。利用新线程的recHandler,注意要加Looper。接着上面的public void run:<br/>
 <pre>....
 Looper.prepare()
 recHandler=new Handler(){
 public void handlerMessage(Message msg){
 if(msg.what==0x234){
 OutputStream os=ss.getOutputStream();
 os.write((msg.obj.toString+"\r\n").getBytes("utf-8"));}
 }}</pre>
