# 🏃 Running_page 部署手册

> 从咕咚导出跑步数据 → 搭建个人跑步主页 → 部署到 GitHub Pages

---

## 目录

1. [环境准备](#1-环境准备)
2. [下载开源项目](#2-下载开源项目)
3. [导出咕咚数据](#3-导出咕咚数据)
4. [数据分析与构建](#4-数据分析与构建)
5. [私有化修改（推荐）](#5-私有化修改推荐)
6. [部署到 GitHub Pages](#6-部署到-github-pages)
7. [国内网络特殊配置（高德地图）](#7-国内网络特殊配置高德地图)
8. [日常更新数据](#8-日常更新数据)
9. [常见问题](#9-常见问题)

---

## 1. 环境准备

### 1.1 需要的工具

| 工具 | 用途 | 下载 |
|------|------|------|
| Python 3.10+ | 运行数据导出脚本 | https://python.org |
| Git | 版本管理 | https://git-scm.com |
| GitHub 账号 | 托管主页 | https://github.com |
| 手机号 | 注册 GitHub | — |

### 1.2 验证环境

```bash
python3 --version   # 需 ≥ 3.10
git --version
```

---

## 2. 下载开源项目

Running_page 是一个开源跑步主页项目，由 [yihong0618](https://github.com/yihong0618/running_page) 开发。

### 2.1 Fork 项目

1. 打开 https://github.com/yihong0618/running_page
2. 点右上角 **Fork** → 创建到你自己的 GitHub 账号下
3. 你得到自己的仓库：`https://github.com/你的用户名/running_page`

### 2.2 克隆到本地

```bash
git clone https://github.com/你的用户名/running_page.git
cd running_page
```

### 2.3 安装 Python 依赖

```bash
pip install -r requirements.txt
```

如果国内网络慢，换国内镜像：

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

---

## 3. 导出咕咚数据

### 3.1 准备工作

你需要：
- 咕咚账号（手机号 + 密码）
- 运行数据导出脚本

### 3.2 导出脚本

项目自带 `codoon_sync.py`，位于 `run_page/` 目录下。

```bash
cd run_page
python3 codoon_sync.py --phone 你的手机号 --password 你的密码
```

脚本会：
1. ✅ 登录咕咚 API
2. ✅ 下载所有跑步记录的 GPX 文件
3. ✅ 保存到 `GPX_OUT/` 目录
4. ✅ 生成数据库 `data.db`

### 3.3 注意事项

- **首次导出可能较慢**（几百次跑步需几分钟）
- 脚本会**增量同步**：下次只会下载新增的
- 如果报错 "invalid credentials"，检查手机号/密码
- 如果有验证码，脚本会提示输入

### 3.4 生成前端数据

导出完成后，构建前端需要的数据文件：

```bash
cd ..
python3 build_data.py
```

这会把 GPX 数据处理成 `src/static/activities.json` 等前端可读的格式。

---

## 4. 数据分析与构建

### 4.1 查看跑步统计

项目有分析脚本可以查看统计数据：

```bash
python3 analyze_runs.py
```

输出示例：
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

### 4.2 构建前端（可选，后续部署用）

安装前端依赖：

```bash
# 需要 Node.js 18+
npm install
```

构建：

```bash
# 如果是 Pages 部署在 /running_page/ 路径下
PATH_PREFIX=/running_page npm run build
```

构建产物在 `dist/` 目录。

---

## 5. 私有化修改（推荐）

根据个人喜好修改配置。

### 5.1 个人信息

编辑 `src/static/site-metadata.ts`：

```typescript
export const siteMetadata = {
  title: '🏃 我的咕咚跑步主页',
  description: '492次 / 6353km 跑步记录',
  author: '你的名字',
  siteUrl: 'https://你的用户名.github.io/running_page/',
};
```

### 5.2 地图配置（重要！中国用户必读）

编辑 `src/themes/classic/utils/const.ts`：

| 配置项 | 说明 | 推荐值 |
|--------|------|--------|
| `MAP_TILE_VENDOR` | 地图瓦片供应商 | `'gaode'`（高德，国内最快） |
| `IS_CHINESE` | 是否中文用户 | `true` |
| `LIGHTS_ON` | 默认显示地图 | `true` |
| `MAPBOX_TOKEN` | Mapbox 鉴权 token | 保持默认（仅用于 Mapbox GL JS 初始化） |

> **高德瓦片**是国内用户的最佳选择：免 Key、加载快、中文街景完整。

### 5.3 样式调整

```typescript
// 路线颜色
const nike = 'rgb(224,237,94)';  // 主色调

// 是否显示虚线路线
const USE_DASH_LINE = true;

// 路线透明度
const LINE_OPACITY = 0.4;

// 隐私模式（只显示路线不显示地图）
const PRIVACY_MODE = false;
```

---

## 6. 部署到 GitHub Pages

### 6.1 方法一：GitHub Actions（自动部署）

在项目根目录下创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy
on:
  push:
    branches: [main]
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

推送到 GitHub 后自动部署。

### 6.2 方法二：手动部署

```bash
# 构建
PATH_PREFIX=/running_page npm run build

# 切换到 gh-pages 分支
git checkout --orphan gh-pages
rm -rf *
cp -r dist/* .
git add -A
git commit -m "deploy"
git push origin gh-pages

# 切回主分支
git checkout main
```

### 6.3 启用 GitHub Pages

1. 打开仓库 → Settings → Pages
2. Source: **Deploy from a branch**
3. Branch: **gh-pages** → **/(root)**
4. 点 Save
5. 等待 1-2 分钟，访问 `https://你的用户名.github.io/running_page/`

---

## 7. 国内网络特殊配置（高德地图）

### 7.1 为什么需要

Mapbox GL JS 默认使用 Mapbox 的瓦片服务，在国内**完全被墙**，地图无法加载。高德瓦片是国内最好的替代方案。

### 7.2 配置方法

编辑 `src/themes/classic/utils/const.ts`：

```typescript
// 修改第 1 处：瓦片供应商
export const MAP_TILE_VENDOR = 'gaode';

// 修改第 2 处：默认亮灯
const LIGHTS_ON = true;
```

### 7.3 如果还遇到问题

如果地图依然无法显示，检查浏览器的控制台：

- `ERR_ADDRESS_INVALID` → Mapbox 遥测被墙 → 已在源码中修复
- `CORS` 错误 → GitHub Pages 配置问题
- 白屏 → 检查 `PATH_PREFIX` 是否设置正确

### 7.4 验证效果

部署后打开页面，地图应该：
- ✅ 秒开（高德国内 CDN）
- ✅ 显示中文街道名称
- ✅ 显示 POI（餐馆、地铁站等）
- ✅ 跑步轨迹彩色标注

---

## 8. 日常更新数据

### 8.1 手动更新

跑完新的步后，重新导出并构建：

```bash
# 同步新数据
cd run_page
python3 codoon_sync.py --phone 你的手机号 --password 你的密码

# 重新构建
cd ..
python3 build_data.py
PATH_PREFIX=/running_page npm run build

# 部署
# 复制 dist/ 到 gh-pages 分支
```

### 8.2 注意事项

- GitHub Actions **无法**直接连接咕咚 API（中国区 GitHub Actions 可能超时）
- 推荐：在自己电脑上定期（比如每月）执行一次更新
- 可以写个简单脚本一键更新：

```bash
#!/bin/bash
# update.sh
cd run_page
python3 codoon_sync.py --phone $PHONE --password $PASSWORD
cd ..
python3 build_data.py
PATH_PREFIX=/running_page npm run build
echo "数据已更新，请手动复制 dist/ 到 gh-pages 分支"
```

---

## 9. 常见问题

### Q: 导出时提示 Invalid Credentials？

A: 检查手机号和密码，确认咕咚账号正常。如果改了密码需要更新脚本参数。

### Q: 地图不显示中文地名？

A: 检查 `MAP_TILE_VENDOR` 是否设为 `'gaode'`。如果是 `'mapcn'` 或 `'mapbox'`，国内不会显示中文。

### Q: 控制台报 `api.mapbox.com` 错误？

A: 这是 Mapbox GL JS 的 session 验证请求，在国内会被墙，但**不影响地图显示**，可以忽略。

### Q: 更新数据后页面没变？

A: GitHub Pages 有 CDN 缓存，通常 1-2 分钟生效。如果需要强制刷新，可以加 `?v=2` 参数访问。

### Q: 部署后页面空白/404？

A: 常见原因：
1. `PATH_PREFIX` 未设置或设置错误（仓库名是否匹配）
2. `gh-pages` 分支内容不完整（确保 `dist/` 内容完整复制）
3. 资源路径不对（检查浏览器控制台的 404 地址）

---

> 如有其他问题，欢迎提 Issue 或联系我 🐼

---

## 附录：踩坑记录 & 最终解决方案

下面记录了本项目从咕咚数据导出到部署完成过程中遇到的所有问题和最终解法。

### 坑 1：咕咚 API 签名验证失败

**现象：** `codoon_sync.py` 报 `401 Unauthorized`

**原因：** 咕咚 API 需要模拟手机 App 的请求头（包括 User-Agent、设备信息、签名算法）。GitHub 上的脚本默认模拟的是 Android 7 + 咕咚 App 8.9.0。

**解决：** 脚本已封装好签名算法（HMAC-SHA1，Base64），需要确保：
- 手机号 + 密码正确
- 如果改过密码，重新运行脚本
- 脚本不会自动检测 token 过期，必要时删掉 `data.db` 重新登录

---

### 坑 2：GPX 导出不完整 / 丢失轨迹

**现象：** 部分跑步记录导出了但轨迹点很少（只有起点终点）

**原因：** 咕咚 API 限制单次请求返回的记录条数，部分长轨迹被截断

**解决：**
- 脚本已分批请求，每次最多 20 条
- 增量同步（只下载新记录）避免重复
- 最终用 `polyline.decode()` 解码轨迹 polyline 字符串

---

### 坑 3：Mapbox 瓦片在国内完全被墙

**现象：** 地图一片空白，控制台报 `net::ERR_ADDRESS_INVALID`

**根源问题链：**
```
events.mapbox.com  ↛  遥测上报（被墙）
api.mapbox.com     ↛  样式/鉴权（被墙）
*.tiles.mapbox.com ↛  地图瓦片（被墙）
```

**最终解法（4 步）：**

**第 1 步：瓦片源改高德**

修改 `const.ts`：
```typescript
export const MAP_TILE_VENDOR = 'gaode';
```

新增高德内联 style（Mapbox GL JS 的 style JSON 格式）：
```json
{
  "version": 8,
  "sources": {
    "gaode-raster": {
      "type": "raster",
      "tiles": [
        "https://webrd01.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=8&x={x}&y={y}&z={z}"
      ],
      "tileSize": 256
    }
  },
  "layers": [{
    "id": "gaode-raster-layer",
    "type": "raster",
    "source": "gaode-raster",
    "minzoom": 0,
    "maxzoom": 18
  }]
}
```

**第 2 步：修复样式名不匹配**

Gaode 的 style 对象键名必须是 `osm-bright` / `dark-matter`（不能是 `light` / `dark`），否则会报：
```
Style "osm-bright" is not valid for vendor "gaode"
```

**第 3 步：修复 MapboxLanguage 插件冲突**

MapboxLanguage 插件只兼容 Mapbox vector tile v8 样式，加载自定义样式时会崩溃。加守卫：
```typescript
if (MAP_TILE_VENDOR === 'mapbox' || MAP_TILE_VENDOR === 'mapcn') {
  map.addControl(new MapboxLanguage({ defaultLanguage: 'zh-Hans' }));
}
```

**第 4 步：裁剪遥测上报**

Mapbox GL JS v3 会向 `events.mapbox.com` 发包遥测，这个域名在国内被墙，导致 `ERR_ADDRESS_INVALID` 进而破坏地图渲染。

在 `node_modules/mapbox-gl/dist/mapbox-gl.js` 中替换域名：
```python
# 构建前执行的补丁脚本
data = data.replace(b'events.mapbox.com', b'0-events.local')
data = data.replace(b'events.mapbox.cn', b'0-events.local')
```

注意不能简单替换旧域名，因为 URL 由多段拼接而成（`hostname + '/events/v2'`），替换不精确会导致双 `/events` 路径。

---

### 坑 4：Vite 构建路径不对

**现象：** 部署后页面 404，favicon 不显示

**原因：** Vite 的 `base` 配置默认为 `/`，但 GitHub Pages 的子路径部署需要设为 `/仓库名/`

**构建命令的坑：**
```bash
# ❌ 错误：资源路径变成 /assets/xxx.js，浏览器去根路径找
npm run build

# ✅ 正确：使用 PATH_PREFIX 环境变量
PATH_PREFIX=/running_page npm run build
```

`vite.config.ts` 中读取环境变量：
```typescript
base: process.env.PATH_PREFIX ? `${process.env.PATH_PREFIX}/` : '/',
```

---

### 坑 5：Git Push 连不上 GitHub（从 WSL2/国内）

**现象：**
```
fatal: unable to access 'https://github.com/...': 
  Failed to connect to github.com port 443: Connection timed out
```

**原因：** WSL2 在国内网络环境对 GitHub 的连接不稳定

**解法 A（GH API）：** 绕过 git 协议，直接用 GitHub REST API 创建 commit：
```bash
# 1. 上传每个文件创建 blob → 获取 SHA
# 2. 用所有 SHA 创建 git tree
# 3. 用 tree SHA 创建 commit
# 4. 更新 ref 到新 commit

gh api repos/owner/repo/git/blobs -X POST --input blob.json
gh api repos/owner/repo/git/trees -X POST --input tree.json
gh api repos/owner/repo/git/commits -X POST --input commit.json
gh api repos/owner/repo/git/refs/heads/gh-pages -X PATCH --input ref.json
```

**解法 B（SSH 隧道）：** 配置 SSH key 后通过 git@ 协议推送（前提是 SSH 端口 22 未被墙）

---

### 坑 6：多次部署后仓库残留旧文件

**现象：** `gh-pages` 分支下有几十个不同 hash 的旧 mapbox-gl 文件

**原因：** 每次构建生成新的 hash，旧文件不会自动删除

**解决：** 创建 git tree 时 **不设 `base_tree`**，从零构建一颗新树：
```python
# ❌ 会保留旧文件
tree_data = {"base_tree": old_tree_sha, "tree": new_items}

# ✅ 只包含新文件，旧文件自动消失
tree_data = {"tree": new_items}
```

---

### 坑 7：Mapbox GL JS v3 强制要求 access_token

**现象：**
```
Error: A valid Mapbox access token is required to use Mapbox GL JS.
```

**原因：** 即使使用自定义瓦片样式（高德），Mapbox GL JS v3 也必须配置一个 access_token 才能运行。

**解决：** 保留项目中默认的 Mapbox token。它仅用于库初始化（session 验证），不会影响高德瓦片的加载。session 验证请求到 `api.mapbox.com` 在国内依然会失败，但**不影响地图渲染**。

---

### 坑 8：地图默认不显示街道（需要点 Turn on the Light）

**现象：** 打开页面地图是暗的，点了 "Turn on the Light" 按钮后才显示

**原因：** `const.ts` 中 `LIGHTS_ON = false`，这个设计原本是为了隐私模式暗掉地图

**解决：**
```typescript
// const.ts
const LIGHTS_ON = true;  // 默认显示
```

---

### 坑 9：Mapbox GL JS 的 telemetry 请求导致地图崩溃

**现象：** 地图刚打开正常显示，点几下（平移/缩放）后街道消失

**原因链：**
1. 高德瓦片加载成功，地图正常显示
2. 用户操作 → Mapbox GL JS 触发 postStyleLoadEvent
3. 尝试向 `events.mapbox.com` 发送遥测数据
4. 域名被墙 → `ERR_ADDRESS_INVALID`
5. Mapbox GL JS 内部状态出错 → 地图图层被重置/隐藏

**解决：** 从 mapbox-gl 源码中直接删除遥测上报地址（见坑 3 第 4 步）
