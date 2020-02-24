# 类加载过程
类加载过程为JVM将类描述数据从.class文件中加载到内存，并对数据进行解析和初始化，最终形成被JVM直接使用的Java类型。包含：
- 加载：获取该类的二进制字节流，将字节流代表的静态存储结构转化为方法区的运行时数据结构，并在内存生成代表该类的 java.lang.Object 对象作为方法区该类的访问入口
- 验证：确保 Class 文件的字节流中包含的信息符号当前虚拟机的要求（文件格式验证、元数据验证、字节码验证、符号引用验证）
- 准备：为类变量分配内存并设置类变量初始值
- 解析：将常量池内的符号引用替换为直接引用
- 初始化：执行类构造器 <client>() 方法

# 类加载器

类加载过程中的加载操作由类加载去完成。类加载器分为：
- **启动类加载器/Bootstrap ClassLoader**：负责加载JAVA_HOME/lib目录中的所有类，或者加载由选项-Xbootcalsspath指定的路径下的类；
- **扩展类加载器/ExtClasLoader**：负责加载JAVA_HOME/lib/ext目录中的所有类型，或者由参数-Xbootclasspath指定路径中的所有类型；
- **应用程序类加载器/AppClassLoader**：负责加载用户类路径ClassPath下的所有类型
- **自定义加载器**：所有继承抽象类java.lang.ClassLoader的类加载器

# 双亲委派模型
![image](https://www.cnblogs.com/images/cnblogs_com/wupeixuan/1186116/o_%e5%8f%8c%e4%ba%b2%e7%b1%bb%e5%8a%a0%e8%bd%bd%e8%bf%87%e7%a8%8b.jpg)

双亲委派过程：当一个类加载器收到类加载任务时，立即将任务委派给它的父类加载器去执行，直至委派给最顶层的启动类加载器为止。如果父类加载器无法加载委派给它的类时，将类加载任务退回给它的下一级加载器去执行。

双亲委派模型最大的好处就是让Java类同其类加载器一起具备了一种带优先级的层次关系。举个例子来说明下：比如我们要加载顶层的Java类——java.lang.Object类，无论我们用哪个类加载器去加载Object类，这个加载请求最终都会委托给启动类加载器（Bootstrap ClassLoader），这样就保证了所有加载器加载的Object类都是同一个类。如果没有双亲委派模型，就会出现 Wupx::Object 和 Huyx::Object 这样两个不同的Object类。

# 双亲委派模型案例
java.lang.ClassLoader 的 loadClass() 方法
```
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

<center><img src="https://img2018.cnblogs.com/blog/1356806/201910/1356806-20191009000648748-355850292.png" /></center>
