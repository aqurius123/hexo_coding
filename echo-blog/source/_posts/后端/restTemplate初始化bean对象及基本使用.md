---
title: restTemplate
categories:
- 后端
tags: restTemplate
date:
---

> 使用场景：作为远程请求接口，SpringMVC提供 RestTemplate请求http接口，RestTemplate的底层可以使用第三方的http客户端工具实现http 的请求，常用的http客户端工具有Apache HttpClient、OkHttpClient等，原因也是因为它的性能比较出众；

## 集成restTemplate准备：
1. 引入pom依赖：
  <dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
  </dependency>  
2. 初始化restTemplate的bean对象，并交给spring容器对象管理(注意：这里是直接在启动类上初始化restTemplate对象，因为@SpringBootApplication注解内部已经实现了@Configuration；也可以自定义一个配置类，专门用来初始化bean对象---使用@Configuration和@Bean注解即可):
~~~
   @SpringBootApplication
   @EntityScan("com.xuecheng.framework.domain")
   @ComponentScan(basePackages = {"com.xuecheng.api"})
   @ComponentScan(basePackages = {"com.xuecheng.manage.cms"})
   public class ManageCmsApplication {
     public static void main(String[] args) {
        SpringApplication.run(ManageCmsApplication.class,args);
    }

    //配置restTemplate----bean对象
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate(newOkHttp3ClientHttpRequestFactory());
    }
}
~~~
## restTemplate的基本使用：
   1. 注入restTemplate对象
      @Autowired
      private RestTemplate restTemplate;
   2. 调用相关Api实现(注意：getForEntity方法的两个参数一个是请求的接口路径的url,第二个参数则是HTTP响应转换被转换成的对象类型)：
   ~~~
    @Test
    public void testRestTemplate() {
        ResponseEntity<Map> forEntity = restTemplate.getForEntity("http://localhost:31001/cms/config/getModel/5a791725dd573c3574ee333f", ++Map.class++);
        System.out.println(forEntity);
        System.out.println("===================="+ 
                   forEntity.getStatusCode());	//获取返回的状态值
        System.out.println("!!!!!!!!!!!!!!!!!!!!"+ 
                   forEntity.getBody()); //获取返回值中的map对象
    }
~~~

特别注意：restTemplate相关的api方法中的关于返回值转换对象类型这个参数是一定不能少的；
