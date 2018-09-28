# spring-cloud-config-server
##创建config-service
* application.yml中添加文件地址
```yaml
spring:
    cloud:
        config:
            server:
                git:
                    uri: https://github.com/yjpfj1203/static-resource
                    search-paths: config-files
                    username: yjpfj1203@163.com
                    password: Liuhan1203
```
地址实际如下图：
![spring-cloud-config-url](https://raw.githubusercontent.com/yjpfj1203/static-resource/master/config-files/images/spring-cloud-config-url.png)
* bootstrap.yml中添加如下代码
```yaml
server:
    port: 8100
#    ssl.key-store: classpath:keystore.p12
#    ssl.key-store-password: Liuhan1203
#    ssl.keyStoreType: PKCS12
#    ssl.keyAlias: tomcat
spring:
    application:
        name: config-server
```
* 启动类添加注释
```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConfigServerApplication.class, args);
	}
}
```

启动执行，访问地址：http://localhost:8100/user-service/dev<br>
这里会加载四个文件，具体如图
![spring-cloud-config-url](https://raw.githubusercontent.com/yjpfj1203/static-resource/master/config-files/images/user-service-dev-config.png)

##其他服务引用config-service中的配置文件
* 添加pom引用
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```
* bootstrap.yml中添加如下代码
```yaml
spring:
    application:
        name: user-service
    cloud:
        config:
            uri: http://localhost:8100
```
如果启动时出现如下代码，则为正常启动<br>
```test
2018-09-27 17:00:37.292  INFO 27164 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8100<br>
2018-09-27 17:00:40.078  INFO 27164 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Located environment: name=user-service, profiles=[dev], label=null, version=024de370ae0fe05b6c029233a1626a6e750ad011, state=null<br>
2018-09-27 17:00:40.079  INFO 27164 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='https://github.com/yjpfj1203/static-resource/config-files/application-dev.yml'}, MapPropertySource {name='https://github.com/yjpfj1203/static-resource/config-files/user-service.yml (document #0)'}]}

```
##对配置文件敏感信息进行加密（{cipher}）
* 首先[下载jce1.8](https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)<br>
 将目录中的local_policy.jar，US_export_policy.jar拷贝到jdk的lib/security下，mac下为/Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home/jre/lib/security<br>
 测试配置成功<br>
  * 启动config-service，如果在启动日志里有如下代码则表示配置OK
  ```test
  2018-09-28 09:07:45.141  INFO 71880 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/encrypt],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.encrypt(java.lang.String,org.springframework.http.MediaType)
  2018-09-28 09:07:45.141  INFO 71880 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/encrypt/{name}/{profiles}],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.encrypt(java.lang.String,java.lang.String,java.lang.String,org.springframework.http.MediaType)
  2018-09-28 09:07:45.142  INFO 71880 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/decrypt/{name}/{profiles}],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.decrypt(java.lang.String,java.lang.String,java.lang.String,org.springframework.http.MediaType)
  2018-09-28 09:07:45.142  INFO 71880 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/decrypt],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.decrypt(java.lang.String,org.springframework.http.MediaType)
  2018-09-28 09:07:45.142  INFO 71880 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/encrypt/status],methods=[GET]}" onto public java.util.Map<java.lang.String, java.lang.Object> org.springframework.cloud.config.server.encryption.EncryptionController.status()

  ```
  
  * 配置encrypt.key<br>
   访问地址：http://localhost:8100/encrypt/status，返回结果如下
  ```json
    {
        description: "No key was installed for encryption service",
        status: "NO_KEY"
    }
   ```
  在config-service中我添加到application.yml中是无效的，我加到bootstrap.yml中是有效的。如下<br>
  ```yaml
  encrypt.key: test123456789
  ``` 
   再次访问http://localhost:8100/encrypt/status，返回结果如下
  ```json
  {
      status: "OK"
  }
  ```
  * 加密、解密<br>
  如果要对用户名(name)，密码(password)进行密码，可通过postman进行操作，具体如下图：
  ![encrypt](https://raw.githubusercontent.com/yjpfj1203/static-resource/master/config-files/images/encrypt.png)
  解密如下图
  ![decrypt](https://raw.githubusercontent.com/yjpfj1203/static-resource/master/config-files/images/decrypt.png)
  
  



