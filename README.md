>[!NOTE]
>本项目已成功构建，版本v1.0

## SecureBin： 一个零知识端到端加密隐私笔记工具

![SECUREBIN-零知识加密-2a7c74](https://img.shields.io/badge/SECUREBIN-零知识加密-2a7c74)![license-MIT-green](https://img.shields.io/badge/license-MIT-green)![Python-3.8%2B-blue](https://img.shields.io/badge/Python-3.8%2B-blue)![JavaScript-ES6%2B-yellow](https://img.shields.io/badge/JavaScript-ES6%2B-yellow)![Web%20Crypto%20API-支持-brightgreen](https://img.shields.io/badge/Web%20Crypto%20API-支持-brightgreen)

Securebin 是一个零知识端到端加密隐私笔记工具，所有数据在客户端加密后发送到服务器，服务器无法访问任何明文内容。

## ✨ 核心特性

### 🔐 零知识加密

- **客户端加密**：所有数据在浏览器中使用 Web Crypto API 加密
- **服务器零访问**：服务器只存储加密数据，无法解密
- **端到端保护**：数据从创建到查看全程加密

### 🛡️ 安全特性

- **AES-GCM-256 加密**：行业标准认证加密算法
- **PBKDF2 密钥派生**：100,000 次迭代的密码加强
- **随机盐值和 IV**：每次加密使用全新随机参数
- **阅后即焚**：可选阅后立即销毁模式
- **密码保护**：可选端到端密码保护

### 📝 功能特点

- **Markdown 编辑器**：完整的 Markdown 基础语法及部分扩展语法支持
- **工具箱**：可折叠的格式化工具栏
- **文件附件**：支持上传和加密各类文件
- **多种销毁模式**：1 小时、12 小时、1 天、7 天、永不过期
- **二维码分享**：一键生成分享二维码
- **响应式设计**：完美适配桌面和移动设备

### 🏗️ 系统架构

```
┌─────────────────┐     加密数据     ┌──────────────┐
│    用户浏览器    │ ──────────────> │  Flask服务器  │
│    (客户端)      │ <────────────── │   (服务端)   │
│                 │     加密数据     │              │
├─────────────────┤                 ├──────────────┤
│ Web Crypto API  │                 │ 存储加密数据  │
│    本地加解密    │                 │   API端点    │
│     密钥管理     │                 │   清理任务   │
│     用户界面     │                 │              │
└─────────────────┘                 └──────────────┘
        ▲                                    ▲
        │                                    │
        ▼                                    ▼
┌───────────────────┐                 ┌──────────────┐
│     URL片段密钥    │                 │ SQLite数据库 │
│       (#key)      │                 │   加密内容    │
│ (从未发送到服务器) │                 │    元数据     │
└───────────────────┘                 └──────────────┘
```

## 🚀 快速开始

### 构建环境

- Python 3.8
- 现代浏览器（支持 Web Crypto API）
- SQLite3（已内置）

### 安装步骤

**目前本工具已成功构建可执行文件版本**

1. 下载`exe`可执行文件到你的电脑

2. 双击直接运行`exe`可执行文件

3. 打开浏览器访问：http://localhost:5000

## 📁 项目结构

```
SecureBin/
├── app.py                 # Flask后端服务器
├── securebin.db           # SQLite3 数据库（自动创建）
├── templates/
│   ├── index.html         # 主编辑器页面
│   └── view.html          # 笔记查看页面
├── static/                # 静态资源（可选）
└── README.md              # 本文件
```

## 🔧 技术栈

### 后端

- **Flask**: 轻量级 Python Web 框架
- **SQLite3**: 嵌入式数据库
- **APScheduler**: 后台定时任务
- **UUID**: 唯一标识符生成

### 前端

- **Web Crypto API**: 浏览器原生加密
- **Tailwind CSS**: 实用优先的 CSS  框架
- **Lucide Icons**: 精美、简约且流行的图标库
- **Marked.js**: Markdown 解析器
- **QRCode.js**: 二维码生成库

### 加密算法

- **AES-GCM-256**: 认证加密算法
- **PBKDF2**: 密码密钥派生函数
- **SHA-256**: 安全哈希算法
- **随机盐值+IV**: 防止重放攻击

## 🔐 加密流程详解

1. 创建笔记（加密）

```javascript
// 1. 生成随机参数
const salt = crypto.getRandomValues(new Uint8Array(16));
const iv = crypto.getRandomValues(new Uint8Array(12));
const uK = Math.random().toString(36).slice(2, 12); // 用户密钥

// 2. 密钥派生
const key = await crypto.subtle.deriveKey(
    { name: "PBKDF2", salt, iterations: 100000, hash: "SHA-256" },
    await crypto.subtle.importKey("raw", new TextEncoder().encode(uK + pw), "PBKDF2", false, ["deriveKey"]),
    { name: "AES-GCM", length: 256 },
    false,
    ["encrypt"]
);

// 3. 数据加密
const encData = await crypto.subtle.encrypt(
    { name: "AES-GCM", iv },
    key,
    new TextEncoder().encode(JSON.stringify({ txt, atts }))
);

// 4. 发送到服务器（仅加密数据）
fetch('/api/create', {
    method: 'POST',
    body: JSON.stringify({
        content: b64(encData),  // 加密内容
        salt: b64(salt),        // 随机盐值
        iv: b64(iv),           // 随机IV
        // ... 其他元数据
    })
});
```

2. 查看笔记（解密）

```javascript
// 1. 从 URL 获取用户密钥
const uK = location.hash.slice(1);

// 2. 从服务器获取加密数据
const rD = await fetch(`/api/get/${noteId}`);

// 3. 本地解密
const dec = await crypto.subtle.decrypt(
    { name: "AES-GCM", iv: db64(rD.iv) },
    await crypto.subtle.deriveKey(
        { name: "PBKDF2", salt: db64(rD.salt), iterations: 100000, hash: "SHA-256" },
        await crypto.subtle.importKey("raw", new TextEncoder().encode(uK + pw), "PBKDF2", false, ["deriveKey"]),
        { name: "AES-GCM", length: 256 },
        false,
        ["decrypt"]
    ),
    db64(rD.data)
);

// 4. 显示解密内容
const plaintext = JSON.parse(new TextDecoder().decode(dec));
```

## 📊 API 接口

- 创建新的加密笔记

`/api/create`（方法：`POST`）

### 请求体

```json
{
    "content": "base64加密数据",
    "salt": "base64盐值",
    "iv": "base64初始化向量",
    "expiry_type": "time|burn|never",
    "duration": "1h|12h|1d|7d",
    "has_pwd": true|false
}
```

### 响应

```json
{
    "id": "笔记唯一ID"
}
```

- 获取加密笔记数据

`/api/get/<note_id>`（方法`GET`）

- 阅后即焚销毁笔记

`/api/burn/<note_id>`（方法`POST`）

## 🎨 编辑器功能

### 工具箱功能

- **默认隐藏**：工具箱面板初始隐藏，节省空间
- **一键切换**：点击工具箱按钮显示/隐藏工具栏
- **格式化工具**：
  - 标题切换（`#`/`##`/`###`）
  - 粗体、斜体、删除线
  - 代码块、列表、引用
  - 表格、链接、图片、分隔线

### Markdown 支持

```markdown
# 标题
## 副标题
### 小标题
#### 四级标题

**粗体**
*斜体*
~~删除线~~

- 无序列表
    - 子列表

1. 有序列表 1
2. 有序列表 2

> 引用块

`行内代码`

```代码块```

---

[链接](url)

![预览](url)

| 表格标题 1 | 表格标题 2 | 表格标题 3 |
|------|------|------|
| 表格内容 1 | 表格内容 2 | 表格内容 3 |
```

### 文件附件

- **本地加密**：文件在客户端加密后上传
- **格式支持**：支持所有文件类型
- **安全下载**：解密后本地下载附件
- **图片预览**：支持图片文件预览

## ⚙️ 配置选项

### 服务器配置（app.py）

```python
# 修改端口
if __name__ == '__main__': 
    app.run(debug=False, port=5000)  # 支持修改默认端口号

# 修改文件大小限制（默认 16 MB）
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024

# 修改清理间隔（默认 5 分钟）
scheduler.add_job(func=cleanup, trigger="interval", minutes=5)
```

### 前端配置

- **主题**：自动跟随系统或手动切换
- **编辑器**：Markdown 实时预览
- **销毁时间**：1 小时、12 小时、1 天、7 天
- **密码保护**：可选访问密码

## 🔒 安全性

### 优势

1. 零知识架构：服务器无法访问任何明文
2. 现代加密：使用 Web Crypto API 原生实现
3. 强随机性：使用`crypto.getRandomValues()`生成随机数
4. 认证加密：AES-GCM 提供完整性和机密性
5. 密钥派生：PBKDF2 防止暴力破解

### 自行构建需要注意的事项

1. URL 密钥片段：用户密钥在`URL`的`#`片段中
- **优点**：简单易用，无需额外存储
- **注意**：浏览器历史记录中可见
2. 客户端依赖：安全性完全依赖客户端 JavaScript
- 确保使用 HTTPS 部署
- 考虑添加完整性检查
3. 文件大小：大文件可能影响性能
- 如需加密大文件，建议添加分块加密支持

## 🚀 部署指南

### 本地部署

```bash
# 1. 安装依赖
pip install flask apscheduler

# 2. 运行服务器
python app.py

# 3. 访问应用
# 浏览器打开: http://localhost:5000
```

### 生产部署建议

1. 使用 HTTPS：确保所有流量加密
2. 反向代理：使用 Nginx/Apache 作为反向代理
3. 进程管理：使用 gunicorn 或 uWSGI
4. 数据库备份：定期备份 SQLite3 数据库
5. 日志监控：设置应用日志和错误监控

## 📱 浏览器兼容性

| 浏览器 | 加密支持 | 最低版本 |
|------|------|------|
| Chrome | ✅ | 60+ |
| Firefox | ✅ | 63+ |
| Safari | ✅ | 14.1+ |
| Edge | ✅ | 79+ |
| Opera | ✅ | 47+ |

要求： 需要支持 Web Crypto API 的现代浏览器

## ✒️ 测试数据

### 已通过测试的加密流程

1. 创建带有密码的笔记
2. 上传附件文件
3. 设置阅后即焚
4. 通过链接访问
5. 输入密码解密
6. 验证数据完整性

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork项目
2. 创建功能分支 (git checkout -b feature/AmazingFeature)
3. 提交更改 (git commit -m 'Add some AmazingFeature')
4. 推送到分支 (git push origin feature/AmazingFeature)
5. 打开Pull Request

## 📄 许可证

本项目基于 MIT 许可证开源

## 🙏 致谢

- **Web Cryptography API** - W3C 标准
- **Flask** - Python 微框架
- **Tailwind CSS** - CSS 框架
- **Lucide Icons** - 图标库

## 📞 支持

- **问题反馈**：GitHub Issues
- **功能建议**：GitHub Discussions
- **安全报告**：通过 GitHub 私密报告

---

**免责声明**

1. 此工具用于教育和研究目的，用户应对自己的数据安全负责。
2. 此工具核心源码由 ai 生成
3. 请务必确保您的隐私和数据安全，开发者不对数据丢失或隐私泄露承担任何责任。

**务必注意**

您的密码就是您的密钥。如果您忘记了密码，数据将会永久遗失且无法恢复，请务必妥存您的密码。。
