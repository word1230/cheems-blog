---
date: '2026-04-30T21:16:20+08:00'
draft: false
slug: 'e6c14ddf'
type: posts
title: 'C Chat Agent(langchain4j基础学习)'
collections: ""
categories: []
---

<!--more-->

## 基础对话生成


### 定义ChatModel







### 构建消息


## AIservice

### 通用方式


#### 定义接口


#### 定义工厂, 创建service

本质是代理模式
### springboot特别支持(声明式)


## 会话记忆-ChatMemory


### 通过Aiservice 绑定记忆窗口

### 自定义会话记忆机制



### 通过memory id 隔离对话


## 结构化输出


### 三种方式


参考: https://glaforge.dev/posts/2024/11/18/data-extraction-the-many-ways-to-get-llms-to-spit-json-content/

prompt

函数调用

```java
var model = VertexAiGeminiChatModel.builder()
    .project(System.getenv("PROJECT_ID"))
    .location(System.getenv("LOCATION"))
    .modelName("gemini-1.5-pro-002")
    .toolCallingMode(ToolCallingMode.ANY)
    .allowedFunctionNames(List.of("extractNameAndAgeFromBiography"))
    .build();
```

JSON 模式方法

```java
var model = VertexAiGeminiChatModel.builder()
    .project(System.getenv("PROJECT_ID"))
    .location(System.getenv("LOCATION"))
    .modelName("gemini-1.5-pro-002")
    .responseMimeType("application/json")
    .build();
```


## JSON schema模式

```java
var model = VertexAiGeminiChatModel.builder()
    .project(System.getenv("PROJECT_ID"))
    .location(System.getenv("LOCATION"))
    .modelName("gemini-1.5-pro-002")
    .responseMimeType("application/json")
    .responseSchema(Schema.newBuilder()
        .setType(Type.OBJECT)
        .putProperties("name", Schema.newBuilder()
            .setType(Type.STRING)
            .setDescription(
                "The name of the person described in the biography")
            .build())
        .putProperties("age", Schema.newBuilder()
            .setType(Type.INTEGER)
            .setDescription(
                "The age of the person described in the biography")
            .build())
        .build())
        .addAllRequired(List.of("name", "age"))
    .build();
```


## RAG


## Tool Call


### 定义

### AIservice 中进行配置


## MCP

