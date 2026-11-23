# 灵感地图

把「想去的地方」和「为什么想去」钉在地图上，去完之后回来补一段观后感。

- 地图是首页：拖到哪儿，点 `＋` 就在那儿记一笔
- 一个地点一条时间线，不是一个点一条灵感
- 地点有三态：**想去 → 去过 → 写过观后感**，不用手动维护
- 数据全在本地，一键导出成 zip 备份

---

## 现在能做什么

| 功能 | 状态 |
|---|---|
| 地图打点、气泡聚合、放大拆开 | ✓ |
| 缩小时按**省 → 市**聚合，点气泡逐级钻下去 | ✓ |
| 右侧 ＋/− 缩放按钮 | ✓ |
| 定位精度圈（±多少米一目了然） | ✓ |
| 记录页：长文、多图、草稿自动保存 | ✓ |
| 地点页：时间线、我去了、写观后感 | ✓ |
| 清单：想去 / 去过 / 写过，按城市分组 | ✓ |
| 导出 / 导入 zip 备份 | ✓ |
| 备份提醒（超过 7 天在清单页提示） | ✓ |
| 离线可用（壳子和看过的瓦片） | ✓ |
| 搜索、时间流、标签 | 下一版 |

---

## 本地跑起来

```bash
cd inspiration-map
python3 -m http.server 8787
```

浏览器打开 `http://127.0.0.1:8787`。

> 必须用 http 服务，直接双击 `index.html` 不行——ES 模块和地理定位都会被浏览器拦住。

手机连同一个 WiFi，用电脑的局域网 IP 访问也能试（`http://192.168.x.x:8787`）。
但**定位功能只在 HTTPS 或 localhost 下可用**，所以真要日常用，还是部署上去。

---

## 部署（5 分钟）

### GitHub Pages

1. 在 GitHub 新建一个仓库，比如 `inspiration-map`
2. 把本目录所有文件传上去（**不要传 `node_modules/`**）
3. 仓库 Settings → Pages → Source 选 `Deploy from a branch`，分支 `main`，目录 `/ (root)`
4. 等一分钟，访问 `https://<你的用户名>.github.io/inspiration-map/`

### Cloudflare Pages

1. Pages → 连接 Git 仓库
2. 构建命令留空，输出目录填 `/`
3. 部署完拿到 `https://xxx.pages.dev`

两条路都是 HTTPS，Service Worker、定位、加到主屏全都能用。

### iPhone 装到主屏

用 **Safari** 打开网址 → 分享按钮 → **添加到主屏幕** → 名字改成「灵感地图」。

图标、启动画面、全屏显示都会生效，和 App 手感一样。

### 安卓

Chrome 打开 → 菜单 → **安装应用** / 添加到主屏幕。

---

## 可选：填一个高德 key

不填也能用，区别是：

| | 不填 | 填了 |
|---|---|---|
| 搜地点 | 用 Photon（OSM 数据），能用；国内小店、新店可能没有 | 高德数据，国内 POI 准 |
| 自动给地点取名 | 有，Photon 反查，街区级（如「延安东路」） | 有，门牌 / POI 级 |

申请：<https://lbs.amap.com/> → 控制台 → 应用管理 → 创建应用 →
添加 Key → 服务平台选 **Web服务**（不是 Web端 JS API）→ 把 key 粘到 App 的「设置」里。

key 只存在你自己设备的浏览器里，不会发给别人。

---

### 搜索报错怎么办

早的版本用的是 OpenStreetMap 官方接口，国内经常连不上，会卡十几秒然后报
`signal is aborted without reason`。0.3.2 之后换成了 Photon，能用了。

如果还是超时，多半是网络问题；填了高德 key 就不会走这条线。

## 数据与备份

- 数据存在浏览器的 IndexedDB 里，**不上传任何服务器**
- 设置 → 导出备份，得到一个 zip：`backup.json` + `photos/*.jpg`
- 导入按 id 合并，重复的跳过，不会覆盖已有内容
- 换手机、清缓存、iOS 清理存储之前，**先导出**
- 超过 7 天没导出，清单页顶部会出现橙色提醒，点「导出」直接下载
- 设置里能看到「上次导出：3 天前」

⚠️ iOS Safari 对网页存储有清理机制，长期不打开可能被系统回收。
这不是可选项，是保命功能。

---

## 目录

```
index.html              页面骨架
css/app.css             全部样式
js/app.js               路由与交互
js/model.js             业务规则（状态推导、聚合）—— 换平台时抄这层
js/db.js                存储层（IndexedDB）—— 换 SQLite 只改这里
js/geo.js               坐标转换、定位、地名查询
js/map.js               Leaflet 地图与气泡
js/ui.js                提示、弹层、图片压缩
js/backup.js            zip 导入导出
sw.js                   Service Worker（离线）
vendor/                 Leaflet、fflate（本地化，不依赖 CDN）
test/                   测试
```

## 测试

```bash
npm i -D playwright && npx playwright install chromium
node test/smoke.mjs     # 业务规则 + zip 往返
node test/static.mjs    # id / import / 文件引用一致性
python3 -m http.server 8787 &
node test/e2e.mjs       # 真实浏览器跑一遍完整流程
```

---

## 已知限制

- 地图瓦片用的是高德未公开的栅格接口，免 key，但**不保证长期可用**。
  失效了改 `js/map.js` 里的 `TILE` 常量，换别的瓦片源即可，界面不用动。
- 没有多设备同步。两台手机之间靠导出/导入。
- iOS 上「添加到主屏」的网页，通知和后台能力都比原生 App 弱。

## 下一步

1. 先用两周，看会不会真的记
2. 记了再补：搜索、时间流、标签
3. 确认好用，再考虑用 Expo / Flutter 做成真 App —— 数据是 JSON，随时能迁
