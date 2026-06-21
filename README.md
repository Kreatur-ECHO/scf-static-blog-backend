# SCF Static Blog Backend

> 腾讯云 SCF Web 函数 — 为静态博客添加后端能力，零外部依赖，零成本（免费额度内）。

## 功能

| 端点 | 方法 | 功能 | 鉴权 |
|------|------|------|------|
| `/` | GET | 读取所有点赞计数 | 公开 |
| `/` | POST | 点赞（IP MD5 去重） | COS 签名 |
| `/visits` | GET | 读取累计访问数 | 公开 |
| `/visits` | POST | 记录访问（每日 IP 去重） | COS 签名 |
| `/article-views` | GET | 读取文章浏览计数 | 公开 |
| `/article-views` | POST | 文章浏览+1（每日 IP 去重） | COS 签名 |
| `/recent-song` | GET | 网易云公开歌单前 5 首 | 公开 |
| `/music-play` | GET | 搜索可播放音源（代理搜索站） | 公开 |

## 快速开始

### 1. 创建 COS Bucket

腾讯云 COS → 新建 Bucket（公有读私有写）→ 记下 Bucket 名和地域。

### 2. 创建 SCF Web 函数

- **类型**：⚠️ **Web 函数**（不是事件函数）
- **运行环境**：Node.js 18.15
- **代码**：复制 `app.js` 全部内容
- **环境变量**：
  - `SECRET_ID` — 腾讯云 API 密钥 ID
  - `SECRET_KEY` — 腾讯云 API 密钥 Key

### 3. 修改配置

在 `app.js` 顶部修改：

```js
const BUCKET = '你的-bucket-名-APPID';
const REGION = 'ap-guangzhou';  // 你的 COS 地域
```

如果使用网易云公开歌单，修改 `PLAYLIST_ID`：

```js
const PLAYLIST_ID = '你的歌单ID';
```

### 4. 配置前端

将 SCF 公网 URL 填入前端配置：

```js
// data/config.js
likesApi: 'https://你的SCF地址.ap-guangzhou.tencentscf.com',
```

## 架构

```
浏览器 ──GET/POST──→ SCF Web 函数 (Node.js HTTP Server, Port 9000)
                         │
                         ├── 读/写 ──→ COS (JSON 文件)
                         │
                         └── 代理 ──→ 网易云 API / 音源搜索站
```

## 关键实现

- **零依赖**：仅用 `http`、`https`、`crypto` 三个 Node.js 内置模块
- **COS 签名**：HMAC-SHA1 自签名，兼容 cos-nodejs-sdk-v5
- **IP 去重**：`x-forwarded-for` 只取第一个 IP（真实客户端），MD5 后存 COS
- **CORS**：公网 URL 配置 `Access-Control-Allow-Origin: *`
- **缓存策略**：每日 4:00 刷新网易云歌单，其余时间读 COS 缓存

## 来自 YEYU's Blog

本代码提取自 [Kreatur-ECHO.github.io](https://github.com/Kreatur-ECHO/Kreatur-ECHO.github.io) 项目。完整博客实践报告：[blog-build-journey](https://github.com/Kreatur-ECHO/blog-build-journey)

## License

MIT
