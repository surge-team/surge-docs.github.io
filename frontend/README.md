# 简介

!> 本文档主要介绍 Surge Frontend 的架构设计和使用方法，可能缺少详细的API说明。建议结合源码和实际项目案例一起阅读，以便更好地理解和应用。

Surge Frontend 是 Surge 项目的核心部分，主要负责为前端项目提供基础的开发框架和工具。由以下4部分组成。

### Surge Framework

前端基础框架，采用纯TypeScript开发，提供了 Http 请求、共享数据管理、通用错误处理、权限认证、工具库等核心功能。框架设计充分考虑了通用性，可无缝对接React、Vue、Angular等主流前端框架以及任意UI组件库。

进入 [Surge Framework](surge-framework.md) 查看详情。

### Surge Admin

Surge Admin 是一套管理 Site 的基础框架，它基于 Surge Framework 开发，并在 Arco Pro 的基础上进行了全面重构和优化，从以下四个维度实现了重大升级：

- 技术层面：技术栈包括 Surge Framework、Vue3、Arco Design、Vue Router 和 i18n 等
- 功能层面：作为管理Site基础框架，集成了权限管理、用户管理、菜单配置、日志记录、消息通知、系统设置等核心模块
- 设计层面：严格遵循设计规范，为所有功能模块提供专业的 Figma 设计文档和完整的文档说明
- 测试层面：具备完善的测试体系，包含单元测试、集成测试等多维度的测试用例和文档

Surge Admin 作为基础框架，不仅为 Mini 基盘和 MiniApp 团队的未来项目提供支持，同时也会持续吸收和整合各个项目中的优秀功能和特性，不断完善和进化。

进入 [Surge Admin](surge-admin.md) 查看详情。

### Surge H5

为 Mobile H5 和 MiniApp 应用提供开发框架，基于Surge Framework开发，采用Vue3、Tailwind CSS、Vue Router、i18n等技术栈。

目前正在开发中。

### Surge Web

为 PC Web 应用提供开发框架，基于Surge Framework开发，采用Vue3、Tailwind CSS、Vue Router、i18n等技术栈。

目前正在开发中。
