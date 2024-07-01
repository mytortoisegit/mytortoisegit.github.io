---
title: springmvc笔记
date: 2024-06-25 09:50:26
tags: springmvc
---



### 基于 Spring Boot 的博客系统设计

#### 1. 项目结构

```
blog-system
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           └── blog
│   │   │               ├── BlogApplication.java
│   │   │               ├── controller
│   │   │               │   ├── BlogController.java
│   │   │               ├── model
│   │   │               │   ├── Blog.java
│   │   │               ├── repository
│   │   │               │   ├── BlogRepository.java
│   │   │               ├── service
│   │   │               │   ├── BlogService.java
│   │   ├── resources
│   │   │   ├── static
│   │   │   ├── templates
│   │   │   │   ├── blog
│   │   │   │   │   ├── create.html
│   │   │   │   │   ├── edit.html
│   │   │   │   │   ├── list.html
│   │   │   │   │   ├── view.html
│   │   │   └── application.properties
└── pom.xml
```

#### 2. 依赖配置

在`pom.xml`中添加以下依赖：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>blog-system</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>blog-system</name>
    <description>Blog System using Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

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

#### 3. 配置文件

在`src/main/resources/application.properties`中添加数据库和Redis的配置：

```properties
# MySQL database configuration
spring.datasource.url=jdbc:mysql://localhost:3306/blogdb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root

# JPA configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect

# Redis configuration
spring.redis.host=localhost
spring.redis.port=6379

# Thymeleaf configuration
spring.thymeleaf.cache=false

# MongoDB configuration (if using MongoDB instead of MySQL)
# spring.data.mongodb.uri=mongodb://localhost:27017/blogdb
```

#### 4. 主应用程序类

**BlogApplication.java**：

```java
package com.example.blog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BlogApplication {

    public static void main(String[] args) {
        SpringApplication.run(BlogApplication.class, args);
    }

}
```

#### 5. 创建模型类

**Blog.java**：

```java
package com.example.blog.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Blog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String content;

    // Getters and Setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
```

#### 6. 创建Repository接口

**BlogRepository.java**：

```java
package com.example.blog.repository;

import com.example.blog.model.Blog;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BlogRepository extends JpaRepository<Blog, Long> {
}
```

#### 7. 创建Service类

**BlogService.java**：

```java
package com.example.blog.service;

import com.example.blog.model.Blog;
import com.example.blog.repository.BlogRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class BlogService {

    @Autowired
    private BlogRepository blogRepository;

    @Cacheable(value = "blogs", key = "#id")
    public Blog getBlogById(Long id) {
        return blogRepository.findById(id).orElse(null);
    }

    @Cacheable(value = "blogs")
    public List<Blog> getAllBlogs() {
        return blogRepository.findAll();
    }

    @CacheEvict(value = "blogs", allEntries = true)
    public Blog saveBlog(Blog blog) {
        return blogRepository.save(blog);
    }

    @CacheEvict(value = "blogs", allEntries = true)
    public void deleteBlog(Long id) {
        blogRepository.deleteById(id);
    }
}
```

#### 8. 创建Controller类

**BlogController.java**：

```java
package com.example.blog.controller;

import com.example.blog.model.Blog;
import com.example.blog.service.BlogService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Controller
@RequestMapping("/blogs")
public class BlogController {

    @Autowired
    private BlogService blogService;

    @GetMapping
    public String listBlogs(Model model) {
        List<Blog> blogs = blogService.getAllBlogs();
        model.addAttribute("blogs", blogs);
        return "blog/list";
    }

    @GetMapping("/{id}")
    public String viewBlog(@PathVariable Long id, Model model) {
        Blog blog = blogService.getBlogById(id);
        model.addAttribute("blog", blog);
        return "blog/view";
    }

    @GetMapping("/create")
    public String createBlogForm(Model model) {
        model.addAttribute("blog", new Blog());
        return "blog/create";
    }

    @PostMapping
    public String saveBlog(@ModelAttribute Blog blog) {
        blogService.saveBlog(blog);
        return "redirect:/blogs";
    }

    @GetMapping("/edit/{id}")
    public String editBlogForm(@PathVariable Long id, Model model) {
        Blog blog = blogService.getBlogById(id);
        model.addAttribute("blog", blog);
        return "blog/edit";
    }

    @PostMapping("/{id}")
    public String updateBlog(@PathVariable Long id, @ModelAttribute Blog blog) {
        blog.setId(id);
        blogService.saveBlog(blog);
        return "redirect:/blogs";
    }

    @GetMapping("/delete/{id}")
    public String deleteBlog(@PathVariable

 Long id) {
        blogService.deleteBlog(id);
        return "redirect:/blogs";
    }
}
```

#### 9. 创建Thymeleaf模板

**list.html**：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Blog List</title>
</head>
<body>
    <h1>Blog List</h1>
    <a href="/blogs/create">Create New Blog</a>
    <ul>
        <li th:each="blog : ${blogs}">
            <a th:href="@{'/blogs/' + ${blog.id}}"
               th:text="${blog.title}">Blog Title</a>
            <a th:href="@{'/blogs/edit/' + ${blog.id}}">Edit</a>
            <a th:href="@{'/blogs/delete/' + ${blog.id}}">Delete</a>
        </li>
    </ul>
</body>
</html>
```

**view.html**：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Blog View</title>
</head>
<body>
    <h1 th:text="${blog.title}">Blog Title</h1>
    <p th:text="${blog.content}">Blog Content</p>
    <a href="/blogs">Back to Blog List</a>
</body>
</html>
```

**create.html**：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Create Blog</title>
</head>
<body>
    <h1>Create Blog</h1>
    <form action="#" th:action="@{/blogs}" th:object="${blog}" method="post">
        <label>Title: <input type="text" th:field="*{title}" /></label>
        <label>Content: <textarea th:field="*{content}"></textarea></label>
        <button type="submit">Save</button>
    </form>
</body>
</html>
```

**edit.html**：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Edit Blog</title>
</head>
<body>
    <h1>Edit Blog</h1>
    <form action="#" th:action="@{${'/blogs/' + ${blog.id}}}" th:object="${blog}" method="post">
        <label>Title: <input type="text" th:field="*{title}" /></label>
        <label>Content: <textarea th:field="*{content}"></textarea></label>
        <button type="submit">Save</button>
    </form>
</body>
</html>
```

#### 10. 启动项目

启动主类`BlogApplication.java`，打开浏览器访问`http://localhost:8080/blogs`，即可查看博客列表，创建、编辑和删除博客。

#### 11. 总结

通过Spring Boot、Thymeleaf和Redis等技术栈，我们构建了一个简单但功能完整的博客系统。进一步扩展可以包括用户认证、权限管理、评论系统等功能，持续优化和完善系统。