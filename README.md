# XSS 注入闯关靶场

一个自包含的 Web 安全交互式教学靶场：把 XSS 与 SQL 注入的成因、过滤缺陷与绕过手法拆成 8 个梯度关卡，学员在浏览器里直接提交 payload，在模拟页面成功触发弹窗即判定通关。**全程本地沙箱执行，无需后端、无需搭环境。**

- 在线体验（GitHub Pages）：https://jiangshang-star.github.io/xss-lab/
- 原投稿版本：https://workbuddy.link/p/2rOYkGOV0K5RtmHxhDz2jS
- 项目性质：个人项目 · WorkBuddy「1024 造物挑战」参赛作品

## 技术栈 / 开发语言

| 项 | 说明 |
|---|---|
| **开发语言** | **HTML5 + 原生 JavaScript（ES6+）+ CSS3** |
| 形态 | 单个 HTML 文件，零依赖、零构建、无框架 |
| 运行时 | 纯浏览器前端；无后端、无数据库、无网络请求 |
| 隔离机制 | `<iframe sandbox="allow-scripts allow-modals" srcdoc>` + `postMessage` |
| 状态持久化 | `localStorage` |
| 测试 | Node 桩环境 + Headless Chromium（Playwright）自动化回归 |

## 快速开始

```bash
# 方式一：直接双击 index.html
# 方式二：起个静态服务（可选）
python -m http.server 8000
# 浏览器打开 http://localhost:8000
```

无需 `npm install`，无需任何配置。

## 关卡设计

| 关 | 类型 | 考察点 |
|---|---|---|
| 1 | XSS · 反射型 | 输入未转义直接拼进 HTML |
| 2 | XSS · 属性出逃 | 闭合属性后注入事件处理器 |
| 3 | XSS · 黑名单绕过 | 绕过 `script` 关键词过滤，改用可执行标签/事件 |
| 4 | XSS · script 块逃逸 | 闭合已有脚本块重新构造代码 |
| 5 | XSS · 存储型 | 留言板存储 + 审核机器人定时巡访渲染触发 |
| 6 | SQLi · 登录绕过 | 布尔恒真条件绕过鉴权 |
| 7 | SQLi · UNION 偷旗 | 自研 mock SQL 引擎下的列数探测与联合查询 |
| 8 | WAF Boss | 综合过滤规则下的组合绕过 |

提示按需领取，3 次内通关记为「完美」；全部通关后生成「注入终结者」结业认证卡。

## 实现要点

**让 payload 真的执行，但不让它逃出靶场。** 这是本项目最主要的设计约束：

- 学员提交的 payload 被写入 `srcdoc`，放进 `sandbox="allow-scripts allow-modals"` 的 iframe 中执行。该沙箱没有 `allow-same-origin`，因此 iframe 内的脚本无法读取主页面 DOM、`localStorage` 或 Cookie。
- 在沙箱内 Hook `alert` / `prompt` / `confirm`，触发后通过 `postMessage` 把通关事件回传主页面判定，主页面只接受约定格式的消息。
- 主页面自身所有用户输入统一经 `esc()` 转义后再渲染，保证靶场本体不被注入。
- 创建 iframe 时先设好 `srcdoc` 与 `sandbox` 再 `appendChild`，规避二次导航竞态导致的偶发不执行（修复后实测 7/7 稳定触发）。

**SQL 注入不需要数据库。** 两关 SQLi 由自研的前端 mock SQL 解析引擎模拟：对拼接后的语句做词法解析与条件求值，判定登录是否绕过、UNION 列数是否匹配。曾修复过 `'1'='1'` 左侧字符串字面量解析错误。

**质量保障。** 用 Headless Chromium 驱动父页 iframe、`--dump-dom` 做逐关回归，验证各关 payload 的触发率与关卡分支分发正确性。

## 安全说明

- 靶场内所有域名（`*.lab`）均为虚构，不发起任何真实网络请求。
- payload 仅在执行受限的沙箱 iframe 内运行，不接触主页面、不接触本机文件系统、不回传任何数据。
- 项目用途为安全教学与防御意识培训，不含针对真实系统的攻击能力。

## 目录结构

```
xss-lab/
├── index.html    # 全部 HTML / CSS / JavaScript（单文件应用）
├── README.md
└── .gitignore
```
