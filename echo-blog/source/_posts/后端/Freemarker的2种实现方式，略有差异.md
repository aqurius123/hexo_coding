---
title: Freemarker
categories:
- 后端
tags: Freemarker
date:
---

> 说明：freemarker作为静态化处理技术，但是实现的方式可能根据项目环境略有不同,特别注意：freemarker技术的使用只能使用@Controller注解，绝对不能使用@RestController

- 方式一：------模板位置确定的情况下使用
     1. 创建一个 Configuration 对象，直接 new 一个对象。构造方法 
            的参数就是 freemarker的版本号。
     2. 设置模板文件所在的路径。
     3. 设置模板文件使用的字符集。一般就是 utf-8.
     4. 加载一个模板，创建一个模板对象。
     5. 创建一个模板使用的数据集，可以是 pojo 也可以是 map。一般 
            是 Map。
     6. 创建一个 Writer 对象，一般创建一 FileWriter 对象，指定生 
            成的文件名。
     7. 调用模板对象的 process 方法输出文件。
     8. 关闭流
- 方式二：------模板存在数据库中的情况下使用
    ~~~
      //生成静态化页面
    private String generatePageHtml(String template, Map model) {
        try {
            //第一步：创建一个 Configuration 对象，直接 new 一个对象。构造方法的参数就是 freemarker的版本号。
            Configuration configuration = new Configuration(Configuration.getVersion());
            //模板加载器
            StringTemplateLoader stringTemplateLoader = new StringTemplateLoader();
            //模板加载器配置模板
            stringTemplateLoader.putTemplate("template", template);
            //配置模板加载器
            configuration.setTemplateLoader(stringTemplateLoader);
            //获取模板
            Template template1 = configuration.getTemplate("template");
            //生成html页面
            FreeMarkerTemplateUtils.processTemplateIntoString(template1, model);
        } catch (TemplateException e) {
            e.printStackTrace();
        }catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }