---
name: hbuildx-uni-app-skill
description: HBuilderX uni-app/uni-app X 开发辅助，帮助生成符合官方规范的代码
---

## 技能概述

提供 uni-app/uni-app X 完整开发规范参考，帮助生成符合 DCloud 官方规范的代码。

## 开发环境要求

- 使用最新版 **HBuilderX 3.9+**（uni-app X 需要 HBuilderX 3.9 及以上版本支持）
- 开发 uni-app X 需要勾选 "uni-app x" 选项创建项目

## 创建项目

1. 打开 HBuilderX，菜单 `文件 -> 新建 -> 项目`
2. 选择 `uni-app` 类型，勾选 `uni-app x` 选项
3. 输入项目名称，选择模板，点击创建

## 完整规范参考

已提前整理好各模块的官方规范总结，放在 `./references/` 目录下：

| 模块 | 规范文件 |
|------|----------|
| 项目结构 | [项目结构规范](./references/project-structure-spec.md) |
| 文件系统 | [文件系统规范](./references/file-system-spec.md) |
| API 调用 | [API 规范](./references/api-spec.md) |
| 组件开发 | [组件规范](./references/component-spec.md) |
| UTS 语法 | [UTS 语法规范](./references/uts-spec.md) |
| UVue 渲染 | [uvue 渲染引擎规范](./references/uvue-spec.md) |

## 开发基本原则

1. **优先使用 uni 跨平台 API**，仅在需要平台特有能力时直接调用原生 API
2. **遵循UTS强类型规范**，所有变量必须声明类型，不支持 undefined，使用 null 替代
3. **静态资源放 static 目录**，只有 static 目录支持变量路径访问
4. **封装原生能力为 uts 插件**，使用 uni_modules 管理，方便复用
5. **兼容开发**：需要同时支持 uni-app 和 uni-app x 时，同时提供 `.vue` 和 `.uvue` 文件

## 官方文档链接

- uni-app X 官方文档：<https://doc.dcloud.net.cn/uni-app-x/>
- uni-app 官网：<https://uniapp.dcloud.net.cn/>
