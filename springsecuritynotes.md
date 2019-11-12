不基于spring boot也可以使用spring security
,使用如下依赖:
```xml
<dependencyManagement>
    <dependencies>
        <!-- ... other dependency elements ... -->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-bom</artifactId>
            <version>5.2.1.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
