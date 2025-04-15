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

## WebSocket 客户端

> surge/webSocketClient/index.ts
>
> surge/webSocketClient/webSocketClientError.ts

### 功能概述
`WebSocketClient` 是一个封装了 WebSocket 客户端功能的类，提供以下核心能力：
- 建立 WebSocket 连接并管理连接状态
- 实现自动重连机制（可配置最大重试次数和间隔）
- 提供连接事件回调（连接成功、关闭、消息接收、错误）
- 封装消息发送功能（包含状态验证和错误处理）
- 支持手动关闭连接
- 提供连接状态查询

### 错误类型
```typescript
export const Errors = {
  SEND_UNCONNECTED: 'WSC_SEND_UNCONNECTED',      // 在未连接状态下尝试发送消息
  SEND_UNKNOWN_ERROR: 'WSC_SEND_UNKNOWN_ERROR',  // 消息发送时发生未知错误
  UNKNOWN_ERROR: 'WSC_UNKNOWN_ERROR'             // 通用WebSocket错误
};
```

### 配置选项
| 参数名 | 类型 | 必填 | 默认值 | 描述 |
|--------|------|------|--------|------|
| url | string \| URL | 是 | - | WebSocket服务器地址 |
| protocols | string \| string[] | 否 | - | 子协议 |
| maxReconnectAttempts | number | 否 | 3 | 最大重连尝试次数 |
| reconnectInterval | number | 否 | 3000 | 重连间隔(毫秒) |

### 事件处理器
| 处理器 | 参数 | 描述 |
|--------|------|------|
| connectedHandler | (client: WebSocketClient) => void | 连接成功时触发 |
| closeHandler | (isManualClose: boolean, client: WebSocketClient) => void | 连接关闭时触发 |
| messageHandler | (data: string, client: WebSocketClient) => void | 收到消息时触发 |
| errorHandler | (error: WebSocketClientError, client: WebSocketClient) => void | 发生错误时触发 |

### 核心方法
#### `send(data: string)`
发送消息到服务器
• **参数**:
  • `data`: 要发送的消息内容
• **可能抛出的错误**:
  • `WSC_SEND_UNCONNECTED`: 未连接状态下尝试发送
  • `WSC_SEND_UNKNOWN_ERROR`: 发送过程中发生未知错误

#### `close()`
手动关闭连接
• 设置`isManualClose`标志防止自动重连
• 触发`closeHandler`回调

#### `status`
获取当前连接状态
• **返回值**: `number | undefined`
  • `0` (CONNECTING): 连接中
  • `1` (OPEN): 已连接
  • `2` (CLOSING): 关闭中
  • `3` (CLOSED): 已关闭
  • `undefined`: 未初始化

### 使用示例
```typescript
const client = new WebSocketClient({
  url: 'ws://example.com',
  maxReconnectAttempts: 5,
  reconnectInterval: 5000,
  connectedHandler: (client) => console.log('Connected'),
  messageHandler: (data) => console.log('Received:', data)
});

client.send('Hello');
client.close();
```

## WebSocket 聊天客户端

> surge/webSocketClient/chatClient.ts

### 功能概述
`ChatClient` 继承自 `WebSocketClient`，提供聊天特定功能：
• 扩展基础WebSocket客户端以支持聊天协议
• 实现群组加入/离开功能（带操作跟踪）
• 提供结构化消息解析和基于事件的处理
• 为所有操作生成唯一actionId

### 扩展的错误类型
```typescript
export const ChatErrors = {
  ...Errors,
  INVALID_MESSAGE_DATA: 'WSCC_INVALID_MESSAGE_DATA',  // 消息JSON格式无效
  INVALID_MESSAGE_EVENT: 'WSCC_INVALID_MESSAGE_EVENT' // 无法识别的事件类型
};
```

### 响应事件类型
```typescript
export const ChatResponseEvents = {
  JOINED_GROUP: 'join',    // 加入群组
  LEAVED_GROUP: 'leave',   // 离开群组
  MESSAGE: 'message',      // 聊天消息
  ERROR: 'error'           // 错误响应
};
```

### 配置选项
继承自`WebSocketClient`的所有选项，并替换以下处理器：

| 处理器 | 参数 | 描述 |
|--------|------|------|
| joinedGroupHandler | (groupName: string, payload: any, actionId: string, client: ChatClient) => void | 成功加入群组时触发 |
| leavedGroupHandler | (groupName: string, payload: any, actionId: string, client: ChatClient) => void | 成功离开群组时触发 |
| chatMessageHandler | (payload: any, actionId: string, client: ChatClient) => void | 收到聊天消息时触发 |

### 核心方法
#### `joinGroup(groupName: string): string`
请求加入聊天群组
• **参数**:
  • `groupName`: 目标群组名称
• **返回值**: 操作ID (用于跟踪此操作)

#### `leaveGroup(groupName: string): string`
请求离开聊天群组
• **参数**:
  • `groupName`: 目标群组名称
• **返回值**: 操作ID (用于跟踪此操作)

#### `chat(payload: any): string`
发送聊天消息
• **参数**:
  • `payload`: 消息内容(将被JSON序列化)
• **返回值**: 操作ID (用于跟踪此操作)

### 使用示例
```typescript
const chat = new ChatClient({
  url: 'wss://chat.example.com',
  joinedGroupHandler: (group, payload, actionId) => 
    console.log(`Joined ${group}`, payload),
  chatMessageHandler: (payload) => 
    console.log('Message:', payload)
});

const actionId = chat.joinGroup('general');
chat.chat({ text: 'Hello world' });
```

### 消息结构
```json
{
  "event": "join|leave|message|error",
  "actionId": "唯一操作ID",
  "groupName": "群组名称(可选)",
  "payload": {}, // 有效负载(可选)
  "error": { "code": "错误代码" } // 仅当event=error时存在
}
```

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