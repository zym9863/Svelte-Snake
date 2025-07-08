[English](README_EN.md) | [中文](README.md)

# Svelte Snake Game

本项目是一个基于 Svelte + TypeScript + Vite 的经典贪吃蛇游戏示例，支持键盘操作、分数统计和速度递增。

## 运行方式

```bash
pnpm install
pnpm dev
```

## 主要特性

- 经典贪吃蛇玩法，支持方向键和 WASD 控制
- 吃到红色食物蛇会变长并加分，速度逐渐加快
- 撞墙或撞到自己游戏结束，可重新开始
- 分数实时显示
- 响应式 Canvas 渲染，界面美观
- 支持暂停/继续（空格键或按钮）

## 目录结构

```
├── public/           # 静态资源
├── src/
│   ├── app.css       # 全局样式
│   ├── App.svelte    # 应用入口，挂载 Snake 组件
│   ├── main.ts       # 启动入口
│   └── lib/
│       ├── Snake.svelte   # 贪吃蛇游戏主逻辑和界面
│       └── Counter.svelte # 计数器示例（未在主界面使用）
├── index.html        # HTML 模板
├── package.json      # 项目依赖与脚本
└── ...
```

## 游戏操作说明

- 方向键或 WASD 控制蛇移动
- 空格键开始/暂停游戏
- 吃到红色方块得分并变长
- 撞墙或撞到自己游戏结束

## 其它说明

- `Counter.svelte` 为 Svelte 组件计数器示例，未集成到主页面，仅供学习参考。
- 如需扩展功能（如排行榜、皮肤等），可在 `Snake.svelte` 基础上继续开发。

---

本项目适合 Svelte 新手和小游戏开发爱好者学习参考。
