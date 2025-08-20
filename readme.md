# 项目说明

## 一、项目简介
`ylz-spring-starter-parent` 是易联众新技术应用研究中心开源实验室为 Spring Boot 技术栈项目打造的 **Maven 父工程**，聚焦标准化、规范化的技术底座建设，提供以下核心价值：  
- **依赖版本集中管控**：统一管理 Spring、数据库驱动、测试框架、代码质量工具等依赖版本，规避多项目依赖冲突。  
- **工程规范预置**：通过 `dependencyManagement` 约束依赖范围，结合 `pluginManagement` 固化单元测试、覆盖率检查、代码扫描等研发流程，让子项目开箱即用。  
- **技术生态整合**：覆盖领域驱动设计（DDD）、工具开发等场景所需的基础依赖（如 Lombok、Knife4j 接口文档、Kubernetes 客户端），加速项目落地。  


## 二、适用场景
- **多模块/多项目统一治理**：作为团队内 Spring Boot 项目的父 POM，子项目继承后无需重复定义依赖版本，专注业务开发。  
- **研发流程标准化**：预置单元测试、覆盖率检查、Sonar 代码扫描等插件配置，推动团队代码质量基线落地。  
- **新技术探索支撑**：为领域驱动设计实践、AI 工具开发、K8s 集成等场景提供依赖与配置基础，降低技术调研成本。  


## 三、核心功能与配置说明

### 1. 依赖管理（`dependencyManagement`）
通过 `<dependencyManagement>` 统一管控版本，**子项目引用时无需声明版本**，示例：  
```xml
<!-- 子项目中引用，自动继承父工程版本 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```  

**关键依赖清单**：  
| 依赖分类          | 核心组件                              | 版本管理键                     |
|-------------------|---------------------------------------|--------------------------------|
| 基础工具          | Lombok、ASM                          | `lombok.version`、`asm.version` |
| Spring 生态       | Spring Cloud、SpringDoc 接口文档     | `springcloud.version`、`springdoc.version` |
| 数据库            | MySQL Connector/J                    | `mysql.driver.version`         |
| 接口文档          | Knife4j（适配 Jakarta）              | `knife4j.version`              |
| 云原生集成        | Kubernetes Java 客户端               | `kubernetes.version`           |
| 测试框架          | JUnit 5、Mockito                     | `junit.version`、`mockito.version` |
| 代码质量工具      | Jacoco（覆盖率）、Sonar（扫描）      | `jacoco.version`、`sonar.version` |  


### 2. 构建插件（`pluginManagement`）
预置研发流程必备插件，**子项目可直接继承配置**，覆盖以下场景：  

#### （1）单元测试与覆盖率  
- **Maven Surefire**：执行 JUnit 5 测试用例，支持排除特定测试（如集成测试），配置：  
  ```xml
  <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <!-- 自动继承父工程版本与配置 -->
  </plugin>
  ```  
- **Jacoco**：统计单元测试覆盖率，强制要求：  
  - 行覆盖率 ≥ 80%  
  - 分支覆盖率 ≥ 70%  
  - 方法覆盖率 ≥ 75%  
  若不满足，构建会失败（通过 `jacoco:check` 约束）。  


#### （2）代码质量扫描  
通过 `sonar-maven-plugin` 对接 SonarQube，执行命令 `mvn sonar:sonar` 即可触发代码扫描，需在 CI/CD 或本地配置 SonarQube 服务地址（`sonar.host.url`）及令牌。  


## 四、快速开始

### 1. 子项目继承父工程
在子项目 `pom.xml` 中添加：  
```xml
<parent>
    <groupId>com.ylz.springframework</groupId>
    <artifactId>ylz-spring-starter-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</parent>
```  

### 2. 引入基础依赖（示例）
```xml
<dependencies>
    <!-- 必选：Lombok 简化 POJO 开发 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    
    <!-- 可选：Knife4j 接口文档（适配 Jakarta） -->
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
    </dependency>
    
    <!-- 可选：Kubernetes 客户端 -->
    <dependency>
        <groupId>io.kubernetes</groupId>
        <artifactId>client-java</artifactId>
    </dependency>
</dependencies>
```  


## 五、最佳实践

### 1. 子项目配置建议
- **依赖精简**：仅引入业务所需依赖，避免因过度继承导致包体积膨胀。  
- **个性化插件配置**：若需覆盖父工程插件参数（如调整 Jacoco 覆盖率阈值），在子项目 `pom.xml` 中重写 `<plugin>` 配置即可（通过 `pluginManagement` 机制，子配置优先级更高）。  


### 2. 代码质量流程落地
- **本地开发**：执行 `mvn clean test` 自动运行单元测试并生成 Jacoco 覆盖率报告（路径：`target/site/jacoco/index.html`）。  
- **CI/CD 集成**：结合 GitOps 流程，在 Pipeline 中加入 `mvn verify`（触发 Jacoco 检查）、`mvn sonar:sonar`（代码扫描）步骤，强制质量门禁。  


## 六、常见问题

### 1. 依赖冲突如何排查？
执行 `mvn dependency:tree -Dverbose` 生成依赖树，重点检查：  
- 不同版本的 Spring Boot/Spring Cloud 依赖（父工程已对齐版本，子项目请勿手动指定版本）。  
- 第三方库传递依赖的冲突（可通过 `<exclusions>` 排除冗余依赖）。  


### 2. 单元测试报错 `--add-opens java.base/java.lang=ALL-UNNAMED`？
父工程已在 `maven-surefire-plugin` 中配置该参数（适配 Java 模块系统），若仍报错：  
- 检查子项目 `Java 版本` 是否与父工程一致（父工程为 Java 21）。  
- 确认 `maven-surefire-plugin` 继承自父工程（未被子项目覆盖）。  


## 七、参与贡献
欢迎通过以下方式共建：  
- **Issues**：提交依赖升级建议、插件配置优化需求 → [仓库 Issues](https://github.com/ylzoslab/ylz-springframework-parent/issues)。
- **Discussions**：交流领域驱动设计实践、AI 技术应用经验 → [仓库 Discussions](https://github.com/ylzoslab/ylz-springframework-parent/discussions)。
- **代码提交**：Fork 仓库后提交 Pull Request，补充新依赖、优化插件配置等。  


通过标准化的父工程建设，我们致力于让技术团队聚焦业务创新，减少重复造轮子成本。期待与社区共同完善这份技术底座，推动开源实验室的项目高质量落地！  
