# Surge Admin

Surge Admin 是一套管理 Site 的基础框架，它基于 Surge Framework 开发，并在 Arco Pro 的基础上进行了全面重构和优化，从以下四个维度实现了重大升级：

- 技术层面：技术栈包括 Surge Framework、Vue3、Arco Design、Vue Router 和 i18n 等
- 功能层面：作为管理Site基础框架，集成了权限管理、用户管理、菜单配置、日志记录、消息通知、系统设置等核心模块
- 设计层面：严格遵循设计规范，为所有功能模块提供专业的 Figma 设计文档和完整的文档说明
- 测试层面：具备完善的测试体系，包含单元测试、集成测试等多维度的测试用例和文档

Surge Admin 作为基础框架，不仅为 Mini 基盘和 MiniApp 团队的未来项目提供支持，同时也会持续吸收和整合各个项目中的优秀功能和特性，不断完善和进化。

## 项目结构

!> 该文件结构仅供参考，最新版 Surge Admin 可能会有所不同。

- `.husky` Git 提交规范
- `.vscode` VSCode 开发规范
- `config` Vite 配置
- `library` 库文件
  - `surge` Surge Framework 核心代码
- `src` 项目源码
  - `assets` 静态资源，包括图片、字体、全局样式表等
    - `images` 图片
    - `style` 全局样式表
  - `config` 存放App配置或默认参数
    - `routes` 路由配置
    - `appSettings.ts` App特性开关
    - `defaultSettings.ts` 用户个性化设置的默认值
  - `i18n` 国际化配置
    - `en-US` 英文配置
    - `ja-JP` 日文配置
    - `zh-CN` 中文配置
    - `index.ts` 国际化配置入口
  - `library` Hooks、状态管理、Utils、Vue指令等各类与视图无关的代码
    - `currentAccount` 当前账号状态管理工具
    - `errorHandler` 错误处理工具
    - `routerManager` 路由实例及管理工具
    - `permissionManager.ts` 权限管理工具
    - `useResponsiveState.ts` 网页响应式管理工具
    - `useReloadRoute.ts` 路由刷新工具
    - `utils.ts` 工具函数
  - `services` 存放第三方服务，如API、WebSocket、WebWorker等
    - `private` 本App私有API
  - `views` 页面视图组件
    - `_components` 公共组件库
    - `_layouts` 布局组件
    - `_not-found` 404页面
    - `_redirect` 重定向页面
    - `login` 登录页面
    - `...` 其他页面
- `.env.development` 开发环境变量
- `.env.production` 生产环境变量
- `.env.stg` STG环境变量
- `.env.test` 测试环境变量
- `.env.uat` UAT环境变量
- `.eslintignore` ESLint 忽略文件
- `.eslintrc.js` ESLint 配置
- `.gitignore` Git 忽略文件
- `.prettierignore` Prettier 忽略文件
- `.prettierrc.js` Prettier 配置
- `.stylelintrc.js` Stylelint 配置
- `babel.config.js` Babel 配置
- `commitlint.config.js` Commitlint 配置
- `components.d.ts` VUE 全局组件类型声明
- `index.html` 入口文件
- `package.json` 项目配置
- `pnpm-lock.yaml` pnpm 锁定文件
- `shims-tsx.d.ts` TSX 类型声明
- `tsconfig.json` TS 配置

## 路由

Surge Admin 采用纯本地路由配置方案，在 Arco Pro 路由（基于 `vue-router`）的基础上进行了深度优化。该方案完全摒弃了服务端路由的概念，并与 `permissionManager.ts` 实现了无缝集成。

主要涉及以下文件：

- `src/config/routes/index.ts` 路由配置的主入口文件
- `src/config/routes/constants.ts` 路由相关常量定义，供 `src/library/routerManager` 使用
- `src/library/routerManager` 路由管理核心模块
  - `index.ts` 路由实例的主入口
  - `listener.ts` 基于 mitt 的路由监听器，用于优化路由性能
  - `constants.ts` 常用路由名称的常量定义（如 LOGIN_ROUTE.name、REDIRECT_ROUTE.name），方便开发使用
  - `typings.d.ts` RouteMeta 类型声明文件，**内含详尽注释说明**

## 权限管理

Surge Admin 采用基于账号、角色、权限的三层权限体系。其中账号可以拥有多个角色，每个角色又可以拥有多个权限。

在前端开发中，我们主要关注账号和权限的对应关系，角色主要用于显示目的。当用户登录成功后，API 会返回该账号所拥有的完整权限列表。

Surge Admin 提供了三种权限认证方式:

- 在路由配置中使用 meta(RouteMeta) 进行路由级权限控制
- 在 TypeScript 代码中调用 `permissionManager.ts::permissionValidator` 进行编程式权限验证  
- 在 Vue3 模板中使用 `v-permissions` 指令进行声明式权限控制

更多实现细节请参考 `src/library/permissionManager.ts` 文件中的注释说明。

## 国际化

Surge Admin 采用 vue-i18n 的国际化方案，支持中、英、日三种语言，支持日期、时间、数字、货币、百分比等多种格式化方式。

更多实现细节请参考 `src/i18n` 目录中代码。

## 错误处理

Surge Admin 继承了 Surge Framework 的[通用错误处理](surge-framework#通用错误处理)机制。

内置的错误处理规则：

- 当 axios 响应拦截器检测到 token 无效或过期(INVALID_OR_EXPIRED_TOKEN)时，系统会自动登出并刷新页面，随后重定向至登录界面。相关代码可查看 `src/services/private/index.ts`。
- 当 axios 响应拦截器检测到其它错误码时，会抛出 `PrivateApiError` 错误，并弹出错误提示。相关代码可查看 `src/services/private/privateApiError.ts`。
- 表单校验错误时，如无特殊需要，一定要抛出 `FormValidationError` 错误，它会弹出错误提示。
```typescript
const handleSubmit = async ({ errors }: { errors: any }) => {
  static readonly errorName = 'FormValidationError';

  if (errors) throw new FormValidationError();

  visibleConfirm.value = true;
};
```

更多实现细节请参考 `src/library/errorHandler` 目录中代码。

## 账号管理

Surge Admin 通过 `currentAccount` 模块统一管理登录账号信息，包括:
- 认证信息
- 个人信息 
- 个性化设置

该模块基于 `shareDataManager` 实现数据共享和响应式状态管理。详情请参考 `src/library/currentAccount` 目录。

## 时区处理

由于 Arco Design Vue 的 DatePicker 组件默认使用浏览器时区，且暂不支持自定义时区设置，Surge Admin 采取了以下解决方案:

1. 在页面顶部醒目位置显示当前浏览器时区，避免用户误操作
2. 前后端统一采用 UNIX 时间戳进行数据传输，从根本上规避时区问题

## 页面响应式布局

Surge Admin 通过 `assets/style/breakpoints.less` 文件定义了不同设备的屏幕尺寸断点，开发者可以在页面样式中直接引用这些断点变量来实现响应式布局。可以参考login页面的源代码(`src/views/login/index.vue`)。

同时，我们还提供了 `useResponsiveState.ts` 响应式状态管理模块，方便开发者在 Vue3 组件中监听和响应屏幕尺寸的变化。通过这种方式，可以更优雅地实现页面的自适应布局。可以参考layout页面的源代码(`src/views/_layout/default-layout.vue`)。

## Services

Web应用的本质是与第三方服务进行交互，它们不只局限于REST API，还包括GraphQL API、WebSocket、WebWorker、WebAssembly等等。Surge Admin 在 `src/services` 目录下实现了对这些服务的统一管理。

#### REST API

Surge Admin 使用 Surge Framework 提供的 [Http 请求库](surge-framework#http-请求)来实现 REST API 调用。具体开发流程如下：

1. 在 `src/services` 目录下新建 API 服务目录。例如 `src/services/private` (private 表示本项目的 API)。根据不同的提供商这个名字也不相同，例如 `src/services/aws` 表示 AWS API。
2. 在 `src/services/private/index.ts` 文件中实例化并导出 Http 类，完成基础配置和拦截器定义。
3. 在 `src/services/private/entities/` 目录下定义 API 相关的类型声明，包括请求参数和响应数据类型。
4. 将功能相近的 API 封装到同一个对象中，并将其保存在 `src/services/private/` 目录下的独立文件中。

具体请参考 `src/services/private` 目录下的代码。

#### WebSocket Client

Surge H5 使用 Surge Framework 提供的 [WebSocket 客户端](surge-framework#websocket-客户端) 来实现与 WebSocket 服务器的通信。业务代码应该存放到 `src/services` 目录下，以 `ws` 为前缀的文件夹名称表示该文件夹下的代码是 WebSocket 相关的。
