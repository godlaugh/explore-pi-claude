# π 圆周率求解演示

<p align="center">
  <img src="https://img.shields.io/badge/Vue-3.4-4FC08D?style=flat-square&logo=vue.js" alt="Vue 3.4">
  <img src="https://img.shields.io/badge/Vite-5.0-646CFF?style=flat-square&logo=vite" alt="Vite 5.0">
  <img src="https://img.shields.io/badge/License-MIT-blue?style=flat-square" alt="MIT License">
</p>

<p align="center">
  <b>探索数学之美，见证无理数的奥秘</b>
</p>

一个使用 Vue 3 构建的交互式圆周率计算演示项目，通过可视化方式展示四种经典的 π 计算算法，兼具教育意义与视觉美感。

## ✨ 功能特性

- 🎲 **蒙特卡洛法** - 随机投点，以概率估算 π
- ∑ **莱布尼茨级数** - 无穷级数逼近 π/4
- ⬡ **阿基米德割圆术** - 正多边形逼近圆周
- 📍 **蒲丰投针实验** - 经典概率实验求 π
- 📊 **实时可视化** - Canvas 动画实时展示计算过程
- 🎛️ **交互控制** - 可调节速度、参数，支持暂停/重置
- 📈 **误差分析** - 实时显示估算值、真实值及误差百分比

## 🛠️ 技术栈

| 技术 | 版本 | 说明 |
|------|------|------|
| [Vue 3](https://vuejs.org/) | ^3.4.0 | 渐进式 JavaScript 框架 |
| [Vite](https://vitejs.dev/) | ^5.0.0 | 下一代前端构建工具 |
| Canvas API | - | 2D 图形渲染 |

## 📁 项目结构

```
explore-pi-claude/
├── index.html              # 入口 HTML
├── package.json            # 项目配置
├── vite.config.js          # Vite 配置
├── bun.lock                # 依赖锁文件
└── src/
    ├── main.js             # 应用入口
    ├── App.vue             # 根组件（导航 + 布局）
    ├── components/         # 算法组件
    │   ├── MonteCarlo.vue       # 蒙特卡洛法
    │   ├── LeibnizSeries.vue    # 莱布尼茨级数
    │   ├── ArchimedesMethod.vue # 阿基米德割圆术
    │   └── BuffonNeedle.vue     # 蒲丰投针实验
    └── styles/
        └── index.css       # 全局样式
```

## 🚀 快速开始

### 环境要求

- Node.js >= 18.0.0
- 包管理器: npm / yarn / pnpm / bun

### 安装依赖

```bash
# 使用 npm
npm install

# 使用 yarn
yarn

# 使用 pnpm
pnpm install

# 使用 bun (推荐，速度更快)
bun install
```

### 启动开发服务器

```bash
# 使用 npm
npm run dev

# 使用 bun
bun run dev
```

访问 http://localhost:5173 查看应用。

### 构建生产版本

```bash
# 使用 npm
npm run build

# 使用 yarn
yarn build

# 使用 pnpm
pnpm build

# 使用 bun
bun run build
```

构建产物将输出到 `dist/` 目录。

### 预览生产构建

```bash
# 使用 npm
npm run preview

# 使用 bun
bun run preview
```

## 📐 算法说明

### 1. 蒙特卡洛法 (Monte Carlo Method)

在边长为 2r 的正方形内随机撒点，统计落在内切圆（半径 r）中的点数。

**原理：**
- 正方形面积: `(2r)² = 4r²`
- 内切圆面积: `πr²`
- 面积比: `πr² / 4r² = π/4`

**公式：**
```
π ≈ 4 × (圆内点数 / 总点数)
```

### 2. 莱布尼茨级数 (Leibniz Series)

使用无穷级数逼近 π/4，由德国数学家莱布尼茨发现。

**公式：**
```
π/4 = 1 - 1/3 + 1/5 - 1/7 + 1/9 - 1/11 + ...
    = Σ(n=0 to ∞) (-1)ⁿ / (2n+1)
```

> ⚠️ 该级数收敛较慢，需要大量项数才能获得较高精度。

### 3. 阿基米德割圆术 (Archimedes' Method)

用正多边形的周长逼近圆的周长，这是人类历史上最早的 π 计算方法之一。

**原理：**
- 内切正 n 边形周长 < 圆周长 < 外切正 n 边形周长
- 当 n → ∞ 时，多边形周长 → 圆周长

**公式：**
```
内切周长 = 2nr × sin(π/n)
外切周长 = 2nr × tan(π/n)
π ≈ (内切周长 + 外切周长) / (4r)
```

### 4. 蒲丰投针实验 (Buffon's Needle)

18 世纪法国数学家蒲丰提出的概率实验。

**实验设置：**
- 平行线间距: d
- 针的长度: l (l ≤ d)

**原理：**
针与平行线相交的概率为 `P = 2l / (πd)`

**公式：**
```
π = (2 × l × n) / (d × c)
```
其中 n 为投掷总次数，c 为相交次数。

## 🎨 设计风格

项目采用中式淡雅配色方案：

| 颜色 | 色值 | 用途 |
|------|------|------|
| 青灰 | `#5B7A8C` | 主色调、强调色 |
| 米白 | `#F8F7F4` | 背景色 |
| 草绿 | `#7A8B7A` | 成功状态（圆内/相交） |
| 棕灰 | `#A89888` | 次要状态（圆外/未相交） |

字体使用 [Noto Serif SC](https://fonts.google.com/noto/specimen/Noto+Serif+SC)（思源宋体）。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

## 📝 可能的改进方向

- [ ] 添加更多 π 计算算法（如 Machin 公式、Chudnovsky 算法）
- [ ] 支持导出计算结果
- [ ] 添加算法收敛速度对比图表
- [ ] 支持深色模式
- [ ] 添加移动端适配优化
- [ ] 国际化支持 (i18n)

## 📄 许可证

本项目基于 [MIT 许可证](LICENSE) 开源。

---

<p align="center">
  Made with ❤️ and Mathematics
</p>
