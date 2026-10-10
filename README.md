# 那些沒說出口的話 — 匿名投稿箱 V1

Next.js + Supabase + Cloudflare Turnstile。訪客匿名投稿，管理員私人閱讀。沒有公開故事牆。

## 1. Supabase 資料庫

1. 建立 Supabase 專案。
2. 在 SQL Editor 執行 `supabase/schema.sql`。
3. 在 Project Settings 找到 Project URL 和 `service_role` key。**service_role key 只允許放在伺服器環境變數，不可放在 NEXT_PUBLIC 變數或公開程式庫。**

## 2. Cloudflare Turnstile

在 Cloudflare Turnstile 建立網站，設定正式網站域名，取得 Site Key 與 Secret Key。這是正式投稿的必要條件；沒有設定就會拒絕投稿。

## 3. 本機啟動

```bash
npm install
cp .env.example .env.local
```

填寫 `.env.local` 的各項值，並另外加上：

```env
NEXT_PUBLIC_TURNSTILE_SITE_KEY=your_turnstile_site_key
```

`ADMIN_PASSWORD` 至少 16 字元。`SESSION_SECRET` 至少 32 字元，建議使用 `openssl rand -hex 32` 產生。

```bash
npm run dev
```

瀏覽 `http://localhost:3000`，投稿在 `/write`，管理員登入在 `/admin`。

## 4. 正式部署 Vercel

1. 把專案推送到自己的 **Private GitHub repository**（勿上傳 `.env.local`）。
2. 在 Vercel 建立 New Project，匯入 GitHub repository。
3. 在 Vercel Environment Variables 設定 `SUPABASE_URL`、`SUPABASE_SERVICE_ROLE_KEY`、`ADMIN_PASSWORD`、`SESSION_SECRET`、`TURNSTILE_SECRET_KEY`、`NEXT_PUBLIC_TURNSTILE_SITE_KEY`。
4. Deploy。取得網站域名後，在 Cloudflare Turnstile 加入該域名。自訂域名則在 Vercel Domains 設定 DNS。
5. 實際測試：未授權投稿、已授權投稿、頻率限制、管理登入、CSV 匯出、刪除、登出。

## 5. 資安與營運注意

- 匿名是指不要求訪客提供姓名或帳號，不是保證完全無法識別。基礎設施供應商可能記錄連線資訊；本系統以加鹽雜湊 IP 作為短期投稿頻率限制用途。
- 授權勾選預設關閉。沒有勾選的投稿不得公開或改寫分享到 Threads。
- 建議正式上線前請法務確認隱私告知、資料保留/刪除政策、使用者撤回授權及個資處理流程。匿名投稿者目前沒有自行查詢或撤回的驗證機制。
- 管理登入目前採單一管理員密碼和 8 小時簽章 Cookie。正式大量使用前建議加入登入限速、多因素驗證、稽核日誌及管理員帳號機制。
- 管理列表目前最多顯示最近 200 筆，正式長期運作應加分頁。
- 建議建立 Supabase 自動備份與資料保存週期，並定期清理 `submission_limits`。
- Turnstile 可阻擋部分機器人，無法完全防止垃圾訊息；若有大量濫用，應加上 WAF、IP 速率限制與審核流程。
- **不要把測試投稿當成已送達，除非已連接 Supabase 並在後台確認。**

## 6. 授權原則

投稿者明確勾選才同意站長匿名引用或改寫至 Threads；未勾選僅供站長私人閱讀。正式營運時應加入完整條款、授權撤回及可辨識資訊移除流程。
