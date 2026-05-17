# Fude Ramen · 会员系统 App

React 19 + Vite 6 + Tailwind CSS v4

包含:
- 顾客端: 注册/登录、菜单浏览、购物车、打包外带/外送、Stripe 充值、积分明细、幸运转盘、订单查询
- 员工端: 订单管理、顾客 CRUD、积分/余额操作、优惠券核销
- 117 道真实菜品(含过敏原信息)
- 2 家店铺(Schio + San Bonifacio)
- 配送距离实时计算(Google Maps Places API)

## 🚀 快速开始

需要 **Node.js 18+**。

```bash
# 1. 安装依赖
npm install

# 2. 启动开发服务器 (热重载)
npm run dev
```

打开浏览器访问 http://localhost:5173

手机调试: 局域网内访问 `http://你电脑IP:5173`

## 📱 在手机上"安装" (PWA)

本 app 是 PWA(Progressive Web App), 不用 App Store 也能像原生 app 一样安装到手机。

### 步骤:

**1. 先部署到网上** (二选一)
   - Vercel: 拖拽整个项目到 https://vercel.com/new → 5 分钟有 URL
   - 或者运行 `npm run build` → 把 `dist/` 拖到 https://app.netlify.com/drop

**2. 用手机打开你的 URL** (例如 https://fude-ramen.vercel.app)

**3. 添加到主屏幕**

#### iPhone (Safari)
- 打开网址
- 点底部"分享"按钮 (向上箭头方框)
- 滑动找到 "**添加到主屏幕**"
- 点"添加"
- 桌面就出现 Fude Ramen 图标,**全屏运行无浏览器栏**

#### Android (Chrome)
- 打开网址
- 通常会自动弹出 "**安装应用**" 横幅 → 点"安装"
- 或者点右上角三个点菜单 → "**添加到主屏幕**"
- 自动添加到桌面 + 应用抽屉
- 跟其它 app 一样可以从最近任务切换

### 功能:
- ✅ 全屏运行(没有地址栏)
- ✅ 自己的图标(出现在桌面/应用列表)
- ✅ 启动时显示品牌色背景(不闪白)
- ✅ Service Worker 缓存,**离线也能打开** (第二次访问后)
- ✅ 菜品图片首次访问后会缓存,后续秒开

### 真上 App Store?

如果以后想正式上架 iOS App Store / Google Play:
- 用 **Capacitor** 把 PWA 打包成 native 壳子
- 申请开发者账号(Apple $99/年, Google $25 一次性)
- 提交审核约 1-2 周

但 PWA 已经覆盖 99% 用户需求,先用 PWA 跑半年再考虑要不要上架。

## 项目结构

```
fude-ramen-app/
├── index.html
├── package.json
├── vite.config.js
├── src/
│   ├── main.jsx           # 入口
│   ├── App.jsx            # 整个 app (~3500 行)
│   ├── index.css          # Tailwind + 全局样式
│   └── storage-shim.js    # window.storage → localStorage 适配
└── README.md
```

## 菜品照片

有两种方式上传/管理菜品照片:

### 方式 1: 后台员工上传(最简单)

1. 进入 app → 员工端 (PIN: 1234)
2. 点 **"菜品照片管理"**
3. 选择类别 → 点击菜品 → 拍照 / 从相册选
4. 照片自动压缩到 1024px / JPEG 85%(~150KB)
5. 存到浏览器 localStorage,所有用户都能看到

**注意**: localStorage 只存当前设备。要跨设备同步,需要接后端,把照片存数据库或 S3。

### 方式 2: 文件批量上传

1. 把照片放到 `public/menu/` 目录
2. 命名规则: `{菜品id}.jpg`(支持 jpg/jpeg/png/webp/avif)
   - 例如: `ramen-1.jpg`、`uramaki-3.webp`
   - 多格式时优先级: webp > jpg > jpeg > png > avif
3. 运行 `npm run sync-images` 生成 `src/menu-images.json`
4. app 启动时自动加载

**优先级**: 方式 1 (后台上传) > 方式 2 (文件) > 默认 emoji

如果一个菜品同时有后台上传和文件,后台上传的优先显示。删除后台上传 → 自动用文件 → 再删 → 显示 emoji。



目前使用浏览器 `localStorage` 保存:
- `restaurant-loyalty-data` - 所有顾客信息
- `restaurant-orders-data` - 所有订单
- `restaurant-location` - 当前选择的店铺

**注意**: localStorage 只在本地浏览器,不同设备数据不互通。要上线必须接后端,见下方。

## 接后端

`server/` 目录里有 Express + SQLite + Stripe 完整后端。要切换:

1. 编辑 `src/storage-shim.js`,把 localStorage 调用替换成 fetch 到 `server.js` 的 API
2. 或者重写 App.jsx 里的 `saveCustomers` / `saveOrders` 改成调用 `/api/customers` 等

## Google Maps API Key

`App.jsx` 顶部的 `GMAPS_KEY` 常量目前是公开值,生产环境务必:

1. 去 [Google Cloud Console](https://console.cloud.google.com/apis/credentials) 创建新 key
2. **限制 HTTP referrer**: `https://你的域名.com/*`
3. **限制 API**: 只勾选 `Places API (New)` + `Geocoding API`
4. 启用每日配额上限,防止账单被刷爆
5. 改成 env 变量: 
   ```js
   const GMAPS_KEY = import.meta.env.VITE_GMAPS_KEY;
   ```
   然后建 `.env.local`:
   ```
   VITE_GMAPS_KEY=你的key
   ```

## 部署到生产

### Vercel (最简单, 免费)

```bash
npm i -g vercel
vercel
```

按提示选择即可。每次 push 自动部署。

### Netlify

拖动 `dist/` 文件夹到 https://app.netlify.com/drop

### 自己服务器

```bash
npm run build
# 把 dist/ 内容上传到任意静态托管 (nginx, apache, S3 等)
```

## 移动端调试

`npm run dev` 启动后, 在手机浏览器访问 `http://你电脑IP:5173`。

iOS Safari: Settings → Safari → Advanced → Web Inspector 打开, Mac 上用 Safari → Develop 连接。

Android Chrome: 手机连 USB → Chrome `chrome://inspect`。

## 常见问题

**Q: 地址自动补全没出现?**
A: 浏览器 console (F12) 看错误。如果是 403/CORS, 检查 Places API (New) 是否在 Google Cloud Console 启用、key 的 referrer 限制是否包含 localhost。

**Q: 重置 localStorage?**
A: 浏览器 DevTools → Application → Storage → Clear site data

**Q: 修改菜单?**
A: 编辑 `src/App.jsx` 顶部的 `MENU` 常量

**Q: 修改店铺信息?**
A: 编辑 `src/App.jsx` 顶部的 `STORES` 常量

**Q: 修改员工 PIN?**
A: 编辑 `src/App.jsx` 顶部的 `EMPLOYEE_PIN` 常量(目前是 `1234`)
