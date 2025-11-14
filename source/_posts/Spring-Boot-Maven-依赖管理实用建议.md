title: Spring Boot Maven 依赖管理实用建议
tags: []
categories: []
date: 2025-11-14 12:13:07
---
# Spring Boot Maven 依赖管理实用建议



## 1. 依赖查找的实用方法

### 1.1 使用 Maven 中央仓库搜索
- **官方网站**：https://mvnrepository.com/
- **使用技巧**：
  - 直接搜索功能名称（如 "redis"、"mysql"、"jwt"）
  - 查看依赖的使用量（Usage）和最新版本
  - 注意查看依赖的维护状态和更新频率

### 1.2 Spring Initializr 快速生成
```bash
# 访问 https://start.spring.io/
# 选择需要的依赖，自动生成 pom.xml
```

**推荐流程**：
1. 在 Spring Initializr 中勾选常用依赖
2. 下载项目查看生成的 pom.xml
3. 复制需要的依赖配置到你的项目

### 1.3 IDE 智能提示
- **IDEA**：在 pom.xml 中输入 `<dependency>` 后，Ctrl+Space 会有提示
- **Eclipse**：使用 Maven 插件的依赖搜索功能

## 2. 常用依赖速查表

### 2.1 Web 开发基础
```xml
<!-- Spring Boot Web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 参数校验 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 2.2 数据库相关
```xml
<!-- MySQL 驱动 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- MyBatis Plus（推荐） -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.5</version>
</dependency>

<!-- 或使用 MyBatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>

<!-- JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- 数据库连接池 Druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.21</version>
</dependency>
```

### 2.3 Redis 缓存
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- Lettuce 连接池 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

### 2.4 工具类库
```xml
<!-- Hutool 工具类（强烈推荐） -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.24</version>
</dependency>

<!-- Lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>

<!-- Apache Commons Lang3 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
</dependency>
```

### 2.5 API 文档
```xml
<!-- Swagger 3 (Springdoc) -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>

<!-- 或使用 Knife4j（国产增强版） -->
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    <version>4.4.0</version>
</dependency>
```

### 2.6 JSON 处理
```xml
<!-- Jackson（Spring Boot 默认包含） -->
<!-- Fastjson2 -->
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>2.0.43</version>
</dependency>
```

### 2.7 认证授权
```xml
<!-- JWT -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>

<!-- Sa-Token（国产，更简单） -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot3-starter</artifactId>
    <version>1.37.0</version>
</dependency>
```

## 3. 依赖冲突解决方案

### 3.1 查看依赖树
```bash
# 查看完整依赖树
mvn dependency:tree

# 查看特定依赖
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core

# 输出到文件
mvn dependency:tree > dependency.txt
```

### 3.2 使用 IDEA 依赖分析
1. 打开 pom.xml
2. 右键 → Diagrams → Show Dependencies
3. 可视化查看依赖关系和冲突

### 3.3 排除冲突依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- 排除默认的 Tomcat -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 使用 Jetty 替代 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

### 3.4 强制指定版本
```xml
<properties>
    <!-- 统一版本管理 -->
    <jackson.version>2.15.3</jackson.version>
</properties>

<dependencyManagement>
    <dependencies>
        <!-- 强制使用指定版本 -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 4. 推荐的 pom.xml 模板结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 1. 继承 Spring Boot 父项目 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <!-- 2. 项目基本信息 -->
    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
    <version>1.0.0</version>
    <name>My Project</name>
    <description>Project Description</description>

    <!-- 3. 属性配置 -->
    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- 自定义版本 -->
        <mybatis-plus.version>3.5.5</mybatis-plus.version>
        <hutool.version>5.8.24</hutool.version>
    </properties>

    <!-- 4. 依赖管理（可选，用于多模块） -->
    <dependencyManagement>
        <dependencies>
            <!-- 统一版本控制 -->
        </dependencies>
    </dependencyManagement>

    <!-- 5. 依赖配置 -->
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 第三方依赖 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>

        <!-- 测试依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- 6. 构建配置 -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

## 5. 学习建议

### 5.1 循序渐进
1. **第一阶段**：只添加最基础的依赖（web、数据库、lombok）
2. **第二阶段**：根据需求逐步添加（redis、jwt、swagger）
3. **第三阶段**：学习依赖管理和版本控制

### 5.2 参考优秀开源项目
推荐学习的项目：
- **RuoYi**：https://gitee.com/y_project/RuoYi-Vue
- **Jeecg-Boot**：https://github.com/jeecgboot/jeecg-boot
- **Pig**：https://gitee.com/log4j/pig

### 5.3 建立自己的依赖库
创建一个文档，记录常用依赖：
```
功能 | GroupId | ArtifactId | Version | 说明
-----|---------|------------|---------|-----
MySQL | com.mysql | mysql-connector-j | runtime | 数据库驱动
Redis | org.springframework.boot | spring-boot-starter-data-redis | - | 缓存
```

## 6. 常见问题速查

| 问题 | 解决方案 |
|------|---------|
| 不知道需要什么依赖 | 查看 Spring Boot 官方文档的 Starters 列表 |
| 找不到 Maven 坐标 | 使用 mvnrepository.com 搜索 |
| 版本不兼容 | 查看 Spring Boot 兼容性矩阵 |
| 依赖冲突 | 使用 `mvn dependency:tree` 分析并排除 |
| 下载速度慢 | 配置阿里云 Maven 镜像 |

## 7. 配置阿里云镜像（提升下载速度）

在 Maven 的 `settings.xml` 中配置：
```xml
<mirrors>
    <mirror>
        <id>aliyunmaven</id>
        <mirrorOf>central</mirrorOf>
        <name>阿里云公共仓库</name>
        <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
</mirrors>
```