#### 应用场景
    不想写大量重复的getter,setter,构建函数，想让项目的代码更加简洁可读。使用lombok你可以实现以下功能：自动生成getter,setter，无参构造函数，全参构造函数，链式的setter等等
#### 工具简介
    Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much mo
[github地址](https://github.com/rzwitserloot/lombok)
[官网地址](https://projectlombok.org)
#### 使用以及代码示例
##### 插件下载
由于lombok是通过注解在编译的时候生成相关的函数，所以编译器如果不安装对应的lombok插件，则编辑器页面会提示找不到方法的错误（备注：编译以及编译后运行jar不会有错，这个错误是编辑器自己检查代码时的逻辑）
1. IDEA插件
- 在线安装：Perference->Plugins 搜索lombok然后安装重启IDEA即可
![image.png](https://pic.leetcode-cn.com/a1a19f60c77aed3352c3bd9e91d49d3918199be43ca167e2a35c5315992808f4-image.png)
- 离线安装 ：[https://github.com/mplushnikov/lombok-intellij-plugin/releases]()
找到指定版本的插件下载，然后选择 Perference->Plugins->Install from disks
![image.png](https://pic.leetcode-cn.com/6383e01e909b04a5186fed8adb3e8ee15b26490012d02fa23ba1fbf06a40289f-image.png)

2. Eclipse插件: 后续补充
##### Maven
Maven地址：[https://mvnrepository.com/artifact/org.projectlombok/lombok]()

```
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.8</version>
            <scope>provided</scope>
        </dependency>
```
##### 注解以及代码示例
官方文档：[https://projectlombok.org/features/all]()
- @NonNull 非空检查
    注解在参数上,当参数为空的时候，会跑出一个空指针异常：NullPointerException("param is marked @NonNull but is null")，这个检查会至于方法的很上层，比如作用于构造函数的时候，会插入在this和super后
```
import lombok.NonNull;

public class UserInfo {
    private String userId;
    private String userName;

    public UserInfo(@NonNull String userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }
    //方便测试，直接在main方法写测试代码
    public static void main(String[] args) {
        new UserInfo(null, null);
    }
}
```
异常：`Exception in thread "main" java.lang.NullPointerException: userId is marked non-null but is null`
- @Cleanup 自动关闭资源
    注解在资源对象上，用于自动关闭资源，比如InputStream等，如果资源的关闭方法是不是默认的close()，而是其他无参方法，可以使用@CleanUp("方法名")的注解，插件会自动验证资源是否存在默认的close方法或者注解的方法，不存在，编辑器会报错
```
import lombok.Cleanup;
import lombok.NonNull;

public class UserInfo {
    private String userId;
    private String userName;

    public UserInfo(@NonNull String userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }

    public void userClose() {
        System.out.println(("用户自定义的关闭方法被调用"));
    }

    public void print() {
        System.out.println(userId + ":" + userName);
    }

    //测试代码
    public static void main(String[] args) {
        @Cleanup("userClose") UserInfo userInfo = new UserInfo("wang", "longfei");
        userInfo.print();
    }
}
```
输出：
```
wang:longfei
用户自定义的关闭方法被调用
```
- @Getter/@Setter
    可以注解到参数上，并且可以给予参数AccessLevel来确定方法的权限，如果作用在类上，则为所有的非静态变量添加方法，在@Data和作用在类的条件下，可以使用注解+AccessLevel.NONE来禁自动生成方法,显示的getter和setter方法会覆盖注解
```
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

@Setter
public class UserInfo {
    @Getter
    private String userId;
    @Getter(AccessLevel.PROTECTED)
    private String userName;

    //测试代码
    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo();
        userInfo.setUserId("wang");
        userInfo.setUserName("longfei");
    }
}
```
- @ToString 重写toString
    重写toString代码，默认为className(field=value),默认打印所有的非静态变量，可以使用@ToString.Exclude来过滤一些实例属性，也可以使用onlyExplicitlyIncluded=true + @ToString.Include来指定只包含一些属性 。这里只涉及到简单使用，更复杂的比如重写父类，请参考官方文档
```
import lombok.ToString;

@ToString
public class UserInfo {
    private String userId;
    private String userName;

    public UserInfo(String userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }

    //测试代码
    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo("wang", "longfei");
        System.out.println(userInfo);
    }
}
```
输出：`UserInfo(userId=wang, userName=longfei)`

- @EqualsAndHashCode 重写equals和HashCode
    重写Equals和HashCode方法，默认使用所有的实例变量，如果需要定制，可以使用 @EqualsAndHashCode.Exclude来去掉一些属性，这里只介绍简单的使用，更多请参考官方文档
```
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class UserInfo {
    private String userId;
    @EqualsAndHashCode.Exclude
    private String userName;

    public UserInfo(String userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }

    //测试代码
    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo("wang", "longfei");
        UserInfo userInfo2 = new UserInfo("wang", "feilong");
        System.out.println(userInfo.equals(userInfo2));
    }
}
```
输出：`true`
- @NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor 构造函数
    生成无参构造函数, 一个包含常量和non-null注解的参数的构造函数，包含所有参数的构造函数
```
import lombok.EqualsAndHashCode;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;

@EqualsAndHashCode
@RequiredArgsConstructor
public class UserInfo {
    @NonNull private String userId;
    @EqualsAndHashCode.Exclude
    @NonNull private String userName;
    private int age;

    //测试代码
    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo("wang", "longfei");
        UserInfo userInfo2 = new UserInfo("wang", "feilong");
        System.out.println(userInfo.equals(userInfo2));
    }
}
```
- @Data 通用注解
     包含了以下注解：@ToString, @EqualsAndHashCode, @Getter / @Setter and @RequiredArgsConstructor，可以使用这种建议方式获得一个范型实例生成方法：@Data(staticConstructor="of")，详情见如下官方代码和参考官方文档
```
import lombok.AccessLevel;
import lombok.Setter;
import lombok.Data;
import lombok.ToString;

@Data public class DataExample {
  private final String name;
  @Setter(AccessLevel.PACKAGE) private int age;
  private double score;
  private String[] tags;
  
  @ToString(includeFieldNames=true)
  @Data(staticConstructor="of")
  public static class Exercise<T> {
    private final String name;
    private final T value;
  }
}
```
- @Value 把属性变成final,只有getter方法
    官方解释：final @ToString @EqualsAndHashCode @AllArgsConstructor @FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE) @Getter 添加一个全参构造函数，把属性变成final和私有，并且只有get方法，详情见官方文档和代码：
```
import lombok.AccessLevel;
import lombok.experimental.NonFinal;
import lombok.experimental.Value;
import lombok.experimental.Wither;
import lombok.ToString;

@Value public class ValueExample {
  String name;
  @Wither(AccessLevel.PACKAGE) @NonFinal int age;
  double score;
  protected String[] tags;
  
  @ToString(includeFieldNames=true)
  @Value(staticConstructor="of")
  public static class Exercise<T> {
    String name;
    T value;
  }
}
```
- @Builder 以链式builder的方式替换构造函数来生成对象
    默认给所有变量赋值0/null,然后提供ClassName.builder().property(value)的形式来生成一个对象，如果不赋值，则默认值都是0或者null，如果要给默认值，需要在属性前面加@Builder.Default。 @Singular 注解注解一个集合时，会默认生成一个集合，如果build传入的是集合的范式，则会自动添加到集合，官方代码：
```

import lombok.Builder;
import lombok.Singular;
import java.util.Set;

@Builder
public class BuilderExample {
  @Builder.Default private long created = System.currentTimeMillis();
  private String name;
  private int age;
  @Singular private Set<String> occupations;
}
```
- @SneakyThrows 帮你捕捉uncheck exception
    应用场景：你的代码没有必须处理的异常，然是代码的运行逻辑可能会出现异常，这种异常你不想忽略掉，希望你代码的调用者必须处理这种异常，你就可以在你的代码前注解@SneakyThrows，让lombok帮你提醒调用者,compile的时候会提醒你，比如，下面的代码就无法运行，因为未处理异常
```
import lombok.Cleanup;
import lombok.SneakyThrows;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.InputStream;

public class UserInfo {
    public static void main(String[] args) {
        throwException();
    }
    @SneakyThrows(value = FileNotFoundException.class)
    public static void throwException(){
        File file = new File("filePath");
        @Cleanup InputStream inputStream = new FileInputStream(file);
    }
}
```
参考文档：[https://blog.csdn.net/weixin_41540822/article/details/86606587]()

- @Synchronized 同步关键字实现
    可以作用于实例方法或静态方法上，如果注解里不加关键字，则使用相同的默认对象$LOCK，如果加了关键字，则使用关键字命名的对象，官网实例代码：
```
import lombok.Synchronized;

public class SynchronizedExample {
  private final Object readLock = new Object();
  
  @Synchronized
  public static void hello() {
    System.out.println("world");
  }
  
  @Synchronized
  public int answerToLife() {
    return 42;
  }
  
  @Synchronized("readLock")
  public void foo() {
    System.out.println("bar");
  }
}
```
这段代码等于：
```

 public class SynchronizedExample {
  private static final Object $LOCK = new Object[0];
  private final Object $lock = new Object[0];
  private final Object readLock = new Object();
  
  public static void hello() {
    synchronized($LOCK) {
      System.out.println("world");
    }
  }
  
  public int answerToLife() {
    synchronized($lock) {
      return 42;
    }
  }
  
  public void foo() {
    synchronized(readLock) {
      System.out.println("bar");
    }
  }
}
```
- @With 替换 @Wither
    提供给final字段赋值的方法，官方示例代码：
```

import lombok.AccessLevel;
import lombok.NonNull;
import lombok.With;

public class WithExample {
  @With(AccessLevel.PROTECTED) @NonNull private final String name;
  @With private final int age;
  
  public WithExample(String name, int age) {
    if (name == null) throw new NullPointerException();
    this.name = name;
    this.age = age;
  }
}
```
等效：
```

import lombok.NonNull;

public class WithExample {
  private @NonNull final String name;
  private final int age;

  public WithExample(String name, int age) {
    if (name == null) throw new NullPointerException();
    this.name = name;
    this.age = age;
  }

  protected WithExample withName(@NonNull String name) {
    if (name == null) throw new java.lang.NullPointerException("name");
    return this.name == name ? this : new WithExample(name, age);
  }

  public WithExample withAge(int age) {
    return this.age == age ? this : new WithExample(name, age);
  }
}
```
- @Getter(lazy=true) 获取值的时候才去初始化实际的final
    修饰一个变量，则创建一个private final的变量，并且在使用的时候才去初始化，适用于某些变量加载时间较长，并且使用的时候才需要加载，官方代码：
```
import lombok.Getter;

public class GetterLazyExample {
  @Getter(lazy=true) private final double[] cached = expensive();
  
  private double[] expensive() {
    double[] result = new double[1000000];
    for (int i = 0; i < result.length; i++) {
      result[i] = Math.asin(i);
    }
    return result;
  }
}
```
- @Log 让lombok接管日志
```
@CommonsLog
Creates private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);
@Flogger
Creates private static final com.google.common.flogger.FluentLogger log = com.google.common.flogger.FluentLogger.forEnclosingClass();
@JBossLog
Creates private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);
@Log
Creates private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
@Log4j
Creates private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);
@Log4j2
Creates private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);
@Slf4j
Creates private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
@XSlf4j
Creates private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
@CustomLog
Creates private static final com.foo.your.Logger log = com.foo.your.LoggerFactory.createYourLogger(LogExample.class);
```
 能够自动生成一个log对象给你，官方代码示例：
```

 import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;

@Log
public class LogExample {
  
  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

@Slf4j
public class LogExampleOther {
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

@CommonsLog(topic="CounterLog")
public class LogExampleCategory {

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}
```

- val
- var
- experimental
