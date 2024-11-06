# Surge Admin

## 项目结构

- `.husky` Git 提交规范
- `.vscode` VSCode 开发规范
- `config` Vite 配置
- `library` 库文件
  - `compute_md5` Rust开发的MD5计算库，封装为WebAssembly模块
  - `surge` Surge Framework 核心代码
- `src` 项目源码
  - `assets` 静态资源，包括图片、字体、全局样式表等
    - `images` 图片
    - `styles` 全局样式表
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

## 文件命名规范

- 文件/目录名应具有描述性和实际意义
- 采用小驼峰命名法(camelCase)，即首字母小写
- 特殊用途的文件/目录使用下划线(`_`)作为前缀，例如`src/views/_layout/`
- 国际化相关的文件/目录需遵循locale标准格式：使用连字符(`-`)连接，语言代码小写，地区代码大写，如`zh-CN`

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

Surge Admin 继承了 Surge Framework 的[通用错误处理](surge-framework#通用错误处理)机制。详细实现请参考 `src/library/errorHandler` 目录。

值得注意的是，当 axios 响应拦截器检测到 token 无效或过期(INVALID_OR_EXPIRED_TOKEN)时，系统会自动登出并刷新页面，随后重定向至登录界面。相关代码可查看 `src/services/private/index.ts`。

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

#### WebSocket

开发中...

#### WebWorker

开发中...

#### WebAssembly

开发中...
