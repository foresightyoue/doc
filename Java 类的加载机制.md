加载.class文件的方式
- – 从本地系统中直接加载
- – 通过网络下载.class文件
- – 从zip，jar等归档文件中加载.class文件
- – 从专有数据库中提取.class文件
- – 将Java源文件动态编译为.class文件

加载

- 加载时类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：
  - 通过一个类的全限定名来获取其定义的二进制字节流。
  - 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
  - 在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。

连接

- 验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

 - 验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

- 准备阶段是正式为类变量分配内存并设置类变量初始值的阶段
- 解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。
 - 符号引用就是一组符号来描述目标，可以是任何字面量。
 - 直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

初始化
 - 为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。
  - 声明类变量是指定初始值
  - 使用静态代码块为类变量指定初始值

 - 类初始化时机：只有当对类的主动使用的时候才会导致类的初始化，类的主动使用包括以下六种：
  - 创建类的实例，也就是new的方式
  - 访问某个类或接口的静态变量，或者对该静态变量赋值
  - 调用类的静态方法
  - 反射（如Class.forName(“com.shengsiyuan.Test”)）
  - 初始化某个类的子类，则其父类也会被初始化
  - Java虚拟机启动时被标明为启动类的类（Java Test），直接使用java.exe命令来运行某个主类

结束生命周期
- 在如下几种情况下，Java虚拟机将结束生命周期
  - 执行了System.exit()方法
  - 程序正常执行结束
  - 程序在执行过程中遇到了异常或错误而异常终止
  - 由于操作系统出现错误而导致Java虚拟机进程终止

JVM类加载机制
- 全盘负责，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
- 父类委托，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
- 缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效

类的加载
- 命令行启动应用时候由JVM初始化加载
- 通过Class.forName()方法动态加载      
- 通过ClassLoader.loadClass()方法动态加载     

```java
package com.neo.classloader;
public class loaderTest {
        public static void main(String[] args) throws ClassNotFoundException {
                ClassLoader loader = HelloWorld.class.getClassLoader();
                System.out.println(loader);
                //使用ClassLoader.loadClass()来加载类，不会执行初始化块
                loader.loadClass("Test2");
                //使用Class.forName()来加载类，默认会执行初始化块
//                Class.forName("Test2");
                //使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块
//                Class.forName("Test2", false, loader);
        }
}
```

```java
public class Test2 {
        static {
                System.out.println("静态初始化块执行了！");
        }
}
```  
- Class.forName()：将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块；
- ClassLoader.loadClass()：只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。
  - 注：Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 。


双亲委派机制:
- 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。
- 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。
- 如果BootStrapClassLoader加载失败（例如在$JAVA_HOME/jre/lib里未查找到该class），会使用ExtClassLoader来尝试加载。
- 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

```java
public Class<?> loadClass(String name)throws ClassNotFoundException {
            return loadClass(name, false);
    }

    protected synchronized Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException {
            // 首先判断该类型是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
                //如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                try {
                    if (parent != null) {
                         //如果存在父类加载器，就委派给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {
                    //如果不存在父类加载器，就检查是否是由启动类加载器加载的类，通过调用本地方法native Class findBootstrapClass(String name)
                        c = findBootstrapClass0(name);
                    }
                } catch (ClassNotFoundException e) {
                 // 如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
```

双亲委派模型意义
- 系统类防止内存中出现多份同样的字节码
- 保证Java程序安全稳定运行

自定义类加载器
- 比如应用是通过网络来传输 Java 类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，这样则需要自定义类加载器来实现。
- 自定义类加载器一般都是继承自 ClassLoader 类，从上面对 loadClass 方法来分析来看，我们只需要重写 findClass 方法即可。

``` Java
package com.neo.classloader;

import java.io.*;

public class MyClassLoader extends ClassLoader {

    private String root;

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] loadClassData(String className) {
        String fileName = root + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args)  {

        MyClassLoader classLoader = new MyClassLoader();
        classLoader.setRoot("E:\\temp");

        Class<?> testClass = null;
        try {
            testClass = classLoader.loadClass("com.neo.classloader.Test2");
            Object object = testClass.newInstance();
            System.out.println(object.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}

```  

      
