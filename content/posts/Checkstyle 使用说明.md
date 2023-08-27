---
title: Checkstyle 使用说明
date: 2023-06-12
categories: ['技术']
alias: checkstyle
draft: false
---

### 一、什么是 Checkstyle？

Checkstyle 是一个检查 Java 代码风格的开发工具，可以使一个多开发人员的项目保持一致的代码风格，更多介绍 [查看官网](https://checkstyle.org/)。

### 二、支持哪些代码风格？

Checkstyle 默认支持 Sun Code Conventions 和 Google Java Style，我们选择基于 Google 的风格做些修改。

- 调整所有缩进x2, 和 IDEA 保持一致。

修改原因：

> Google 的默认缩进为 2 个空格，不太习惯，而且层级深了也看不清楚。
相信大部分 Java 程序员从接触 Java 开始都是 4 个空格缩进。
搞不清楚 Google 为什么这样做，不过有些开源项目也有使用 2 缩进的。
但是我们和 IDEA 的缩进风格保持一致。
> 

最终文件：

[checkstyle.xml](/checkstyle/checkstyle.xml)

### 三、将最终文件导入 IDEA。

将 checkstyle.xml 导入 IDEA，这样一来在 IDEA 中格式化代码就会和 Checkstyle 一致。

1. 下载文件至本地电脑
2. 打开 IDEA 设置页面
    1. 选择 Editor -> Code Style
    2. 点击 Scheme 后面的齿轮图标
    3. 选择 Import Schema -> Checkstyle configuration -> 选择下载好的 checkstyle.xml 文件

### 四、Gradle 集成

与 Gradle 集成，我们就可以使用 check 命令检查代码风格，配合 ci 工具，就可以强制检查。

下面是 Gradle8 的配置样例

```groovy
plugins {
    id 'java'
    id 'checkstyle'
}
checkstyle {
    toolVersion = '10.12.1'
    maxWarnings = 0
    maxErrors = 0
    configFile = rootProject.file('checkstyle.xml')
}
group = 'cool.leenstx.demo'
version = '1.0.0'
test {
    useJUnitPlatform {
        includeTags("fast")
    }
}

```

注意：上面的配置是默认 checkstyle.xml 文件在 Gradle 项目根目录下。

然后在项目根目录下执行：

```bash
./gradlew check
```

也可以在 IDEA 中 Gradle 界面上进行操作。执行 check 命令后 Checkstyle 会自动生成一份报告文件（html）在 build -> reports 目录下。

### 五、配置 CI

以我们的 Gitlab 为例。

根据下面的配置修改为你自己的 CI 配置并保存为项目根目录下 `.gitlab-ci.xml` 即可。

CI 模版（当代码合并时执行代码风格检查，检查通过后开始构建）：

```yaml
image: eclipse-temurin:8

cache:
   paths:
      - /root/.m2

stages:
   - lint
   - build

checkstyle:
   allow_failure: false
   stage: lint
   script:
      - echo "代码风格检查"
      - ./gradlew check
   only:
      - merge_requests

compile:
   stage: build
   script:
      - echo "开始构建"
      - ./gradlew build -x check -x test
   only:
      - merge_requests

```

其实（略做配置）会员版 Gitlab 在合并代码界面就可以直接查看 Checkstyle 自动生成的代码质量报告，有兴趣可以自己研究下。

### 六、扩展

- Spotless: Checkstyle 只是检查，Spotless 可以直接做一些修改，以符合代码风格。
- EditConfig: [查看官网](https://editorconfig.org/)。