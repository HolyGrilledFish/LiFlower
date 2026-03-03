# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

锂之花 (LiFlower) - 基于 Vue 3 的角色卡配置应用

## 技术栈

- **框架**: Vue 3 (Composition API + `<script setup>`)
- **构建工具**: Vite 7
- **UI 组件库**: Element Plus
- **路由**: Vue Router 5
- **状态管理**: Pinia 3
- **样式**: SCSS

## 开发命令

```bash
npm run dev      # 启动开发服务器 (http://localhost:3000)
npm run build    # 生产构建
npm run preview  # 预览生产构建
```

## 项目结构

```
src/
├── assets/          # 静态资源
├── components/      # 通用组件
├── data/           # JSON 数据文件 (硬件规格、基本属性、人类起源)
├── router/         # 路由配置
├── stores/         # Pinia 状态管理 (app.js, user.js)
├── views/          # 页面组件
├── App.vue         # 根组件
└── main.js         # 入口文件
```

## 架构说明

- 使用 `@` 路径别名指向 `src/` 目录
- 路由采用 HTML5 History 模式
- 组件使用 Composition API 和 `<script setup>` 语法
- JSON 数据文件可直接通过 `import` 引入

## 数据文件

- `src/data/硬件规格.json` - 人形硬件规格配置 (劳工级/民用级/警用级/军用级/实验级)
- `src/data/基本属性.json` - 基本属性配置
- `src/data/人类起源.json` - 人类起源配置
- `src/data/企业技术.json` - 企业技术配置

## 专有名词

- 人类：Anthropos
- 人形：Anthroform

## 变量命名规范

自 v0.9 起，代码库中的变量名和 JSON 数据字段已全面采用英文命名，以提高代码兼容性和避免潜在的解析问题。

### JSON 数据文件字段名

**硬件规格.json (Hardware Specs):**
| 原中文字段 | 新英文字段 |
|-----------|-----------|
| `名称` | `name` |
| `属性点` | `attributePoints` |
| `属性上限加值` | `attributeLimitBonus` |
| `描述` | `description` |
| `推荐扮演` | `recommendedForRoleplay` |
| `效果描述` | `effectDescription` |
| `效果。属性点数加值` | `effect.attributePointsBonus` |
| `效果。属性上限加值` | `effect.attributeLimitBonus` |

**企业技术.json (Manufacturers):**
| 原中文字段 | 新英文字段 |
|-----------|-----------|
| `中文名` | `nameZh` |
| `英文名` | `nameEn` |
| `描述` | `description` |
| `效果描述` | `effectDescription` |
| `限制` | `restriction` |
| `可生产规格` | `producedSpecs` |
| `效果` | `effect` |

**基本属性.json (Attributes):**
| 原中文字段 | 新英文字段 |
|-----------|-----------|
| `属性系统` | `attributeSystem` |
| `结构` | `structure` |
| `力量` | `strength` |
| `运动` | `athletics` |
| `算力` | `compute` |
| `信息` | `information` |
| `功率` | `power` |
| `描述` | `description` |
| `机制说明` | `mechanism` |
| `效果` | `effects` |

### Vue 组件变量对照

**CharacterCard.vue 变量：**
| 说明 | 变量名 |
|-----|-------|
| 角色对象 | `character` |
| 角色名称 | `character.name` |
| 角色类型 | `character.type` |
| 硬件规格 | `character.hardwareSpec` |
| 生产企业 | `character.manufacturer` |
| 属性点总数 | `character.attributePoints` |
| 属性上限 | `character.attributeLimit` |
| 各属性值 | `character.attributes` |
| 硬件规格列表 | `hardwareSpecList` |
| 生产企业列表 | `manufacturerList` |
| 属性数据 | `attributeData` |
| 当前硬件描述 | `currentHardwareDesc` |
| 当前企业描述 | `currentManufacturerDesc` |

**AttributeAllocator.vue 组件 Props：**
| 说明 | 变量名 |
|-----|-------|
| 属性点总数 | `attributePoints` |
| 属性上限 | `attributeLimit` |
| 当前属性值 | `attributes` |
| 属性数据 | `attributeData` |
| 显示分割线 | `showDivider` |

**AttributeAllocator.vue 内部变量：**
| 说明 | 变量名 |
|-----|-------|
| 属性顺序 | `attributeOrder` |
| 属性名称映射 | `attributeNameMap` |
| 已分配点数 | `allocatedPoints` |
| 剩余点数 | `remainingPoints` |
| 获取当前属性值 | `getCurrentAttributeValue` |
| 获取属性名称 | `getAttributeName` |
| 检查可否增加 | `canIncrease` |
| 检查可否减少 | `canDecrease` |
| 增加属性 | `increaseAttribute` |
| 减少属性 | `decreaseAttribute` |
| 获取属性描述 | `getAttributeDescription` |

### 属性键名映射

属性值使用英文键名，显示时通过 `attributeNameMap` 映射为中文：

```javascript
{
  structure: 0,    // 结构
  strength: 0,     // 力量
  athletics: 0,    // 运动
  compute: 0,      // 算力
  information: 0,  // 信息
  power: 0         // 功率
}
```
