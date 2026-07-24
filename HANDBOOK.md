# 🏃 Running_page 部署完全手册

> 从咕咚导出跑步数据 → 搭建个人跑步主页 → 部署到 GitHub Pages

---

## 目录

1. [项目介绍](#1-项目介绍)
2. [环境搭建](#2-环境搭建)
3. [Fork 开源项目](#3-fork-开源项目)
4. [导出咕咚数据](#4-导出咕咚数据)
5. [数据分析 & 构建](#5-数据分析--构建)
6. [配置你的主页](#6-配置你的主页)
7. [国内地图适配（重要）](#7-国内地图适配重要)
8. [构建前端](#8-构建前端)
9. [部署到 GitHub Pages](#9-部署到-github-pages)
10. [日常更新数据](#10-日常更新数据)
11. [附录：踩坑记录](#11-附录踩坑记录)

---

## 1. 项目介绍

### 1.1 什么是 Running_page

[Running_page](https://github.com/yihong0618/running_page) 是一个开源的个人跑步主页生成器，作者 [yihong0618](https://github.com/yihong0618)。

它能做什么：
- ✅ 从各大运动平台（咕咚、Garmin、Strava 等）导出跑步数据
- ✅ 生成美观的个人跑步主页，包含地图、热力图、年度统计等
- ✅ 部署到 GitHub Pages 免费托管

### 1.2 技术栈

| 技术 | 用途 |
|------|------|
| **Python 3.10+** | 数据导出、分析脚本 |
| **React 19 + TypeScript** | 前端页面 |
| **Vite** | 前端构建工具 |
| **Mapbox GL JS** | 地图渲染（可更换为高德瓦片） |
| **Leaflet** | 备选地图库 |
| **Chart.js / Recharts** | 图表渲染 |
| **GitHub Pages** | 免费静态托管 |

### 1.3 数据流

```
咕咚 App
  ↓ (模拟 App 登录)
codoon_sync.py ───→ GPX_OUT/*.gpx（原始轨迹文件）
  ↓
build_data.py ────→ src/static/activities.json（前端数据）
  ↓
npm run build ────→ dist/（静态页面）
  ↓
git push ────────→ GitHub Pages（上线）
```

---

## 2. 环境搭建

### 2.1 安装 Python

```bash
# 检查版本
python3 --version
# 需要 ≥ 3.10，如果版本太低请升级
```

**Ubuntu/Debian：**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

**macOS：**
```bash
brew install python
```

**Windows：** 从 https://python.org 下载安装包

### 2.2 安装 Node.js

```bash
# 需要 Node.js 18+
node --version
npm --version
```

**推荐使用 nvm 安装：**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
nvm install 20
```

### 2.3 安装 Git

```bash
git --version
# 如果没有：sudo apt install git 或 https://git-scm.com
```

### 2.4 注册 GitHub

1. 打开 https://github.com
2. 点 **Sign up**，用邮箱注册
3. 记住你的 GitHub 用户名（比如 `yourname`）

---

## 3. Fork 开源项目

### 3.1 选择上游项目

Running_page 是一个开源项目，地址是：
```
https://github.com/yihong0618/running_page
```

它支持多个运动平台（咕咚、Garmin、Strava、Keep 等），我们这里用**咕咚（Codoon）**。

### 3.2 Fork 到自己账号

1. 打开 https://github.com/yihong0618/running_page
2. 点右上角 **Fork** 按钮
3. 在弹出窗口选择你自己的 GitHub 账号
4. 等待 fork 完成

Fork 完成后你就得到了自己的仓库：
```
https://github.com/你的用户名/running_page
```

### 3.3 克隆到本地

```bash
git clone https://github.com/你的用户名/running_page.git
cd running_page
```

**国内网络慢的话，设置代理：**
```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

### 3.4 安装 Python 依赖

```bash
pip install -r requirements.txt
```

**国内加速（清华镜像）：**
```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**主要依赖说明：**

| 包名 | 用途 |
|------|------|
| `requests` | HTTP 请求（调用咕咚 API） |
| `gpxpy` | 解析 GPX 轨迹文件 |
| `polyline` | 编码/解码 Google Polyline 格式 |
| `python-dateutil` | 日期时间处理 |

### 3.5 安装前端依赖

```bash
npm install
```

或使用 pnpm（更快）：

```bash
npm install -g pnpm
pnpm install
```

---

## 4. 导出咕咚数据

### 4.1 数据导出原理

咕咚 App 的数据可以通过其 API 导出。Running_page 项目中的 `codoon_sync.py` 脚本做了这件事。

**工作原理：**
1. 模拟咕咚手机 App 向 API 发送登录请求
2. 登录成功后获取 access_token
3. 用 token 请求用户的跑步记录列表
4. 逐条下载每条跑步的 GPX 轨迹文件
5. 保存到 `GPX_OUT/` 目录

### 4.2 找到导出脚本

脚本位置在：
```
run_page/codoon_sync.py
```

### 4.3 运行导出

```bash
cd run_page
python3 codoon_sync.py --phone 138xxxxxxx --password 你的咕咚密码
```

**脚本执行过程（实际输出）：**
```
👟 咕咚数据同步工具
📱 正在登录...
✅ 登录成功
📥 正在获取跑步列表...
📦 共获取 505 条跑步记录
⬇️  下载 GPX: 283544320.gpx ✓
⬇️  下载 GPX: 283882669.gpx ✓
⬇️  下载 GPX: 284072264.gpx ✓
...
✅ 全部完成！共导出 505 个 GPX 文件
```

### 4.4 导出完成后的文件

```
running_page/
├── GPX_OUT/           ← 所有 GPX 轨迹文件（505 个）
│   ├── 283544320.gpx
│   ├── 283882669.gpx
│   └── ...
├── run_page/
│   ├── data.db        ← SQLite 数据库
│   └── codoon_sync.py ← 导出脚本
```

### 4.5 常见问题

**Q: 登录失败 / Invalid credentials？**
- 检查手机号和密码是否正确
- 如果改过密码，重新运行脚本
- 咕咚 API 可能要求验证码，关注终端提示

**Q: 只导出了一部分？**
- 脚本是增量同步的，重复运行只下载新增的
- 删掉 `data.db` 会重新全部下载

**Q: 导出速度慢？**
- 网络原因，正常现象
- 505 条记录大约需要 2-5 分钟

---

## 5. 数据分析 & 构建

### 5.1 GPX 文件解析

GPX（GPS Exchange Format）是一种 XML 格式的 GPS 轨迹文件。每个 `.gpx` 文件包含一次跑步的完整轨迹，格式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<gpx version="1.1">
  <trk>
    <trkseg>
      <trkpt lat="30.7565" lon="103.9263">
        <ele>500.0</ele>
        <time>2024-01-07T09:24:29Z</time>
      </trkpt>
      <trkpt lat="30.7565" lon="103.9265">
        <ele>501.2</ele>
        <time>2024-01-07T09:24:32Z</time>
      </trkpt>
      <!-- 更多轨迹点... -->
    </trkseg>
  </trk>
</gpx>
```

### 5.2 运行数据构建脚本

```bash
cd /path/to/running_page
python3 build_data.py
```

**这个脚本做了什么事：**

| 步骤 | 说明 |
|------|------|
| 读取 GPX | 遍历 `GPX_OUT/` 下所有 `.gpx` 文件 |
| 解析轨迹 | 用 `gpxpy` 提取坐标点、时间、海拔 |
| 过滤无效 | 跳过距离 <500m、>60km、速度异常的记录 |
| 计算统计 | 距离、时长、配速、爬升 |
| 编码轨迹 | 把坐标点编码为 Google Polyline 格式 |
| 生成 JSON | 输出到 `src/static/activities.json` |
| 计算连续 | 统计连续跑步天数 |

**脚本执行输出：**
```
✅ JSON: 492 activities -> src/static/activities.json
   Date range: 2018-01-07 ~ 2026-07-14
   Total distance: 6353.1 km
```

### 5.3 数据分析脚本

项目还提供了分析脚本（由我们自行编写），可以看统计数据：

```bash
python3 analyze_runs.py
```

**输出示例：**
```
📊 跑步统计报告
━━━━━━━━━━━━━━━━━━
总次数: 492
总距离: 6,353 km
总时间: 620 小时
平均距离: 12.9 km
平均配速: 5:51 /km
最佳单次: 44.4 km (2024-11-08)
巅峰年份: 2024年 (1,500 km)
最爱星期: 周六 (97次)
最舒适区间: 10-15 km (164次)
```

### 5.4 生成 SVG 海报（可选）

```bash
python3 run_page/gen_svg.py --from-db --type github --output assets/github.svg
```

这会生成 GitHub 风格的跑步热力图 SVG。

---

## 6. 配置你的主页

### 6.1 个人信息

编辑 `src/static/site-metadata.ts`：

```typescript
export const siteMetadata = {
  title: '🏃 我的咕咚跑步主页',      // 页面标题
  description: '492次 / 6353km 跑步记录',  // 描述
  author: '你的名字',               // 作者
  siteUrl: 'https://你的用户名.github.io/running_page/',  // 主页地址
  avatar: 'https://...',            // 头像 URL（可选）
  navLinks: [                       // 导航链接
    { name: 'GitHub', url: 'https://github.com/你的用户名' },
    { name: '博客', url: 'https://你的博客地址' },
  ],
};
```

### 6.2 主题配置

编辑 `src/themes/classic/utils/const.ts`：

```typescript
// 基本设置
const IS_CHINESE = true;           // 中文用户
const USE_DASH_LINE = true;        // 路线用虚线
const LINE_OPACITY = 0.4;          // 路线透明度
const MAP_HEIGHT = window.innerWidth <= 768 ? 250 : 600;  // 地图高度
const PRIVACY_MODE = false;        // 隐私模式（隐藏地图只留路线）
const LIGHTS_ON = true;            // 默认亮灯（显示地图街道）
const ROAD_LABEL_DISPLAY = true;   // 显示道路标签

// 颜色主题
const nike = 'rgb(224,237,94)';    // 主色调（可以改成你喜欢的颜色）
export const MAIN_COLOR = nike;
```

### 6.3 添加跑步数据总结（可选）

编辑 `src/themes/classic/components/YearsStat/index.tsx`，

在 `<p className="leading-relaxed">` 下方添加：

```tsx
<div className="mt-2 rounded-lg bg-white/5 p-3 text-xs leading-relaxed text-gray-400">
  🏃 8年跑过492次 · 6,353km · 620小时<br />
  平均12.9km/次 · 配速5:51/km<br />
  巅峰2024年1,500km · 单次最远44.4km<br />
  最爱周六 (97次) · 最舒适10-15km<br />
  从2018跑到2026，用脚步丈量城市，用汗水标记时光。
</div>
```

---

## 7. 国内地图适配（重要）

### 7.1 为什么需要适配

Running_page 默认使用 **Mapbox GL JS** 渲染地图。但 Mapbox 的所有域名在国内都被封锁：

| 域名 | 用途 | 国内状态 |
|------|------|----------|
| `api.mapbox.com` | 样式/鉴权 | ❌ 被墙 |
| `events.mapbox.com` | 遥测上报 | ❌ 被墙 |
| `*.tiles.mapbox.com` | 地图瓦片 | ❌ 被墙 |

如果不做适配，地图会完全空白，控制台报 `ERR_ADDRESS_INVALID`。

### 7.2 更换瓦片源为高德

**步骤一：** 修改 `src/themes/classic/utils/const.ts`

```typescript
// 把 MAP_TILE_VENDOR 改成 'gaode'
export const MAP_TILE_VENDOR = 'gaode';
```

**步骤二：** 在高德瓦片供应商配置中，使用内联 Mapbox GL Style JSON：

```typescript
gaode: {
  'osm-bright': {
    version: 8,
    name: 'Gaode Light',
    sources: {
      'gaode-raster': {
        type: 'raster',
        tiles: [
          'https://webrd01.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}'
        ],
        tileSize: 256,
        attribution: '© 高德地图'
      }
    },
    layers: [{
      id: 'gaode-raster-layer',
      type: 'raster',
      source: 'gaode-raster',
      minzoom: 0,
      maxzoom: 18
    }]
  },
  'dark-matter': {
    // 同上，黑色主题版本
    ...
  }
},
```

**注意：** style 对象的键名必须是 `'osm-bright'` 和 `'dark-matter'`，不能是 `'light'` / `'dark'`，否则会报 "Style not valid" 错误。

### 7.3 修复 MapboxLanguage 插件冲突

在 `RunMap/index.tsx` 中，MapboxLanguage 插件只能工作在 Mapbox 矢量瓦片上，需要加守卫：

```typescript
// 修改前（会崩溃）：
if (map && IS_CHINESE) {
  map.addControl(new MapboxLanguage({ defaultLanguage: 'zh-Hans' }));
}

// 修改后（只在 Mapbox 供应商时启用）：
if (map && IS_CHINESE && (MAP_TILE_VENDOR === 'mapbox' || MAP_TILE_VENDOR === 'mapcn')) {
  map.addControl(new MapboxLanguage({ defaultLanguage: 'zh-Hans' }));
}
```

### 7.4 裁剪遥测上报

Mapbox GL JS v3 会向 `events.mapbox.com` 发遥测包，这个域名在国内被墙会导致地图崩溃。需要在 `node_modules` 中打补丁：

```bash
# 在构建前执行
cd node_modules/mapbox-gl/dist
python3 -c "
with open('mapbox-gl.js', 'rb') as f:
    data = f.read()
data = data.replace(b'events.mapbox.com', b'0-events.local')
data = data.replace(b'events.mapbox.cn', b'0-events.local')
with open('mapbox-gl.js', 'wb') as f:
    f.write(data)
"
```

**注意：** 替换一定要精确。原始的 URL 由多段拼接：`hostname + '/events/v2'`，如果替换不完整会导致 `0.0.0.0/events/events/v2` 双路径错误。

### 7.5 开启默认亮灯

地图默认是暗的（需要点 "Turn on the Light"），修改 `const.ts`：

```typescript
const LIGHTS_ON = true;  // 改为 true
```

---

## 8. 构建前端

### 8.1 构建命令

```bash
# 关键！如果部署到 GitHub Pages 的子路径，必须设置 PATH_PREFIX
PATH_PREFIX=/running_page npm run build
```

**`PATH_PREFIX` 的作用：**

`vite.config.ts` 中：
```typescript
base: process.env.PATH_PREFIX ? `${process.env.PATH_PREFIX}/` : '/',
```

如果部署在 `https://你的用户名.github.io/running_page/`，base 必须是 `/running_page/`。如果不设置，资源路径会变成 `/assets/xxx.js`（去根路径找），导致 404。

### 8.2 构建产物

构建完成后，产物在 `dist/` 目录：

```
dist/
├── index.html           ← 入口页面
├── 404.html             ← 404 页面
├── assets/
│   ├── index-xxxx.js    ← 主 JS 入口
│   ├── classic-xxxx.js  ← 经典主题组件
│   ├── mapbox-gl-xxx.js ← 地图库
│   ├── activities-xxx.json  ← 你的跑步数据
│   ├── const-xxx.js     ← 配置常量
│   └── ...              ← 其他组件和资源
├── images/
│   └── favicon.png      ← 网站图标
```

---

## 9. 部署到 GitHub Pages

### 9.1 方法一：GitHub Actions 自动部署（推荐）

在项目根目录创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [master, main]

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: PATH_PREFIX=/running_page npm run build
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

推送到 GitHub 后自动部署。**注意：** GitHub Actions 在美国节点运行，如果数据需要从咕咚同步，需要另外处理。

### 9.2 方法二：手动部署

```bash
# 1. 构建
PATH_PREFIX=/running_page npm run build

# 2. 准备 gh-pages 分支
git checkout --orphan gh-pages
rm -rf *
cp -r dist/* .

# 3. 提交并推送
git add -A
git commit -m "deploy: update running page"
git push origin gh-pages

# 4. 切回主分支
git checkout master
```

### 9.3 方法三：API 部署（国内网络专用）

当 `git push` 连不上 GitHub（国内 WSL2 常见问题），可以用 GitHub API 间接推送：

```python
# push_via_api.py - 通过 GitHub REST API 部署
import json, os, base64, subprocess

REPO = "你的用户名/running_page"
BRANCH = "gh-pages"
DIST_DIR = "./dist"

def gh(args, stdin_data=None):
    cmd = ["gh", "api"] + args.split()
    r = subprocess.run(cmd, input=stdin_data, capture_output=True, text=True)
    return json.loads(r.stdout)

# 1. 上传所有文件创建 blob
blobs = []
for root, dirs, files in os.walk(DIST_DIR):
    for f in files:
        full = os.path.join(root, f)
        rel = os.path.relpath(full, DIST_DIR)
        with open(full, "rb") as fp:
            content = fp.read()
        try:
            data = {"content": content.decode(), "encoding": "utf-8"}
        except:
            data = {"content": base64.b64encode(content).decode(), "encoding": "base64"}
        blob = gh(f"repos/{REPO}/git/blobs -X POST --input -", stdin_data=json.dumps(data))
        blobs.append((rel, blob["sha"]))

# 2. 创建 tree（不设 base_tree = 全新树）
tree = gh(f"repos/{REPO}/git/trees -X POST --input -",
    stdin_data=json.dumps({
        "tree": [{"path": p, "mode": "100644", "type": "blob", "sha": s} for p, s in blobs]
    }))

# 3. 创建 commit
commit = gh(f"repos/{REPO}/git/commits -X POST --input -",
    stdin_data=json.dumps({
        "message": "deploy",
        "tree": tree["sha"],
        "parents": []
    }))

# 4. 更新分支
gh(f"repos/{REPO}/git/refs/heads/{BRANCH} -X PATCH --input -",
   stdin_data=json.dumps({"sha": commit["sha"], "force": True}))
```

### 9.4 启用 GitHub Pages

1. 打开你的仓库 → **Settings** → **Pages**
2. **Source**: 选 **Deploy from a branch**
3. **Branch**: 选 **gh-pages** → **/(root)**
4. 点 **Save**
5. 等 1-2 分钟，访问：
   ```
   https://你的用户名.github.io/running_page/
   ```

### 9.5 如果页面 404/空白

常见原因排查：

| 现象 | 原因 | 修复 |
|------|------|------|
| 白屏，Console 报 404 | `PATH_PREFIX` 未设置 | 构建时加 `PATH_PREFIX=/running_page` |
| 地图不显示 | Mapbox 被墙 | 改成 `MAP_TILE_VENDOR = 'gaode'` |
| 控制台 `ERR_ADDRESS_INVALID` | 遥测上报被墙 | 裁剪 mapbox-gl 中的 events 地址 |
| 头像/图标不显示 | favicon 路径不对 | 确认 `images/favicon.png` 存在 |
| 数据不对（沈阳/大连） | 用了上游测试数据 | 用自己的 `activities.json` 构建 |

---

## 10. 日常更新数据

### 10.1 跑完步后更新

```bash
# 1. 同步新数据
cd running_page/run_page
python3 codoon_sync.py --phone 138xxxxxxx --password 你的密码

# 2. 重新构建数据
cd ..
python3 build_data.py

# 3. 复制数据到源码
cp src/static/activities.json /tmp/running_page_src/src/static/activities.json

# 4. 构建前端
cd /tmp/running_page_src
PATH_PREFIX=/running_page npm run build

# 5. 部署
cd /path/to/running_page_gh
rm -rf assets images .vite
cp -r /tmp/running_page_src/dist/* .
python3 push_via_api.py
```

### 10.2 一键更新脚本

```bash
#!/bin/bash
# update.sh - 一键更新跑步数据
set -e

PHONE="$1"          # 咕咚手机号
PASSWORD="$2"       # 咕咚密码
DATA_DIR="/home/dministrator/running_page"
BUILD_DIR="/tmp/running_page_src"
DEPLOY_DIR="/tmp/running_page_gh"

echo "1️⃣  同步咕咚数据..."
cd $DATA_DIR/run_page
python3 codoon_sync.py --phone $PHONE --password $PASSWORD

echo "2️⃣  构建数据..."
cd $DATA_DIR
python3 build_data.py

echo "3️⃣  复制数据到构建目录..."
cp src/static/activities.json $BUILD_DIR/src/static/activities.json

echo "4️⃣  构建前端..."
cd $BUILD_DIR
PATH_PREFIX=/running_page npm run build

echo "5️⃣  部署到 GitHub Pages..."
cd $DEPLOY_DIR
rm -rf assets images .vite
cp -r $BUILD_DIR/dist/* .
python3 /tmp/push_via_api.py

echo "✅ 完成！"
```

使用方式：
```bash
bash update.sh 138xxxxxxx 你的咕咚密码
```

### 10.3 注意事项

- **GitHub Actions 不能直接连咕咚 API**（美国节点会超时）
- 推荐的更新方式：在自己电脑上每月跑一次更新脚本
- 数据会增量同步，不会重复下载

---

## 11. 附录：踩坑记录

### 坑 1：使用了上游的测试数据

**现象：** 地图显示沈阳/大连的跑步轨迹，但实际跑在成都。

**原因：** 构建时用的 `activities.json` 来自上游仓库 `yihong0618/running_page`，里面包含的是作者的测试数据（3600 条沈阳记录）。自己的真实数据（492 条成都记录）在另一个目录。

**解决：** 构建前把自己的 `src/static/activities.json` 复制到构建目录：
```bash
cp 你的项目路径/src/static/activities.json /tmp/running_page_src/src/static/activities.json
```

### 坑 2：Mapbox 瓦片在国内被墙

**现象：** 地图空白，控制台报 `ERR_ADDRESS_INVALID`。

**解决：** 共 4 步（详见第 7 章）：
1. `MAP_TILE_VENDOR = 'gaode'`
2. style 键名用 `osm-bright` / `dark-matter`
3. 拦截 MapboxLanguage 插件
4. 裁剪 mapbox-gl 中的 events 地址

### 坑 3：Vite 构建路径不对

**现象：** 部署后页面白屏/404。

**解决：**
```bash
# ❌ 错误
npm run build

# ✅ 正确
PATH_PREFIX=/仓库名 npm run build
```

### 坑 4：git push 连不上 GitHub

**现象：**
```
fatal: unable to access 'https://github.com/...': Connection timed out
```

**解决：** 使用 GitHub API 代替 git 协议推送（见 9.3 节）。

### 坑 5：gh-pages 分支堆积旧文件

**现象：** 分支下有几十个不同 hash 的旧 mapbox-gl 文件。

**解决：** 创建 tree 时不设 `base_tree`：
```python
# ❌ 会保留旧文件
tree_data = {"base_tree": old_tree_sha, "tree": new_items}

# ✅ 只包含新文件
tree_data = {"tree": new_items}
```

### 坑 6：Mapbox GL JS v3 要求 access_token

**现象：** `Error: A valid Mapbox access token is required`

**解决：** 保留项目默认的 Mapbox token。即使使用高德瓦片，Mapbox GL JS v3 也需要一个 token 才能初始化。session 验证请求到 `api.mapbox.com` 会被墙，但**不影响地图渲染**。

### 坑 7：地图默认不显示街道

**现象：** 打开页面地图是暗的，要点 "Turn on the Light"。

**解决：** `const.ts` 中 `LIGHTS_ON = true`

---

> 祝你跑得开心 🐾
