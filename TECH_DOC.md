# alibabaTestChat 简易技术文档

## 1. 项目简介
`alibabaTestChat` 是一个基于 Spring Boot + Spring AI Alibaba（DashScope） 的示例项目，演示了：

- 文本对话（普通调用）
- 文本对话（SSE 流式输出）
- 图片理解（多模态）
- 前端简易聊天页面（`simp-chat.html`）

默认启动端口为 `8080`，前端页面通过同源接口调用后端聊天能力。

---

## 2. 技术栈

### 2.1 后端
- `Java 17`
- `Spring Boot 3.4.5`
- `spring-boot-starter-web`（REST API）
- `spring-ai 1.0.0`
- `spring-ai-alibaba 1.0.0.2`
- `spring-ai-alibaba-starter-dashscope`（接入阿里云百炼/通义模型）
- `Project Reactor (Flux)`（流式返回）

### 2.2 前端
- 原生 `HTML + CSS + JavaScript`
- `EventSource`（SSE）实现流式聊天

### 2.3 文档与扩展依赖
- `Swagger 2 (springfox-swagger2)`（注解已存在，未看到完整 Swagger 配置类）
- `MCP` 相关依赖（项目中已引入，当前代码中未见主要业务使用）

---

## 3. 核心配置
配置文件：`src/main/resources/application.properties`

当前关键项：

- `spring.application.name=alibabaai`
- `server.port=8080`
- `spring.ai.dashscope.api-key=...`
- `spring.ai.dashscope.chat.options.model=qwen-max`

> 建议：不要在仓库中明文保存 API Key，优先使用环境变量或外部配置。

可参考（示例）：

```properties
spring.ai.dashscope.api-key=${DASHSCOPE_API_KEY}
```

---

## 4. 主要代码结构

- 启动类：`src/main/java/com/example/alibabaai/AlibabaaiApplication.java`
- 聊天控制器：`src/main/java/com/example/alibabaai/controller/chat/SimpChatController.java`
- 模型演示控制器：`src/main/java/com/example/alibabaai/controller/chat/ChatModelController.java`
- 前端页面：`src/main/resources/static/simp-chat.html`

Spring Boot 会自动托管 `static` 下的页面资源，可通过浏览器直接访问页面文件。

---

## 5. 后端接口说明

## 5.1 `SimpChatController`

### `GET /chat`
- 功能：普通文本聊天（一次性返回）
- 参数：`input`
- 返回：`String`

### `GET /stream/chat`
- 功能：流式文本聊天
- 参数：`input`
- 返回：`text/event-stream`（SSE）
- 说明：前端通过 `EventSource` 逐段接收模型输出，形成“打字机”效果

### `POST /image/analyze/upload`
- 功能：图片分析（多模态）
- 参数：
  - `prompt`（可选，默认“请分析这张图片的内容”）
  - `file`（必填，图片文件）
- 返回：`String`
- 说明：内部使用 `qwen-vl-max-latest` 并开启多模态参数

## 5.2 `ChatModelController`（`/model` 前缀）

### `GET /model/simple/chat`
- 功能：基础模型调用示例

### `GET /model/stream/chat`
- 功能：流式调用示例（Flux）

### `GET /model/tokens`
- 功能：返回输出文本及 token 使用信息

### `GET /model/custom/chat`
- 功能：动态指定模型与采样参数（`topP`、`topK`、`temperature` 等）
- 说明：代码中参数优先级高于配置文件中的默认参数

---

## 6. 前端简易聊天功能说明
页面文件：`src/main/resources/static/simp-chat.html`

该页面聚焦 `SimpChatController` 的流式接口（`/stream/chat`），实现了一个最小可用聊天体验：

- 输入问题（`textarea`）
- 点击“流式对话”发起 SSE 请求
- 实时拼接模型返回内容并展示
- 显示状态文案（准备就绪、流式输出中、连接结束、错误提示）
- “停止流式”按钮可主动关闭当前 EventSource
- “清空”按钮可清空输出并重置状态

核心前端逻辑：

1. 点击“流式对话”后校验输入
2. 创建 `EventSource(/stream/chat?input=...)`
3. 在 `onmessage` 中将新片段追加到输出区
4. 在 `onerror` 或手动停止时关闭连接

---

## 7. 运行方式（Maven）
在项目根目录执行：

```powershell
.\mvnw.cmd spring-boot:run
```

启动后可访问：

- 前端页面：`http://localhost:8080/simp-chat.html`
- 示例接口：`http://localhost:8080/chat?input=你好`

---

## 8. 已知注意事项

- `pom.xml` 中存在 BOM 以普通依赖方式声明的情况（可后续优化为 `dependencyManagement`），当前文档仅描述现状。
- 项目中使用 `springfox-swagger2 2.9.2`，与较新 Spring Boot 版本组合时可能需要额外兼容处理。
- 流式接口依赖 SSE，若经过网关/代理请确认未关闭流式转发。
- 图片分析接口会校验 `Content-Type` 是否为 `image/*`。

---

## 9. 快速体验建议

1. 先验证 `GET /chat` 是否可用（快速确认 Key 和模型配置）
2. 再打开 `simp-chat.html` 体验流式输出
3. 最后测试 `/image/analyze/upload` 验证多模态能力

