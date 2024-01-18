# properties详解

## 两种注解方法

### @Value注解

实体类：

```java
@Data
@Component
public class ApplicationProperty {
    @Value("${application.name}")
    private String name;
    @Value("${application.version}")
    private String version;
}
```

配置：

```yaml
application:
  name: dev环境 @artifactId@
  version: dev环境 @version@
```

### @ConfigurationProperties注解

实体类：

```
@Data
@ConfigurationProperties(prefix = "developer")
@Component
public class DeveloperProperty {
    private String name;
    private String website;
    private String qq;
    private String phoneNumber;
}
```

配置：

```
developer:
  name: dev环境 xkcoding
  website: dev环境 http://xkcoding.com
  qq: dev环境 237497819
  phone-number: dev环境 17326075631
```

## additional-spring-configuration-metadata.json文件的作用

作用：在写配置文件时，IDE可以给出提示。

```json
{
	"properties": [
		{
			"name": "application.name",
			"description": "Default value is artifactId in pom.xml.",
			"type": "java.lang.String"
		},
		{
			"name": "application.version",
			"description": "Default value is version in pom.xml.",
			"type": "java.lang.String"
		},
		{
			"name": "developer.name",
			"description": "The Developer Name.",
			"type": "java.lang.String"
		},
		{
			"name": "developer.website",
			"description": "The Developer Website.",
			"type": "java.lang.String"
		},
		{
			"name": "developer.qq",
			"description": "The Developer QQ Number.",
			"type": "java.lang.String"
		},
		{
			"name": "developer.phone-number",
			"description": "The Developer Phone Number.",
			"type": "java.lang.String"
		}
	]
}
```

