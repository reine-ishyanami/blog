---
title: GraalVM native-image 编译
date: 2023/7/22
tags:
 - JavaFX
 - SpringBoot
---

# GraalVM native-image 编译

## 基础环境配置

1. 下载[graalvm](https://www.graalvm.org/downloads/)

2. 配置 `JAVA_HOME` 和 `GRAALVM_HOME` 环境变量，内容为下载的 graalvm 的解压路径；将二进制文件，即 `%JAVA_HOME%\bin` 添加到 `Path` 环境变量中

3. 安装 native-image（新版graalvm已自带native-image，可省略此步骤）
   
   ```shell
   gu.cmd install native-image
   # 也可以先下载native-image的jar然后手动安装
   gu.cmd install -L <jar包路径>
   ```

4. 安装[wixtoolset](https://wixtoolset.org/)

5. 下载[Microsoft Visual Studio](https://visualstudio.microsoft.com/downloads/)
   
   > 选择以下组件
   > 
   > - 英语语言包
   > - MSVC v142 - VS 2019 C++x64/x86生成工具（或者之后的版本）
   > - Windows 10 SDK （10.0.19041.0 或者之后的版本）
   > - MSVC v142 - VS 2019 C++ x64/x86 Spectre缓解库（和上面的生成工具版本一样）
   > - 对v142生成工具的C++/CLI支持（或者之后的版本）
   > - Windows Universal CRT SDK

6. 查找对应的变量值
   
   - vs msvc 的 bin 目录 `D:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64`，其中版本号 `2022\Enterprise` 因选择的不同版本而有所差异，下面在配置环境变量时将 `D:\Program Files\Microsoft Visual Studio\2022\Enterprise` 简写为 `$vs` 
   
   - 通过注册表获取 windows sdk 的安装目录，注册表路径 `计算机\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Microsoft SDKs\Windows\v10.0`
     复制键为 `InstallationFolder` 的值，下面步骤会用到，简写为 `$windows10sdk` 

7. 需要添加的环境变量内容

    **其中没有对应变量名的值，对应其往上最近的变量名**
   
   | 变量名     | 内容                                            |
   |---------|-----------------------------------------------|
   | Path    | $vs\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64 |
   |         | $windows10sdk\bin\10.0.20348.0\x64            |
   | INCLUDE | $windows10sdk\Include\10.0.22000.0\ucrt       |
   |         | $windows10sdk\Include\10.0.22000.0\um         |
   |         | $windows10sdk\Include\10.0.22000.0\shared     |
   |         | $vs\VC\Tools\MSVC\14.35.32215\include         |
   | LIB     | $windows10sdk\Lib\10.0.22000.0\um\x64         |
   |         | $windows10sdk\Lib\10.0.22000.0\ucrt\x64       |
   |         | $vs\VC\Tools\MSVC\14.35.32215\lib\x64         |

8. 编译测试
   
   > 新建 `HelloWorld.java` 文件，内容如下
   
   ```java
   public class HelloWorld {
       public static void main(String[] args) {
           System.out.println("Hello, World!");
       }
   }
   ```
   
   > 编译测试 
   
   ```shell
   javac HelloWorld.java
   native-image ./HelloWorld
   ./HelloWorld # 出现Hello, World!代表环境配置成功
   ```

## SpringBoot 项目

此章节依赖于上一章节的环境配置

**GraalVM native-image 编译 SpringBoot3 项目**

[官方地址](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image)

### maven 项目

1. 创建 SpringBoot 项目，需要勾选 `GraalVM Native Support` 依赖

   > 项目的 `pom.xml` 会多出以下插件

   ```xml
   <plugins>
      <plugin>
         <groupId>org.graalvm.buildtools</groupId>
         <artifactId>native-maven-plugin</artifactId>
      </plugin>
      ...
   </plugins>
   ```

2. 在 `pom.xml` 中添加程序入口类

   ```xml
   <project>
     ...
     <properties>
         ...
         <mainClass>com.example.Application</mainClass>
     </properties>
     ...
   </project>
   ```

3. 执行命令

   ```shell
   ./mvnw -Pnative native:compile
   ```

4. 可在 `project\target` 文件夹中找到生成的可执行文件

### gradle 项目

1. 创建 SpringBoot 项目，需要勾选 `GraalVM Native Support` 依赖 

   > 项目的`build.gradle`会多出以下插件
   
   ```groovy
   plugins {
      ...
      id 'org.graalvm.buildtools.native' version '0.9.28'
   }
   ```

2. 执行命令 (gradle 项目无需手动配置主启动类)
   
   ```shell
   ./gradlew nativeCompile
   ```

3. 可在 `project\build\native\nativeCompile` 文件夹中找到生成的可执行文件

## JavaFX 项目

**GraalVM native-image 编译 JavaFX 项目**

请使用 IDEA 中的 JavaFX 创建项目

### maven 项目

1. [插件主页](https://github.com/gluonhq/gluonfx-maven-plugin)

2. `pom.xml` 配置

   ```xml
   <project>
     ...
     <build>
         ...
         <plugins>
             ...
             <plugin>
                 <groupId>org.openjfx</groupId>
                 <artifactId>javafx-maven-plugin</artifactId>
                 <version>0.0.8</version>
                 <executions>
                     <execution>
                     <!-- Default configuration for running with: mvn clean javafx:run -->
                     <id>default-cli</id>
                         <configuration>
                         <mainClass>com.reine.demo/com.reine.demo.HelloApplication</mainClass>
                         <launcher>app</launcher>
                         <jlinkZipName>app</jlinkZipName>
                         <jlinkImageName>app</jlinkImageName>
                         <noManPages>true</noManPages>
                         <stripDebug>true</stripDebug>
                         <noHeaderFiles>true</noHeaderFiles>
                         <!-- 
                             启动一次，对所有的UI控件进行功能测试，测试完成后正常关闭窗口，可以省略配置反射
                             反射配置文件生成在resources目录下的META-INF/native-image目录下（如异常关闭则不会生成）
                             如果项目文件比较多，不想配置反射或者不知道该怎么配置反射时，推荐使用这种方式
                          -->
                         <options>
                             <option>-agentlib:native-image-agent=config-merge-dir=src/main/resources/META-INF/native-image</option>
                         </options>
                         </configuration>
                     </execution>
                 </executions>
             </plugin>
             <plugin>
                 <groupId>com.gluonhq</groupId>
                 <artifactId>gluonfx-maven-plugin</artifactId>
                 <version>1.0.22</version>
                 <configuration>
                     <target>host</target>
                     <bundlesList>
                         <!--fxml文件-->
                         <list>com.example.hello-view</list>
                     </bundlesList>
                     <reflectionList>
                         <!--controller控制器-->
                         <list>com.example.HelloController</list>
                     </reflectionList>
                     <mainClass>com.example.HelloApplication</mainClass>
                     <verbose>false</verbose>
                 </configuration>
             </plugin>
         </plugins>
     </build>
   </project>    
   ```

3. 编译

   ```shell
   mvn javafx:run  # 第2步描述的启动一次，测试所有UI控件，生成反射文件
   
   mvn gluonfx:build # 执行此命令完成后可能需要等待一段时间，此命令执行完成后可以看的一个exe文件
   
   mvn gluonfx:nativerun # 运行生成的exe文件
   ```

### gradle 项目

1. [插件主页](https://github.com/gluonhq/gluonfx-gradle-plugin)

2. `build.gradle` 配置

   ```groovy
   buildscript {
     repositories {
         maven {
             url "https://plugins.gradle.org/m2/"
         }
     }
     dependencies {
         classpath 'com.gluonhq:gluonfx-gradle-plugin:1.0.22'
     }
   }
   apply plugin: 'com.gluonhq.gluonfx-gradle-plugin'    
   
   
   // 配置主启动类，记得修改成自己的主启动类的全限定名
   mainClassName = 'com.example.HelloApplication'
   
   // 启动一次，对所有的UI控件进行功能测试，测试完成后正常关闭窗口，可以省略配置反射
   // 反射配置文件生成在resources目录下的META-INF/native-image目录下（如异常关闭则不会生成）
   // 如果项目文件比较多，不想配置反射或者不知道该怎么配置反射时，推荐使用这种方式
   run {
     jvmArgs=["-agentlib:native-image-agent=config-merge-dir=src/main/resources/META-INF/native-image"]
   }
   
   // 一些自定义的插件配置，可以不写
   gluonfx {
     // 一般填host
     target = "host"
     bundlesList = ["com.example.hello-view"]
     reflectionList = ["com.example.HelloController"]
     verbose = false
   }
   ```

3. 编译

   ```shell
   gradle run  # 第2步描述的启动一次，测试所有UI控件，生成反射文件
   
   gradle nativeBuild # 执行此命令完成后可能需要等待一段时间，此命令执行完成后可以看的一个exe文件
   
   gradle nativeRun # 运行生成的exe文件
   ```


### 附录

> javafx gradle 项目的 windows graalvm 打包脚本(github action workflow)，需要先运行 gradle run 生成反射配置并上传到远程仓库

**相关链接**

[graalvm](https://github.com/marketplace/actions/github-action-for-graalvm)

[gradle](https://docs.github.com/zh/actions/publishing-packages/publishing-java-packages-with-gradle)

```yml
name: Windows Native-Image Build
# 定义在推送 tag 时触发工作流程与手动触发工作流程
on:
  push:
    tags:
      - '*'
  workflow_dispatch:
jobs:
  build:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      # 安装 GraalVM，配置JAVA_HOME GRAALVM_HOME
      - name: Setup GraalVM
        uses: graalvm/setup-graalvm@v1
        with:
          java-version: '21'
          distribution: 'graalvm-community'
          github-token: ${{ secrets.GITHUB_TOKEN }}
      - name: Validate Gradle wrapper
        uses: gradle/wrapper-validation-action@ccb4328a959376b642e027874838f60f8e596de3
      # 使用 Gradle 构建您的项目
      - name: GraalVM Native Build
        run:
          .\gradlew nativeBuild
      # 上传打包完成后的产品，以供下载
      - name: Upload
        uses: actions/upload-artifact@v2
        with:
          # 打包后的压缩包名称
          name: XXX
          path: build/gluonfx/x86_64-windows/*.exe
```