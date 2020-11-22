---
title: springBoot 中的那些“开关”
categories:
- 后端
tags: springBoot
date:
---
> 在之前我们就Swagger使用篇，可以了解到根据不同环境更改为不同的配置，让不同的配置逻辑生效的处理办法。其实吧，有没有觉得这东西就像是一个开关，那么SpringBoot 我们可以怎么获取当前环境（获取其他配置相同）从而进行判断做一个开关呢？今天我们就来总结下(以下列举的是常用的几种方式)：

### 方式一：@Value 注解

代码如下（基本容器启动什么地方都可以用）：

```JAVA
@RestController
@RequestMapping("/test")
public class TestController {

    @Value("${spring.profiles.active}")
    String active;

    @GetMapping("hello")
    public String sayHello(){
        return "hello, active env is: ["+active+"]";
    }
}
```

### 方式二：Spring 配置上下文

代码如下（启动时获取）：

```JAVA
@SpringBootApplication
public class SpringStudyApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(SpringStudyApplication.class, args);
        String active = context.getEnvironment().getProperty("spring.profiles.active");
        System.out.println(">>>>>>>>>>>>>>>>>>>>> active env is: ["+active+"]");
        // FileHelper.xmlName = active+".xml";
        // FileHelper.ReaderXml();
    }
}
```

> 题外话：现在用了springBoot 后各种简化了的配置，properties，yaml；那你还写xml 吗？

个人觉得如果配置多了，yaml 和properties 的这种配置反而有点不直观（一行就可以写清楚的，一直往下点很多个...），可能还是得用一些xml 配置；当然不用也不是不可以，比如你们用了Apollo 自动配置，那实时修改不用发包就能生效，肯定所有的配置都整成yaml 格式的要好啊！

但是xml配置我们也得知道，看如下代码读取xml 配置文件并组装成key,value使用；对应的环境配置也可以从此处读取

引入dom4j依赖：

```xml
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
```

在resources下新增文件夹configuration,新增配置文件dev.xml

```xml
<?xml version = "1.0" encoding="UTF-8"?>
<system>
    <config name="env-context" explain="环境相关">
        <item name="active" value="dev" explain="dev 环境"/>
    </config>
    <config name="def-config" explain="自定义配置">
        <item name="host" value="xxx.xxx.xxx.xxx" explain="主机ip"/>
        <item name="port" value="22" explain="端口号"/>
        <item name="account" value="root" explain="账号"/>
        <item name="password" value="123456" explain="密码"/>
    </config>
</system>
```

添加工具读取配置文件FileHelper类：

```java
public class FileHelper {
    public static String xmlName = "";
    public static final Map<String,String > map = new HashMap<String, String>();

    public static void ReaderXml(){
        // 创建SAXReader的对象reader
        SAXReader reader = new SAXReader();
        ResourceLoader resourceLoader = new DefaultResourceLoader();
        String name = "";
        String key = "";
        try {
            // 通过reader对象的read方法加载configuration.xml文件,获取docuemnt对象。
            File file = new File(xmlName);
            Resource resource = new FileSystemResource(file);

            if (!resource.exists()) {
                //jar同级目录加载目录下还是找不到，那就直接用classpath下的
                resource = resourceLoader.getResource("classpath:configuration/"+xmlName);
            }
            Document document = reader.read(resource.getInputStream());
            // 通过document对象获取根节点bookstore
            Element bookStore = document.getRootElement();
            // 通过element对象的elementIterator方法获取迭代器
            Iterator it = bookStore.elementIterator();
            // 遍历迭代器
            String keyName = "name";
            String keyValue = "value";
            while (it.hasNext()) {
                Element str = (Element) it.next();
                // 获取属性名以及属性值
                List<Attribute> strAttrs = str.attributes();
                for (Attribute attr : strAttrs) {
                    if(attr.getName().equals(keyName)){
                        name = attr.getValue();
                    }
                }
                //解析子节点的信息
                Iterator itt = str.elementIterator();
                while (itt.hasNext()) {
                    Element strChild = (Element) itt.next();
                    List<Attribute> bookChildList = strChild.attributes();
                    for (Attribute attr : bookChildList) {
                        if(attr.getName().equals(keyName)){
                            key = name +"." + attr.getValue();
                        }
                        if(attr.getName().equals(keyValue)){
                            map.put(key,attr.getValue());
                        }
                    }
                }
            }
            System.out.println(">>>>>>>>>>>>>>>>读取配置文件内容如下："+map);
        } catch (DocumentException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 重新加载xml配置文件
     * @return
     */
    public static Map<String,String> RestReaderXml(){
        map.clear();
        ReaderXml();
        return map;
    }
}
```

读取配置文件内容如下：`{def-config.account=root, def-config.host=xxx.xxx.xxx.xxx, def-config.password=123456, env-context.active=dev, def-config.port=22}`

那么,我们就可以根据对应的key 获取对应的配置了。

### 方式三：自定义SpringContextUtil工具类

代码如下：

```JAVA
@Component
public class SpringContextUtil implements ApplicationContextAware {

    private static ApplicationContext context = null;

    /* (non Javadoc)
     * @Title: setApplicationContext
     * @Description: spring获取bean工具类
     * @param applicationContext
     * @throws BeansException
     * @see org.springframework.context.ApplicationContextAware#setApplicationContext(org.springframework.context.ApplicationContext)
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext)
            throws BeansException {
        this.context = applicationContext;
    }

    // 传入线程中
    public static <T> T getBean(String beanName) {
        return (T) context.getBean(beanName);
    }

    // 国际化使用
    public static String getMessage(String key) {
        return context.getMessage(key, null, Locale.getDefault());
    }

    /// 获取当前环境
    public static String getActiveProfile() {
        return context.getEnvironment().getActiveProfiles()[0];
    }
}
```

### 方式四：Environment 类

代码如下（在bean中使用，例如之前Swagger 的配置，其实只要初始化配置能够读到的地方都可以这样用)：

```JAVA
Profiles pro = Profiles.of("dev","test");
// 判断是否是对应的环境
boolean enable = env.acceptsProfiles(pro);
```

详见之前的博客：<a src="http://imecho.life/archives/swagger" target="_blank">Swagger食用方法详解</a>

