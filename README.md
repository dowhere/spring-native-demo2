# spring-native-demo2
使用GraalVM便以为本地可执行文件

# 1. 系统要求
在安装GraalVM native-image编译器之前，需要一些先决条件。

本机映像编译器有多种发行版，这里我们重点介绍这两种：
	基于GraalVM开源存储库和LabsJDK的GraalVM CE
	Bellsoft Liberica Native Image Kit（NIK）基于GraalVM开源存储库和Liberica JDK


要在MacOS或Linux上安装本机映像编译器，建议使用SDKMAN：
*	安装SDKMAN。
*	安装一个GraalVM本机映像发行版，GraalVM CE（grl后缀）或Bellsoft Liberica NIK（NIK后缀），下面是Liberica NIK Java 11变体：sdk安装Java 21.3.0。r11 nik
*	确保使用新安装的JDK with sdk使用java 21.3.0。r11 nik
*	运行gu install native image将本机映像扩展引入JDK。

或者，如果您使用的是Microsoft Windows，则可以从GraalVM或Liberica NIK手动安装版本。如果需要，不要忘记适当地设置JAVA_HOME/PATH，并运行gu install native image以引入本机映像扩展。
windows上需要安装Visual Studio 构建工具


# 验证Spring Boot版本
提示：Spring Native 0.11.1 仅支持 Spring Boot 2.6.2，因此如有必要，请更改版本。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.2</version>
    <relativePath/>
</parent>
```

# 添加 Spring Native 依赖项
`org.springframework.experimental:spring-native`提供了`@NativeHint`等本机配置API，以及作为本机映像运行spring应用程序所需的其他必需类。您只需要使用Maven显式地指定它。
```xml
<dependencies>
    <!-- ... -->
    <dependency>
        <groupId>org.springframework.experimental</groupId>
        <artifactId>spring-native</artifactId>
        <version>0.11.1</version>
    </dependency>
</dependencies>
```

# 添加Spring AOT插件
Spring AOT插件执行改进本机映像兼容性和占用空间所需的提前转换。
```xml
<build>
    <plugins>
        <!-- ... -->
        <plugin>
            <groupId>org.springframework.experimental</groupId>
            <artifactId>spring-aot-maven-plugin</artifactId>
            <version>0.11.1</version>
            <executions>
                <execution>
                    <id>generate</id>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
                <execution>
                    <id>test-generate</id>
                    <goals>
                        <goal>test-generate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```


# 添加本机构建工具插件
GraalVM提供Gradle和Maven插件，从构建中调用本机映像编译器。下面的示例添加了一个本机配置文件，在打包阶段触发插件：

```xml
<profiles>
    <profile>
        <id>native</id>
        <dependencies>
            <!-- Required with Maven Surefire 2.x -->
            <dependency>
                <groupId>org.junit.platform</groupId>
                <artifactId>junit-platform-launcher</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <version>0.9.9</version>
                    <extensions>true</extensions>
                    <executions>
                        <execution>
                            <id>build-native</id>
                            <goals>
                                <goal>build</goal>
                            </goals>
                            <phase>package</phase>
                        </execution>
                        <execution>
                            <id>test-native</id>
                            <goals>
                                <goal>test</goal>
                            </goals>
                            <phase>test</phase>
                        </execution>
                    </executions>
                    <configuration>
                        <!-- ... -->
                    </configuration>
                </plugin>
                <!-- Avoid a clash between Spring Boot repackaging and native-maven-plugin -->
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <configuration>
                        <classifier>exec</classifier>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```



# Maven 存储库
将构建配置为包含spring本机依赖项的发行版存储库，以及包含Gradle的Maven Central本机构建工具库，如下所示：
```xml
<repositories>
    <!-- ... -->
    <repository>
        <id>spring-release</id>
        <name>Spring release</name>
        <url>https://repo.spring.io/release</url>
    </repository>
</repositories>
```

插件也是如此
```xml
<pluginRepositories>
    <!-- ... -->
    <pluginRepository>
        <id>spring-release</id>
        <name>Spring release</name>
        <url>https://repo.spring.io/release</url>
    </pluginRepository>
</pluginRepositories>
```



# 3. 构建原生应用程序
本机应用程序可以按如下方式构建：
```bash
$ mvn -Pnative -DskipTests package
```
此命令在目标目录中创建一个包含Spring Boot应用程序的本机可执行文件。
