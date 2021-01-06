#### 网络开发的经典模型——Echo程序

1. 服务器类：ServerSocket，主要工作在服务器端，用于接收用户的请求

    构造方法：public ServerSocket（int port）设置监听端口

    接收客户端连接：public Socket accept（）；

    取得客户端输出功能，Socket类定义方法 OutputStream getOutputStream()；

2. 客户端类：Socket，每一个连接到服务器上的用户都通过Socket

    构造方法：public Socket(String host, int port)；

    host表示主机的IP地址，如果是本机直接访问，则使用localhost（127.0.0.1）代替IP；

    得到输入数据：public InputStream getInputStream()；

    

**服务器端**

~~~java
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Scanner;
class EchoThread implements Runnable {          
    private Socket client;                                
    public EchoThread(Socket client) {           
        this.client = client;
    }
    @Override
    public void run() {    
        try {    
            // 每个线程对象取得各自Socket的输入流与输出流
            Scanner scan = new Scanner(client.getInputStream());
            PrintStream out = new PrintStream(client.getOutputStream());
            // 控制多次接收操作
            boolean flag = true;                     
            while (flag) {
                if (scan.hasNext()) {                    
                    String str = scan.next().trim();         
                    if (str.equalsIgnoreCase("byebye")) {
                        out.println("拜拜，下次再会！");
                        flag = false;                    
                    } else {                         
                        // 应该回应输入信息
                        out.println("ECHO : " + str);        
                    }
                }
            }
            scan.close();
            out.close();
            client.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
public class EchoServer {
    public static void main(String[] args) throws Exception {
        ServerSocket server = new ServerSocket(9999);
        boolean flag = true;                     
        while (flag) {                                
            Socket client = server.accept();            
            new Thread(new EchoThread(client)).start();
        }
        server.close();
    }
}

~~~



**客户端**

~~~java
import java.io.PrintStream;
import java.net.Socket;
import java.util.Scanner;

public class EchoClient {
    public static void main(String[] args) throws Exception {
        Socket client = new Socket("localhost", 9999);
        // 键盘输入数据
        Scanner input = new Scanner(System.in);         
        // 利用Scanner包装客户端输入数据（服务器端输出），PrintStream包装客户端输出数据；
        Scanner scan = new Scanner(client.getInputStream());
        PrintStream out = new PrintStream(client.getOutputStream());
        // 设置键盘输入分隔符
        input.useDelimiter("\n");
        // 设置回应数据分隔符
        scan.useDelimiter("\n");                        
        boolean flag = true;                       
        while (flag) {
            System.out.print("请输入要发送数据：");
            if (input.hasNext()) {                    
                String str = input.next().trim(); 
                // 发送数据到服务器端
                out.println(str);                     
                if (str.equalsIgnoreCase("byebye")) {       
                    flag = false;                     
                }
                if (scan.hasNext()) {                   
                    System.out.println(scan.next());       
                }
            }
        }
        input.close();
        scan.close();
        out.close();
        client.close();
    }
}

~~~

