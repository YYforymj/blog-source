---
title: springboot引入swagger
date: 2018-09-03 19:00:30
categories: springboot

---

> swagger是现在很流行的 REST APIs文档生成工具之一，本文记录了一次在springboot中引入swagger-ui的操作与代码。

<!-- more -->

## Swagger 介绍

- **官网**： https://swagger.io/
- **介绍**： swagger最主要的作用就是REST APIs文档的生成，同时还具有一定的测试功能，比较适合互联网企业项目周期短的特点。

## Springboot 引入 swagger—ui

- swagger-ui 支持 maven，所以导入就十分简单了，只需要在pom文件中加入依赖即可，其中 ${swagger.version} 为版本号；

  ```
  <!-- swagger  -->
          <dependency>
              <groupId>io.springfox</groupId>
              <artifactId>springfox-swagger2</artifactId>
              <version>${swagger.version}</version>
          </dependency>
          <dependency>
              <groupId>io.springfox</groupId>
              <artifactId>springfox-swagger-ui</artifactId>
              <version>${swagger.version}</version>
          </dependency>
          <!-- swagger  -->
  ```

- 当然仅仅引入依赖是不够的，想要运行起来需要进行配置，以下为配置代码，其中 `package` 为 swagger 需要扫描的 controller 所在的包：

  ```
  @Configuration
  @EnableSwagger2
  public class SwaggerConfig {
      @Bean
      public Docket enableSwagger(){
          return new Docket(DocumentationType.SWAGGER_2)
                  .apiInfo(new ApiInfoBuilder().title("example").version("0.0.1").build())
                  .protocols(Sets.newHashSet("http", "https"))
                  .select()
                  .apis(RequestHandlerSelectors.basePackage("package"))
                  .paths(PathSelectors.any())
                  .build()
                  .enable(true); //设置为false则swagger失效
      }
  
  }
  ```

  至此在 springboot 项目中应该就可以使用 swagger 自动生成文档了，启动 springboot 项目后，访问 `http://localhost:8089/swagger-ui.html` 就可以查看生成的文档。需要注意的是，swagger 使用的端口是 springboot 项目配置的端口（例如项目使用默认端口8080，则访问地址为 `http://localhost:8080/swagger-ui.html`）。还有一个问题就是如果项目配置文件中配置了项目路径，如下所示，则文档访问页面也需要加上改前缀（以下面配置为例，地址为 `http://localhost:8080/example/swagger-ui.html`）:

  ```
  server:
    servlet:
      context-path: /example
  ```

## 与GSON兼容

* 由于本人项目中使用了 gson 替代了 springboot 默认的 jackson，导致 swagger 无法解析 json，需要进行代码上的处理，记录如下：

* GsonSwaggerConfig.java

  ```
  @Configuration
  public class GsonSwaggerConfig {
      //设置swagger支持gson
      @Bean
      public IGsonHttpMessageConverter IGsonHttpMessageConverter() {
          return new IGsonHttpMessageConverter();
      }
  }
  ```

* IGsonHttpMessageConverter.java

  ```
  public class IGsonHttpMessageConverter extends GsonHttpMessageConverter {
      public IGsonHttpMessageConverter() {
          //自定义Gson适配器
          super.setGson(new GsonBuilder()
                  .registerTypeAdapter(Json.class, new SpringfoxJsonToGsonAdapter())
                  .serializeNulls()//空值也参与序列化
                  .create());
      }
  }
  ```

- SpringfoxJsonToGsonAdapter.java

  ```
  public class SpringfoxJsonToGsonAdapter implements JsonSerializer<Json> {
      @Override
      public JsonElement serialize(Json json, Type type, JsonSerializationContext jsonSerializationContext) {
          return new JsonParser().parse(json.value());
      }
  }
  ```

- 在项目中加入以上代码即可实现 swagger-ui 对 gson 的兼容（实际上如注释所示，就是添加了Gson适配器）。