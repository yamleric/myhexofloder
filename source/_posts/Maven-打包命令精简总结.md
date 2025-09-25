title: Maven 打包命令精简总结
tags: []
categories: []
date: 2025-09-25 13:58:42
---
以下是 Maven 打包 Java 应用程序的常用命令总结，涵盖不同场景需求：

---

### **基础打包命令**
```bash
mvn clean package
```
- **作用**：清理编译缓存 + 编译代码 + 运行测试 + 打包（生成 JAR/WAR）
- **输出位置**：`target/` 目录
- **适用场景**：标准打包流程（需提前配置好 `pom.xml`）

---

### **跳过测试打包**
```bash
mvn clean package -DskipTests
```
- **作用**：跳过所有测试（不编译也不执行）
- **替代命令**（仅跳过执行，仍编译测试）：
  ```bash
  mvn clean package -Dmaven.test.skip=true
  ```

---

### **生成可执行 JAR（含依赖）**
需配合插件使用（二选一）：

#### 1. 使用 `maven-shade-plugin`（推荐）
```bash
mvn clean package
```
**`pom.xml` 配置示例**：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.4.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.example.MainApp</mainClass> <!-- 替换主类 -->
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### 2. 使用 `maven-assembly-plugin`
```bash
mvn clean package
```
**`pom.xml` 配置示例**：
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.5.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <archive>
                    <manifest>
                        <mainClass>com.example.MainApp</mainClass> <!-- 替换主类 -->
                    </manifest>
                </archive>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

---

### **生成 WAR 包（Web 项目）**
```bash
mvn clean package
```
**前提**：在 `pom.xml` 中设置打包类型为 `war`：
```xml
<packaging>war</packaging>
```

---

### **安装到本地仓库**
```bash
mvn clean install
```
- **作用**：打包 + 将生成的 JAR/WAR 安装到本地 Maven 仓库（`~/.m2/repository`）
- **用途**：供其他本地项目依赖

---

### **自定义打包文件名**
在 `pom.xml` 中指定最终包名：
```xml
<build>
    <finalName>my-custom-app-name</finalName>
</build>
```
执行后生成：`target/my-custom-app-name.jar`

---

### **多环境打包（如 dev/test/prod）**
1. **在 `pom.xml` 中定义环境配置**：
   ```xml
   <profiles>
       <profile>
           <id>dev</id>
           <properties>
               <env>development</env>
           </properties>
       </profile>
       <profile>
           <id>prod</id>
           <properties>
               <env>production</env>
           </properties>
           <activation>
               <activeByDefault>true</activeByDefault> <!-- 默认激活 -->
           </activation>
       </profile>
   </profiles>
   ```
2. **指定环境打包**：
   ```bash
   mvn clean package -P prod  # 使用 prod 配置
   ```

---

### **常用命令速查表**
| **命令** | **作用** |
|------------------------|--------------------------------|
| `mvn clean` | 清理 `target` 目录 |
| `mvn compile` | 编译主代码 |
| `mvn test` | 运行单元测试 |
| `mvn package` | 打包（不清理历史构建） |
| `mvn clean package` | 清理+编译+测试+打包 |
| `mvn install` | 打包并安装到本地仓库 |
| `mvn verify` | 运行集成测试 |
| `mvn deploy` | 发布到远程仓库（需配置） |

---

### **注意事项**
1. 生成可执行 JAR 时，务必在插件中配置 `<mainClass>`；
2. 若项目依赖其他模块，使用 `mvn install` 先安装依赖到本地仓库；
3. 多模块项目需在根目录执行命令（自动构建子模块）。

掌握这些命令，即可覆盖 90% 的 Java 应用打包场景！ 🚀