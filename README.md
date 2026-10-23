# 💬 GPT Chat App — React + FastAPI POC

這是一個技術驗證（POC）專案，目的是熟悉 **React + Python FastAPI** 的前後端分離架構，以及 API 的設計與建置。

以 GPT 聊天應用為題材，實作帳號註冊登入、對話管理與訊息紀錄，並用 Docker Compose 把前端、後端、資料庫與 Nginx 串成一套可部署的服務。

> ⚠️ 目前 `/api/chat` 回傳的是示範回覆，尚未串接 OpenAI（程式中保留 `TODO`），重點在 API 與整體架構。

---

## 🔧 使用技術

| 類別 | 技術 |
|---|---|
| 前端 | React 19（Create React App） |
| 後端 | Python FastAPI、SQLAlchemy 2、Pydantic Settings |
| 驗證 | JWT（python-jose）、bcrypt 密碼雜湊（passlib） |
| 資料庫 | PostgreSQL（使用者、對話）、MongoDB（訊息內容，Motor 非同步存取） |
| 部署 | Docker Compose、Nginx 反向代理、Certbot（Let's Encrypt HTTPS） |

---

## 🧱 架構

```
Browser
  ↓
Nginx（80 → 443 轉址、HTTPS）
  ├── /      → frontend（React 靜態檔，容器內 nginx :8081）
  └── /api/  → backend（FastAPI :8080）
                 ├── PostgreSQL：users、conversations
                 └── MongoDB：messages
```

關聯性資料（使用者、對話擁有權）放 PostgreSQL，量大且結構單純的訊息內容放 MongoDB，練習兩種資料庫的分工。

---

## 📂 API

| 方法 | 路徑 | 說明 | 需登入 |
|---|---|---|---|
| POST | `/api/register` | 註冊（username、email、password） | |
| POST | `/api/login` | 登入，回傳 JWT | |
| POST | `/api/conversations` | 建立新對話 | ✅ |
| GET | `/api/conversations` | 列出自己的對話 | ✅ |
| POST | `/api/chat` | 送出訊息並取得回覆（先驗證對話擁有權） | ✅ |
| GET | `/api/messages?conversation_id=` | 讀取對話的訊息紀錄 | ✅ |

---

## 🚀 啟動方式

1. 在專案根目錄建立 `.env`（不會進版控）：

```env
PG_DSN=postgresql+psycopg2://rag_user:rag_password@postgres:5432/rag_demo
MONGO_DSN=mongodb://mongo:27017
MONGO_DB=rag_demo
JWT_SECRET=your_jwt_secret
JWT_ALG=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
OPENAI_API_KEY=your_openai_key
```

2. 把 `nginx/conf.d/app.conf` 裡的 `YOUR_DOMAIN` 換成自己的網域，並先用 certbot 簽好憑證。

3. 啟動：

```bash
docker compose up -d --build
```
