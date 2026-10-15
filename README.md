# 苍穹外卖 · 用户端微信小程序

本仓库是教学项目「苍穹外卖」的**用户端微信小程序**，处于整个教学项目的 **前端（用户侧）** 位置：用户在手机上浏览菜品、加购、下单、支付、查看订单，与后端服务通过 HTTP 接口交互。

> 本项目为学习用途的教学示例，非生产系统。

---

<details>
<summary><h2>前言</h2></summary>

* 本仓库为黑马程序员苍穹外卖用户端微信小程序仓库, 仅供学习参考

### 友情链接

* [黑马哔站教程](https://www.bilibili.com/video/BV1TP411v7v6/?share_source=copy_web&vd_source=043cbf9fed7a1099cc00e49137e0ff99)
* [黑马百度网盘资料](https://pan.baidu.com/s/1MNDzXyVlr3mtmLgBjcPJVw?pwd=6633)
* [前辈的在线讲义网站](https://mouday.github.io/java-cqwm/)
* [个人博客仓库](https://github.com/existed-name/Personal-Blogs)

### 项目仓库总览

* [后端](https://github.com/existed-name/sky-takeout-backend), 已部署
* [管理端 Web 前端](https://github.com/existed-name/sky-takeout-frontend-admin), 已部署, 可通过网址 http://39.105.17.19:80 访问
  - 账号有 `admin`/`mooncake`/`xiaoshuai` 等等, 密码都是 `123456`
* [用户端小程序](https://github.com/existed-name/sky-takeout-miniprogram), 本仓库、通过微信开发者工具编译运行
* [用户端 Web 前端](https://github.com/existed-name/sky-takeout-frontend-user), 已部署, 可通过网址 http://39.105.17.19:8082 访问, 无需密码
* 备注：后端请求路径、请求参数是自己写的，跟教程给的前端接不上，所以这里专门给前端做了同步、再发出来

### 本仓库简要说明

1. 教程发的微信小程序资料是编译产物，可以让 AI 逆向分析出源码，但是我这里偷懒就没还原源码
2. 如果不想弄微信开发者工具、微信登录，可以用 Web 前端做体验
3. 相对教程资料，我的修改内容
    * `common/vendor.js` 里面的请求路径对齐我写的后端
    * `pages` 同步 dto/vo 的处理
    * 修复教程附带的 bug
    * 捕获后端异常、前端提醒用户
    * 店铺状态修改为打烊后，教程的小程序会锁死界面，再改为营业，界面依然锁定，必须重新编译运行；于是在店铺打烊时补充界面刷新按键、用户可以手动刷新
4. 用微信开发者工具运行这个小程序时会有很多 deprecated 警告，不用管
5. 用户前端特性
    * 再来一单会先调用清除购物车接口、再调用再来一单接口
    * 现象：先加购一些东西，点击再来一单，会发现之前加购的东西没了，而后端的再来一单只会新增数据、不会删除数据

### 其他说明

* 除了这里的 `前言`, README 其他部分、[CLAUDE.md](/CLAUDE.md)都是 AI 写的, 可供二次开发参考

</details>



<details>
<summary><h2>⚠ 仓库形态</h2></summary>

**本仓库是 uni-app 的编译产物（编译目标 `mp-weixin`），不是可构建的源码工程。**

- 仓库内**没有** `package.json`、`src/`、`manifest.json`、`pages.json`、`App.vue`，也**没有** `node_modules/`。
  - 注意：目录里有一个**连字符命名**的 `node-modules/`，那是**组件库副本**，**不是**依赖目录。
- 因此**没有 `npm install` / `npm run build` 之类的构建步骤**，也没有任何命令行测试。原始的 uni-app 源工程不在此仓库中，仓库内无法二次编译。
- **部分页面的逻辑被 webpack 抽进了 `common/vendor.js`**：
  - `pages/index/index.js`、`pages/order/index.js`、`pages/details/index.js` 这几个文件里**只有页面外壳**（`createPage` 调用、模板渲染、样式），页面脚本（`data` / `methods` / 事件处理）**不在这些 `.js` 里**，而被编译进了 `common/vendor.js`。
  - 其余页面的脚本仍在各自的 `pages/**/*.js` 里。
- **改代码时的对应关系**：
  - 改页面结构 / 样式 / 配置 → 改对应页面的 `.wxml` / `.wxss` / `.json`。
  - 改页面逻辑 → 多数页面改自己的 `.js`；但首页 / 订单页 / 详情页的逻辑在 `common/vendor.js`。
  - 公共逻辑（请求层、接口、状态、工具、公共组件）→ `common/vendor.js`。
- `common/**` 是 webpack 编译产物（`runtime.js` / `vendor.js` / `main.js` / `main.wxss`），其中 `vendor.js` 体量很大，且内含打包机上的绝对路径等编译产物固有内容，属正常现象。

</details>



<details>
<summary><h2>需要改的地方</h2></summary>

### 1. appid（必改）

`project.config.json` 的 `appid` 字段里带的是一个**真实的 appid**（教程资料附带）。**使用者请换成自己小程序的 appid**，否则无法正常预览 / 真机调试。

### 2. 后端地址

小程序里的后端地址是直连的绝对地址，写在源工程的 `utils/env.js` 中，已被编译进 `common/vendor.js`（默认指向本机 `http://localhost:8080`）。

- 由于本仓库是编译产物、无构建脚本，**切换环境需要直接修改 `common/vendor.js` 中对应的地址常量**（`vendor.js` 内另留有一行被注释掉的备用地址，供切换环境时启用）。
- 修改时请**精确到行**，不要重排或整段重写该文件。

</details>



<details>
<summary><h2>目录结构</h2></summary>

| 目录 / 文件 | 说明 |
|---|---|
| `app.js` / `app.json` / `app.wxss` | 小程序入口、页面注册与全局样式 |
| `pages/` | 各页面（每个页面一套 `.js` / `.json` / `.wxml` / `.wxss`），以及页面级公共组件（`pages/common/`） |
| `components/` | 预编译好的公共组件（如 `uni-popup`、`uni-icons`、`uni-nav-bar`、`uni-phone` 等） |
| `uni_modules/` | 预编译好的第三方 uni-* 组件模块 |
| `common/` | **webpack 编译产物**：`runtime.js`（运行时）、`vendor.js`（最大的共享 chunk，含 Vue 运行时、uni 框架、状态、请求层、接口、工具、公共组件等）、`main.js`（全局入口）、`main.wxss`（全局样式） |
| `static/` | 图片等静态资源 |
| `node-modules/` | 组件库副本（**不是** npm 依赖目录） |
| `sitemap.json` | 小程序搜索索引配置 |
| `project.config.json` | 项目配置（含 appid、编译设置等） |

</details>



<details>
<summary><h2>已知限制</h2></summary>

- **`urlCheck: false`**：`project.config.json` 中关闭了域名校验，这是**开发期**的便利设置。
- **正式发布要求**：上线前后端需以 **HTTPS + 已备案域名**对外，并在**微信公众平台配置 request 合法域名**，否则真机 / 正式版请求会被拦截。上线路径为：用微信开发者工具打开本目录 → 上传 → 微信公众平台提交审核发布。
- **没有构建脚本，也没有命令行测试**：本仓库不存在「一条命令就能跑 / 验」的方式，端到端验证只能人工在微信开发者工具里手动操作。
- **本仓库只负责小程序前端**：不改后端，也不改管理端前端、Web 用户端前端（均为各自独立仓库）。

</details>



<details>
<summary><h2>开源许可证</h2></summary>

本项目采用 **GNU General Public License v3.0（GPL-3.0）** 许可，详见 [LICENSE](LICENSE)。

</details>



