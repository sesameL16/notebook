五个核心类：File 、InputStream 、OutputStream 、Reader 、Writer；

​	InputStream类的功能不足己经被Scanner解决了。

​	Reader类的功能不足被BufferedReader解决了。

​	OutputStreain类的功能不足被PrintStream解决了。

​	Writer类的功能不足被PrintWriter解决了。

一个核心接口：Serializable(标识接口)



**File类**

File类是唯一一个与文件本身操作有关的类，但是不涉及到文件的内容

要想使用File类，那么首先需要通过他的构造方法定义一个要操作的文件路径

设置完整路径：public File（String pathname）；大部分

设置父路径与子文件路径：public File（File parent，String child）；Android用的比较多

 

* 方法：（文件操作）

    创建文件：public bolean createNewFile（）throws IOException；

    > 目录不能访问

    > 文件重名，或者文件名称错误

    删除文件：public boolean delete（）；

    判断文件是否存在：public boolean exists（）；

    找到父路径：public File getParentFile（）；

    创建目录：public boolean mkdirs（）；

~~~java
public static void main(String[] args)throws Exception {
    File file = new File("E:"+File.separator+"demo"+File.separator+"test.txt");
    if(!file.getParentFile().exists()) {
        //创建父路径
        file.getParentFile().mkdirs();  
    }
    if(file.exists()) { //文件存在
        file.delete();
        System.out.println("删除文件");
    }else {
        file.createNewFile();
        System.out.println("创建文件");
    }
}

~~~

* 方法：（获得文件信息）

    取得文件大小：public long length（），按照字节返回；

    判断是否是文件：public boolean isFile（）；

    判断是否是目录：public boolean isDirectory（）；

    最近一次修改日期：public long lastModified（）；

~~~java
public static void main(String[] args)throws Exception {
    File file = new File("E:"+File.separator+"demo"+File.separator+"test.jpeg");
    if(file.exists()) { 
        System.out.println("是否是文件："+file.isFile());
        System.out.println("是否是目录："+file.isDirectory());
        System.out.println("文件大小："+ 
            new BigDecimal((double)file.length()/1024/1024).              
            divide(new BigDecimal(1), 2, BigDecimal.ROUND_HALF_UP) +"M");
        System.out.println("上次修改时间"+new SimpleDateFormat("yyyy-mm-dd").
            format(file.lastModified()));
    }
}

~~~

* 方法：（列出目录)

    列出目录下的信息：public String[] list（）；

    列出所有信息以File类对象包装：public File[] listFiles（）；

~~~java
public static void main(String[] args)throws Exception { 
    File file = new File("E:");
    print(file);
}
public static void print(File file) {
    if(file.isDirectory()) { //如果是文件夹
        File[] listFiles = file.listFiles(); //列出子目录
        if(listFiles != null) {
            for(int i = 0 ; i < listFiles.length ; i++) {
                print(listFiles[i]); //继续列出
            }
        }
    }
    System.out.println(file);
}
~~~



**字节输出流：OutputStream**

* 定义

    OutputStream类是一个专门进行字节输出的一个类，定义如下

    public abstract class OutputStream
            extends Object
            implements Closeable, Flushable

* 方法

    public void close（）；关闭资源

    public void flash（）；缓冲的输出字节写出

    public abstract void write（int b）throws IOException；输出单个字节

    public void write（byte[] b）throws IOException；输出全部字节数组

    public void write（byte[] b ，int off，int len）throws IOException；输出部分字节数组

* 子类

    OutputStream类是一个抽象类，如果想要为抽象类进行对象的实例化操作，那么一定要使用抽象类的子类，文件操作可以使用FileOutputStream子类。构造方法如下：

    > public FileOutputStream（File file）创建或覆盖已有文件；

    > public FileOutputStream（File file ，boolean append）文件内容追加；

~~~java
public static void main(String[] args) throws Exception {
    File file = new File("d:" + File.separator + "demo" + 
                         File.separator + "mldn.txt");
    // 此时由于目录不存在，所以文件不能输出，应该首先创建目录
    if (!file.getParentFile().exists()) { 
        file.getParentFile().mkdirs(); 
    }
    // 2．应该使用OutputStream和其子类进行对象的实例化，此时目录存在，文件还不存在
    OutputStream output = new FileOutputStream(file);
    // 字节输出流需要使用byte类型，需要将String类对象变为字节数组
    String str = "更多课程资源请访问：www.yootk.com";
    // 将字符串变为字节数组
    byte data[] = str.getBytes();
    output.write(data);
    output.close();
}

~~~



**字节输入流：InputStream**

* 定义

    InputStream类是一个专门进行字节输入的一个类，定义如下

    public abstract class InputStream

    ​        extends Object

    ​        implements Closeable

* 方法

    > public void close（）；关闭资源

    > public void flash（）；缓冲的输出字节写出

    > public abstract int read（）throws IOException；读取单个字节返回值：返回读取的字节内容，如果现在已经没有内容了返回-1 

    > public int read（byte[] b）throws IOException；将读取的数据保存在字节数组里返回值：返回读取的数据长度；如果已经读取到结尾返回-1

    > public int read（byte[] b ，int off，int len）throws IOException；将读取的数据保存在部分字节数组里返回值：返回读取的部分数据的长度，如果已经读取到结尾，返回-1

* 子类

    InputStream类是一个抽象类，如果想要为抽象类进行对象的实例化操作，那么一定要使用抽象类的子类，文件操作可以使用FileInputStream子类。构造方法如下：

    > public FileInputStream（File file）



~~~java
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;

public class TestDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("d:" + File.separator + "demo" + 
                             File.separator + "mldn.txt");
        if (file.exists()) {
            //使用InputStream进行读取
            InputStream input = new FileInputStream(file) ;
            byte data [] = new byte [1024] ; 
            //进行数据读取，将内容保存到字节数组中
            int len = input.read(data) ;                    
            input.close();
            // 将读取出来的字节数组数据变为字符串进行输出
            System.out.println("【" + new String(data,0,len) + "】");
        }
    }
}
~~~



~~~java
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
public class TestDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("d:" + File.separator + "demo" + 
                             File.separator + "mldn.txt");                           
        if (file.exists()) {                                
            //使用InputStream进行读取
            InputStream input = new FileInputStream(file) ;
            byte data [] = new byte [1024] ; 
             // 表示字节数组的操作脚标 
            int foot = 0 ;    
             // 表示接收每次读取的字节数据
            int temp = 0 ;                               
            // 第一部分：(temp = input.read())，表示将read()方法读取的字节内容给temp变量
            // 第二部分：(temp = input.read()) != -1，判断读取的temp内容是否是-1
            while((temp = input.read()) != -1) {
                data[foot ++] = (byte) temp ;
            }
            input.close(); 
            System.out.println("【" + new String(data,0,foot) + "】");
        }
    }
}

~~~



**字符输出流：Writer**

* 定义

    Writer类是一个专门进行字符输出的一个类，定义如下

    public abstract class Writer

    ​        extends Object

    ​        implements  Appendable, Closeable, Flushable

* 方法

    public void write（char[]  c）throws IOException；输出全部字符数组

    public void write（String str）throws IOException；输出字符串

* 子类

    Writer类是一个抽象类，如果想要为抽象类进行对象的实例化操作，那么一定要使用抽象类的子类，文件操作可以使用FileWriter子类。构造方法如下：

    > public FileWriter（File file）创建或覆盖已有文件；

    > public FileWriter（File file ，boolean append）文件内容追加；

~~~java
import java.io.File;
import java.io.FileWriter;
import java.io.Writer;
public class TestDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("d:" + File.separator + "demo" + 
                             File.separator + "mldn.txt");                    
        if (!file.getParentFile().exists()) {               
            file.getParentFile().mkdirs();              
        }
        Writer out = new FileWriter(file);
        String str = "更多课程请访问：www.yootk.com";   
        out.write(str);                         
        out.close();   
    }
}

~~~

**字符输入流：Reader**

* 定义

    Reader类是一个专门进行字符输出的一个类，定义如下

    public abstract class Reader

    ​        extends Object

    ​        implements  Readable, Closeable

* 方法

    > public abstract int read（）throws IOException；读取单个字节，返回值：返回读取的字节内容，如果现在已经没有内容了返回-1 

    > public int read（char【】 c）throws IOException；将读取的数据保存在字节数组里，返回值：返回读取的数据长度；如果已经读取到结尾返回-1

    > public int read（char【】 c ，int off，int len）throws IOException；将读取的数据保存在部分字节数组里，返回值：返回读取的部分数据的长度，如果已经读取到结尾，返回-1

* 子类

    Reader类是一个抽象类，如果想要为抽象类进行对象的实例化操作，那么一定要使用抽象类的子类，文件操作可以使用FileReader子类。构造方法如下：

    > public FileReader（File file）

~~~java
import java.io.File;
import java.io.FileReader;
import java.io.Reader;

public class TestDemo {
    public static void main(String[] args) throws Exception { 
        File file = new File("d:" + File.separator + "demo" + 
                             File.separator + "mldn.txt");                        
        if (file.exists()) {
            Reader in = new FileReader(file) ;          
            char data [] = new char [1024] ;           
            int len = in.read(data) ;                
            in.close();                            
            System.out.println(new String(data, 0, len));
        }
    }
}

~~~



**实现文件复制操作**

~~~java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
public class CopyDemo {
    public static void main(String[] args) throws Exception {
        long start = System.currentTimeMillis() ;  
        if (args.length != 2) {  
            System.out.println("命令执行错误！");
            System.exit(1); 
        }
        // 如果输入参数正确，应该进行源文件有效性的验证
        File inFile = new File(args[0]) ; 
        if (!inFile.exists()) {   
            System.out.println("源文件不存在，请确认执行路径。");
            System.exit(1);
        }
        // 如果此时源文件正确，就需要定义输出文件，同时要考虑到输出文件有目录
        File outFile = new File(args[1]) ;
        if (!outFile.getParentFile().exists()) { 
            outFile.getParentFile().mkdirs() ;
        }
        // 实现文件内容的复制，分别定义输出流与输入流对象
        InputStream input = new FileInputStream(inFile) ;
        OutputStream output = new FileOutputStream(outFile) ;
        int temp = 0 ;                            
        byte data [] = new byte [1024] ;             
        // 将每次读取进来的数据保存在字节数组里面，并且返回读取的个数
        while((temp = input.read(data)) != -1) {     
            output.write(data, 0, temp);          
        }
        input.close();                       
        output.close();                    
        long end = System.currentTimeMillis() ; 
        System.out.println("复制所花费的时间：" + (end - start));
    }
}

~~~



**内存流**

​	内存字节流：ByteArrayInputStream、ByteArrayOutputStream

​	内存字符流：CharArrayReader、CharArrayWriter

​	构造方法：  ByteArrayInputStream（byte[] buf）表示将要操作的数据设置到输入流

​	ByteArrayOutputStream（）从内存输出数据

~~~java
//实现文件的合并读取
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;

public class TestDemo {
    public static void main(String[] args) throws Exception { 
        File fileA = new File("D:" + File.separator + "infoa.txt");           
        File fileB = new File("D:" + File.separator + "infob.txt");            
        InputStream inputA = new FileInputStream(fileA);               
        InputStream inputB = new FileInputStream(fileB);             
        ByteArrayOutputStream output = new ByteArrayOutputStream();  
        int temp = 0;                                       
        while ((temp = inputA.read()) != -1) {                  
            output.write(temp);                                  
        }
        while ((temp = inputB.read()) != -1) {                  
            output.write(temp);                      
        }
        // 现在所有的内容都保存在了内存输出流里面，所有的内容变为字节数组取出
        byte data[] = output.toByteArray();                      
        output.close();                                      
        inputA.close();                                     
        inputB.close();                
        // 字节转换为字符串输出
        System.out.println(new String(data)); 
    }

~~~



**打印流**

​	PrintStream类为OutputStream类的子类，构造方法public PrintStream（OutputStream out）

~~~java
import java.io.File;
import java.io.FileOutputStream;
import java.io.PrintStream;
public class TestDemo {
    public static void main(String[] args) throws Exception { // 此处直接抛出
        // 实例化PrintStream类对象，本次利用FileOutputStream类实例化PrintStream类对象
        PrintStream pu = new PrintStream(
            new FileOutputStream(new File("d:" + File.separator + "yootk.txt")));
        pu.print("优拓教育：");
        pu.println("www.yootk.com");
        pu.println(1 + 1);
        pu.println(1.1 + 1.1);
        pu.close();
    }
}

~~~



**缓冲流**

​	字符缓冲区流：BufferedReader、bufferedWriter；

​	字节缓冲区流：BufferedInputStream、BufferedOutputStream

​	BufferedReader类中有个重要方法

​	Public String readLine（）读取一行数据，以分隔符（换行）

​	构造方法  public BufferedReader（Reader in）；

~~~java
import java.io.BufferedReader;
import java.io.InputStreamReader;
public class TestDemo {
    public static void main(String[] args) throws Exception { 
        BufferedReader buf = new BufferedReader(new InputStreamReader(System.in));
        boolean flag = true;                         
        while (flag) {   
            System.out.print("请输入年龄：");
            String str = buf.readLine(); 
            // 输入数据由数字组成
            if (str.matches("\\d{1,3}")) {                
                System.out.println("年龄是：" + Integer.parseInt(str));
                flag = false ;  
            } else {
                System.out.println("年龄输入错误，应该由数字所组成。");
            }
        }
    }
}
~~~

~~~java
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
public class TestDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("D:" + File.separator + "yootk.txt");
        // 使用文件输入流实例化BufferedReader类对象
        BufferedReader buf = new BufferedReader(new FileReader(file));
        String str = null;                          
        while ((str = buf.readLine()) != null) {
            System.out.println(str);   
        }
        buf.close();
    }
}
~~~



**扫描流（java.util.Scanner）**

​	构造方法： public Scanner(InputStream source);

​	接收一个InputStream类对象，表示的是由外部设置输入的位置

​	读取数据：public 数据类型 nextXxx（）

​	设置分隔符方法：public Scanner useDelimiter（String pattern）

~~~java
import java.util.Scanner;
public class TestDemo {
    public static void main(String[] args) throws Exception {
        Scanner scan = new Scanner(System.in); 
        System.out.print("请输入内容：");  
        if (scan.hasNext()) { 
            System.out.println("输入内容：" + scan.next());
        }
        scan.close();
    }
}
~~~

~~~java
import java.io.File;
import java.io.FileInputStream;
import java.util.Scanner;
public class TestDemo {
    public static void main(String[] args) throws Exception {
        Scanner scan = new Scanner
            (new FileInputStream(new File("D:" + File.separator + "yootk.txt"))); 
        // 设置读取的分隔符
        scan.useDelimiter("\n");                     
        while (scan.hasNext()) {                       
            System.out.println(scan.next());    
        }
        scan.close();
    }
}
~~~

