java.NIO包里包括三个基本的组件

l buffer：因为NIO是基于缓冲的，所以buffer是最底层的必要类，这也是IO和NIO的根本不同，虽然stream等有buffer开头的扩展类，但只是流的包装类，还是从流读到缓冲区，而NIO却是直接读到buffer中进行操作。

因为读取的都是字节，所以在操作文字时，要用charset类进行编解码操作。

l channel：类似于IO的stream，但是不同的是除了FileChannel，其他的channel都能以非阻塞状态运行。FileChannel执行的是文件的操作，可以直接DMA操作内存而不依赖于CPU。其他比如socketchannel就可以在数据准备好时才进行调用。

l selector：用于分发请求到不同的channel，这样才能确保channel不处于阻塞状态就可以收发消息。


![image](https://gss2.bdstatic.com/9fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=f800152ede3f8794c7f2407cb3726591/3c6d55fbb2fb431607678cec2aa4462309f7d328.jpg)


![image](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/crop%3D5%2C0%2C546%2C361%3Bc0%3Dbaike80%2C5%2C5%2C80%2C26/sign=bca1918ea98b87d6440df15f3a3d0408/8b13632762d0f7039e32b66300fa513d2797c588.jpg)
