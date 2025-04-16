# Surge Framework

前端基础框架，采用纯TypeScript开发，提供了 Http 请求、WebSocket客户端、共享数据管理、通用错误处理、权限认证、工具库等核心功能。框架设计充分考虑了通用性，可无缝对接React、Vue、Angular等主流前端框架以及任意UI组件库。

## 安装方式

目前 Surge Framework 没有以npm包的形式发布，而是直接集成到了项目中。

以 Vite 项目为例，vite.config.ts 中添加以下配置：

```typescript
export default defineConfig({
  ...
  resolve: {
    alias: {
      {
        find: '@surge',
        replacement: resolve(__dirname, '../library/surge'),
      },
    },
  },
  ...
});
```

tsconfig.json 中添加以下配置：

```json
{
  "compilerOptions": {
    "paths": {
      "@surge/*": ["library/surge/*"]
      "@surge": ["library/surge"]
    }
  }
}
```

详细配置请参考 [Surge Admin 源码](https://github.com/surge-team/surge-admin)。

## 基础配置

> surge/config.ts

对 Surge Framework 的特性的默认配置，参数如下：

|config|default|intro|
|-|-|-|
|debugEnabled|import.meta.env.MODE !== 'production'|关闭的话 `utils.ts::debug` 函数将不会打印日志|
|surgeDebugEnabled|import.meta.env.MODE !== 'production'|关闭的话将不会打印 Surge Framework 相关日志<br />`当debugEnabled为false时本项无效`|

项目中可以引入 `import surgeConfig from 'surge/config.ts'` 文件，修改其中的配置。

## Http 请求

> surge/http/*
>
> surge/vue/useAsyncState.ts

这是对 [axios](https://github.com/axios/axios) 的二次封装，主要做了以下优化：

- 每次请求时，`config` 参数为必填项，且 config 中必须包含 `abortSignal` 选项，目的是显式声明请求的取消逻辑
- 通过`SmartAbort`和`useSmartAbort`函数，可以方便的实现请求的取消逻辑
- 设计了一套标准的Http请求配置模式，以解决一个项目接入多个API提供商时，配置混乱的问题
- 配合 `useAsyncState` 简化 Vue3 项目异步请求的编写逻辑，获取isLoading、isReady等状态

可以参考 [Surge Admin 源码](https://github.com/surge-team/surge-admin) 中的 `src/services` 目录，了解具体的使用方式。
> surge/webSocketClient/index.ts
>
> surge/webSocketClient/webSocketClientError.ts

我将查看 WebSocket 客户端的代码，然后为您编写详细的中文说明文档。


Read file: library/surge/webSocketClient/index.ts
基于代码分析，我将为您编写一个详细的 WebSocket 客户端说明文档：

## WebSocket 客户端

> surge/webSocketClient/index.ts
>
> surge/webSocketClient/webSocketClientError.ts

### 简介

WebSocket 客户端是一个功能完整的 WebSocket 连接管理工具，提供了连接管理、自动重连、事件处理和错误处理等核心功能。该客户端使用 TypeScript 编写，支持现代浏览器环境。

### 主要特性

- 自动建立和管理 WebSocket 连接
- 支持自动重连机制，可配置最大重试次数和间隔时间
- 提供完整的事件处理回调（连接状态变化、消息接收、错误处理）
- 内置连接状态验证和错误处理
- 支持手动关闭连接
- 提供连接状态查询接口

### 安装和使用

#### 基本用法

```typescript
const client = new WebSocketClient({
  url: 'ws://example.com',
  maxReconnectAttempts: 5,
  reconnectInterval: 5000,
  statusChangedHandler: (status) => console.log('连接状态:', status),
  messageHandler: (data) => console.log('收到消息:', data)
});

// 发送消息
client.send('Hello');

// 关闭连接
client.close();
```

#### 配置选项

```typescript
interface Options {
  url: string | URL;                    // WebSocket 服务器地址
  protocols?: string | string[];        // 可选的协议
  maxReconnectAttempts?: number;        // 最大重连次数，默认 3
  reconnectInterval?: number;           // 重连间隔时间（毫秒），默认 3000
}
```

#### 事件处理器

```typescript
interface HandlerOptions extends Options {
  // 连接状态变化处理器
  statusChangedHandler?: (
    status: ConnectStatus,
    payload: { isManualClose?: boolean },
    client: WebSocketClient
  ) => void;

  // 消息接收处理器
  messageHandler?: (data: string, client: WebSocketClient) => void;

  // 错误处理器
  errorHandler?: (error: WebSocketClientError, client: WebSocketClient) => void;
}
```

### 连接状态

客户端定义了三种连接状态：

```typescript
const ConnectStatus = {
  CONNECTED: 'CONNECTED',    // 已连接
  CONNECTING: 'CONNECTING',  // 连接中
  CLOSED: 'CLOSED'          // 已关闭
};
```

### 错误处理

客户端定义了以下标准错误类型：

```typescript
const Errors = {
  SEND_UNCONNECTED: 'WSC_SEND_UNCONNECTED',     // 未连接时发送消息
  SEND_UNKNOWN_ERROR: 'WSC_SEND_UNKNOWN_ERROR', // 发送消息时发生未知错误
  UNKNOWN_ERROR: 'WSC_UNKNOWN_ERROR'            // 通用 WebSocket 错误
};
```

### 重要说明

1. 自动重连机制：
   - 仅在非手动关闭的情况下触发
   - 达到最大重试次数后停止重连
   - 手动关闭（isManualClose=true）会阻止自动重连

2. 调试日志：
   - 所有调试日志通过 surgeDebug 工具输出
   - 包含连接状态变化、重连尝试和错误信息

3. 状态管理：
   - 使用内部状态存储确保状态一致性
   - 提供状态查询接口方便外部监控

### 最佳实践

1. 错误处理：
   ```typescript
   const client = new WebSocketClient({
     url: 'ws://example.com',
     errorHandler: (error, client) => {
       console.error('WebSocket 错误:', error);
       // 实现自定义错误处理逻辑
     }
   });
   ```

2. 状态监控：
   ```typescript
   const client = new WebSocketClient({
     url: 'ws://example.com',
     statusChangedHandler: (status, payload, client) => {
       if (status === 'CONNECTED') {
         console.log('连接成功');
       } else if (status === 'CLOSED') {
         console.log('连接关闭', payload.isManualClose ? '手动关闭' : '自动关闭');
       }
     }
   });
   ```

3. 消息处理：
   ```typescript
   const client = new WebSocketClient({
     url: 'ws://example.com',
     messageHandler: (data, client) => {
       try {
         const message = JSON.parse(data);
         // 处理消息
       } catch (e) {
         console.error('消息解析错误:', e);
       }
     }
   });
   ```

### 注意事项

1. 确保在组件卸载时调用 `close()` 方法关闭连接
2. 合理设置重连参数，避免过于频繁的重连请求
3. 实现适当的错误处理逻辑，提高应用稳定性
4. 注意处理消息的序列化和反序列化
5. 在发送消息前检查连接状态

好的，我将重新组织文档结构，使用"##"作为第一级标题：

## 聊天客户端

> surge/webSocketClient/chatClient.ts

### 简介

聊天客户端（ChatClient）是基于 WebSocket 客户端扩展的专门用于聊天功能的客户端实现。它提供了群组管理、结构化消息处理等聊天相关的核心功能。

### 主要特性

- 继承基础 WebSocket 客户端的所有功能
- 支持群组加入/退出操作
- 提供结构化的消息解析和基于事件的处理机制
- 为所有操作生成唯一的 actionId
- 支持 JSON 格式的消息交互

### 安装和使用

#### 基本用法

```typescript
const chat = new ChatClient({
  url: 'wss://chat.example.com',
  joinedGroupHandler: (group, payload) => console.log(`加入群组 ${group}`, payload),
  chatMessageHandler: (payload) => console.log('收到消息:', payload)
});

// 加入群组
const joinActionId = chat.joinGroup('general');

// 发送消息
const messageActionId = chat.chat({ text: '你好，世界！' });

// 退出群组
const leaveActionId = chat.leaveGroup('general');
```

#### 事件类型

```typescript
const ChatResponseEvents = {
  JOINED_GROUP: 'join',    // 加入群组成功
  LEAVED_GROUP: 'leave',   // 退出群组成功
  MESSAGE: 'message',      // 收到消息
  ERROR: 'error'          // 发生错误
};
```

#### 处理器配置

```typescript
interface ChatHandlerOptions {
  // 加入群组成功处理器
  joinedGroupHandler?: (
    groupName: string,
    payload: any,
    actionId: string,
    client: ChatClient
  ) => void;

  // 退出群组成功处理器
  leavedGroupHandler?: (
    groupName: string,
    payload: any,
    actionId: string,
    client: ChatClient
  ) => void;

  // 聊天消息处理器
  chatMessageHandler?: (
    payload: any,
    actionId: string,
    client: ChatClient
  ) => void;
}
```

### 消息格式

#### 请求消息格式

```typescript
// 加入群组
{
  event: 'join',
  actionId: string,
  groupName: string
}

// 退出群组
{
  event: 'leave',
  actionId: string,
  groupName: string
}

// 发送消息
{
  event: 'message',
  actionId: string,
  payload: any
}
```

#### 响应消息格式

```typescript
{
  event: ChatResponseEvents,
  actionId: string,
  groupName?: string,
  payload?: any,
  error?: { code: string }
}
```

### 错误处理

聊天客户端扩展了基础 WebSocket 客户端的错误类型：

```typescript
const ChatErrors = {
  ...Errors,
  INVALID_MESSAGE_DATA: 'WSCC_INVALID_MESSAGE_DATA',    // 消息格式错误
  INVALID_MESSAGE_EVENT: 'WSCC_INVALID_MESSAGE_EVENT'   // 未知的事件类型
};
```

### 最佳实践

#### 错误处理示例

```typescript
const chat = new ChatClient({
  url: 'wss://chat.example.com',
  errorHandler: (error, client) => {
    switch (error.code) {
      case ChatErrors.INVALID_MESSAGE_DATA:
        console.error('消息格式错误');
        break;
      case ChatErrors.INVALID_MESSAGE_EVENT:
        console.error('未知的事件类型');
        break;
      default:
        console.error('其他错误:', error);
    }
  }
});
```

#### 消息处理示例

```typescript
const chat = new ChatClient({
  url: 'wss://chat.example.com',
  chatMessageHandler: (payload, actionId, client) => {
    // 处理不同类型的消息
    if (payload.type === 'text') {
      console.log('收到文本消息:', payload.content);
    } else if (payload.type === 'image') {
      console.log('收到图片消息:', payload.url);
    }
  }
});
```

#### 群组管理示例

```typescript
const chat = new ChatClient({
  url: 'wss://chat.example.com',
  joinedGroupHandler: (groupName, payload, actionId) => {
    console.log(`成功加入群组: ${groupName}`);
    // 可以在这里更新UI或进行其他操作
  },
  leavedGroupHandler: (groupName, payload, actionId) => {
    console.log(`成功退出群组: ${groupName}`);
    // 可以在这里更新UI或进行其他操作
  }
});
```

### 注意事项

#### 连接管理

- 确保在组件卸载时调用 `close()` 方法关闭连接
- 合理处理重连机制，避免频繁断开重连

#### 消息处理

- 所有消息都应该是有效的 JSON 格式
- 消息必须包含 `event` 字段
- 建议对消息内容进行类型检查和验证

#### 错误处理

- 实现完整的错误处理逻辑
- 对不同类型的错误采取相应的处理措施
- 记录错误日志以便调试

#### 性能优化

- 避免在消息处理器中执行耗时操作
- 合理使用 actionId 进行消息追踪
- 注意内存泄漏问题

#### 安全性

- 确保 WebSocket 连接使用 WSS 协议
- 对敏感消息进行加密处理
- 实现适当的身份验证机制

## 共享数据管理器

> - surge/shareDataManager.ts
> - surge/vue/useShareDataManagerStore.ts

该组件适合身份认证、用户资料、个性化配置等信息的存取及跨页面共享，具备以下特性：

- 可以自定义通道名称，以区分不同的数据类别
- 支持 LocalStorage/SessionStorage/Cookie(浏览器关闭时清空Cookie) 持久化存储，或者相互切换
- 支持数据变更的事件通知
- 支持同域跨页面的数据变更通知
- vue/useShareDataManagerStore.ts 中提供了 Vue3 的响应式数据管理方案

可以参考 [Surge Admin 源码](https://github.com/surge-team/surge-admin) 中的 `src/library/currentAccount` 目录，了解具体的使用方式。

## 权限认证

> surge/permissionValidator.ts

Surge 定义了一套标准的权限认证方案，考虑了前后端的集成和解偶，以及开发者的使用体验。
一个典型的前端权限认证流程如下：

- 前端登录成功后，会获取当前账户的权限列表，并缓存到本地
- 前端在显示功能菜单、页面按钮等UI组件时，会通过权限认证函数检查当前账户是否有权限访问

Surge Framework 仅包含了权限认证的代码（权限管理在 [Surge Admin](surge-admin.md) 中实现），下面是权限认证的几个例子：

```typescript
// 定义账户具备的权限列表
const objectPermissions = [
  'system.user.list',
  'system.user.create',
  'erp.financial.*', // 与 'erp.financial' 等价
  'blog',
  'book.create',
  'book.modify',
];

// 创建权限认证对象
const permissionValidator = new PermissionValidator(objectPermissions);

// 检查权限 (xxx.* 表示具备 xxx 或 xxx 下的任意一个权限即可)
console.log(permissionValidator.checkPermissions('system.user')); // false
console.log(permissionValidator.checkPermissions('system.user.*')); // true
console.log(permissionValidator.checkPermissions('system.user.list')); // true
console.log(permissionValidator.checkPermissions('system.user.create')); // true
console.log(permissionValidator.checkPermissions('system.user.update')); // false
console.log(permissionValidator.checkPermissions('blog')); // true
console.log(permissionValidator.checkPermissions('blog.*')); // true
console.log(permissionValidator.checkPermissions('book')); // false
console.log(permissionValidator.checkPermissions('book.*')); // true
console.log(permissionValidator.checkPermissions('book.create')); // true
console.log(permissionValidator.checkPermissions('book.modify')); // true

// 如果传递数组，则表示具备数组中任意一个权限即可
console.log(permissionValidator.checkPermissions(['system.user.*', 'crm.customer'])); // true
console.log(permissionValidator.checkPermissions(['crm.customer', 'crm.visitlog'])); // false

// 当然，也可以通过 `checkEveryPermissions` 方法检查是否同时具备所有权限
console.log(permissionValidator.checkEveryPermissions(['system.user.list', 'book.*'])); // true
console.log(permissionValidator.checkEveryPermissions(['system.user.*', 'crm.customer'])); // false
console.log(permissionValidator.checkEveryPermissions(['erp.financial', 'crm.visitlog'])); // false

// 空字符串无法验证通过
console.log(permissionValidator.checkPermissions('')); // false
console.log(permissionValidator.checkPermissions([''])); // false
console.log(permissionValidator.checkPermissions(['erp.financial', ''])); // true
console.log(permissionValidator.checkEveryPermissions(['erp.financial', ''])); // false

// `*` 通配符表示具备任何一个权限即可
console.log(permissionValidator.checkPermissions('*')); // true

// 但没有任何权限时，将无法通过认证
permissionValidator.setPermissions([]); // 清空权限列表
console.log(permissionValidator.checkPermissions('*')); // false
```

可以参考 [Surge Admin 源码](https://github.com/surge-team/surge-admin) 中的 `src/library/permissionManager.ts` 文件，了解具体的使用方式。

## 通用错误处理

> - surge/errorHandler.ts
> - surge/vue/createErrorHandler.ts

前端开发往往只在axios拦截器中处理API错误，其它如组件错误、websocket错误、Promise错误等很容易被忽略。Surge Framework 中定义了一套标准的错误处理方案，目的是全自动的处理所有运行时错误，99%的业务代码中不需要写try...catch逻辑，除非开发者有个性化的错误处理需求。

核心原理是通过 window.addEventListener('error') 和 window.addEventListener('unhandledrejection') 捕获所有未处理的错误，而 Vue3 项目使用 `registerVueErrorHandler.ts` 捕获 Vue.config.errorHandler 未处理的错误，然后捕获到的 Error 对象交给 `ErrorHandler.processor` 函数进行处理，这个函数通过提前配置好的 `错误类型->错误处理器` 映射表，分发给对应的处理器进行处理。

下面是 Vue3 项目的示例代码：

```typescript
// ------------------------------
// 错误类型定义、错误处理配置、创建错误处理器
// ------------------------------

// 定义一个私有的API错误类型
class PrivateApiError extends Error {
  static readonly errorName = 'PrivateApiError';

  code: string;

  constructor(error: any) {
    super(error.response?.data?.message || error.message || '未知错误');

    // 从 response 中获取错误码
    this.code = error.response?.data?.code;
  }
}

// 定义API错误处理器
function apiErrorProcessor(error: any) {
  // 通过Message组件显示错误信息
  Message.error(`${error.message} (错误码: ${error.code})`);
}

// 定义默认错误处理器
function defaultErrorProcessor(error: any) {
  console.error(error);
}

// 定义错误收集器
function errorCollector(error: any) {
  // Send error to server
}

// 创建错误处理器
const errorHandler = new ErrorHandler({
  [PrivateApiError.name]: [apiErrorProcessor, errorCollector], // 定义API错误处理
  '.': [defaultErrorProcessor, errorCollector], // 遇到上面未定义的错误类型时使用该设置
});

// ------------------------------
// 注册Vue错误处理器
// ------------------------------
const app = createApp(App);
app.use(registerVueErrorHandler(errorHandler));



// ------------------------------
// 业务代码例子1：点击页面中的提交按钮时请求API发生错误
//
// 如果发生错误时会发生以下结果：
// 1. axios拦截器抛出 PrivateApiError 异常并终止后续代码运行
// 2. registerVueErrorHandler 会捕获到任何未处理的异常，交给 errorHandler.processor 处理
// 3. errorHandler.processor 会根据配置交给 apiErrorProcessor 和 errorCollector 两个处理器处理。
// 4. apiErrorProcessor 通过 Message 组件显示错误信息，errorCollector 收集错误信息并发送到服务器
// ------------------------------

// 定义axios拦截器，将API错误转换为PrivateApiError并抛出异常
axios.interceptors.response.use(undefined, (error) => {
  throw new PrivateApiError(error);
});

// 点击页面中的提交按钮时请求API
const handleSubmit = async () => {
  axios.get('/api/v1/user/list');

  Message.success('操作成功');
};

// ------------------------------
// 业务代码例子2：图片裁剪时发生错误
//
// 如果发生错误时会发生以下结果：
// 1. 抛出 Error 异常并终止后续代码运行
// 2. registerVueErrorHandler 会捕获到任何未处理的异常，交给 errorHandler.processor 处理
// 3. 因为没有配置过 Error 的错误处理器，errorHandler.processor 会交给 defaultErrorProcessor 和 errorCollector 两个处理器处理。
// 4. defaultErrorProcessor 通过 console.error 打印错误信息，errorCollector 收集错误信息并发送到服务器
// ------------------------------
function imageClip(file: Blob, width: number, height: number) {
  throw new Error('图片裁剪失败');
}

const handleImageClip = () => {
  const image = imageClip(file, 100, 100);

  Message.success('操作成功');
};
```

实际业务过程中会有更多考虑，比如错误文言要结合i18n和错误码表生成，可以参考 [Surge Admin 源码](https://github.com/surge-team/surge-admin) 中的 `src/library/errorHandler` 目录，了解具体的使用方式。

## 工具库

> surge/utils.ts

Surge Framework 中定义了一些常用的工具函数，请查看源码自行了解。

## 多框架支持

Surge Framework 本身是Typescript编写，支持任何前端框架，但为了方便 Vue3 项目开发，在 `surge/vue` 目录中定义了一套 Vue3 的组件，包括：

- useAsyncState: 异步请求组件
- useShareDataManagerStore: 数据共享管理器(ShareDataManager)的 Vue3 Hook 封装
- registerVueErrorHandler: 错误处理器(ErrorHandler)的 Vue3 install 封装

如果你有其它框架的需求（如 React 或 Angular），目前有两个方法：

- 在你的项目中复制一份 `surge/vue` 目录，然后根据你的框架需求修改代码
- 在 Surge Framework 中创建你使用的框架的目录，并参考 Vue3 的组件自己实现一套。

当然 `surge/vue` 目录的代码中引用了 Vue3，你需要手动删除这个目录。