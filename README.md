# SAFE 資安掃描（2026 年 10 月版）

IG @marcorefusestocode 嘅 DM 禮物。今個月一次過出咗幾個大漏洞，呢度教你點樣做一次大掃除，有兩個做法。

## 做法 A：你有 Claude Code 或者 Codex

1. 打開呢個 repo 入面嘅 RUNBOOK 檔案
2. 撳右上角 Copy raw file
3. 貼入 Claude Code 或者 Codex，後面加一句：我啲項目喺 ______ 資料夾

AI 會幫你檢查以下幾樣：
- 部電腦同啲工具有冇更新到今個月修補咗嘅版本
- 你啲項目有冇用緊有問題嘅版本
- 有冇 AI key 擺咗喺檔案度

佢淨係睇，唔會改嘢，亦唔會顯示你條 key 嘅內容，只會話你知邊個檔案有。睇完報告，你打 fix 幾號，佢先會郁嗰一項。

測試過：macOS 上面嘅 Claude Code 同 Codex。Windows 未測試。

## 做法 B：冇 AI 工具，自己跟

自己：

1. 更新 iPhone：設定 Settings > 一般 General > 軟件更新 Software Update
2. 更新電腦：Mac 去 系統設定 > 一般 > 軟件更新；Windows 去 設定 > Windows Update
3. 更新 Chrome：右上角三點 > 說明 > 關於 Google Chrome，裝完撳重新啟動
4. 開雙重驗證：
   - WhatsApp：Settings > Account > Two-step verification
   - Apple ID：設定 > 你個名 > 登入與安全性 > 雙重認證
   - Google：myaccount.google.com > 安全性 > 兩步驗證
   - 銀行同股票 app：喺 app 入面搵保安設定
5. 有用 AI API 先要做：每月設上限，唔用嘅 key 刪咗佢
   - Anthropic：console.anthropic.com > Billing > Spend limits
   - Google Gemini：aistudio.google.com/spend
   - OpenAI：platform.openai.com > Settings > 個 project 嘅 Limits
   - OpenRouter：openrouter.ai/settings/keys，每條 key 都可以設 credit limit

有網站或者外判 IT 嘅老闆：

6. WhatsApp 你做網站嗰個，可以照抄：

   我哋網站 PHP 而家係咩版本？HKCERT 22 號出咗極高風險漏洞警報，要升到 8.2.35、8.3.36、8.4.27 或者 8.5.12 以上，可唔可以今個禮拜幫手更新？另外遙距登入同後台帳戶有冇開雙重驗證？

7. 外判 IT 可以遙距登入你間舖部電腦嘅話，問佢兩條問題：遙距登入軟件更新咗未？登入要唔要第二步驗證？
8. 舊員工同唔用嘅帳戶，刪咗佢

## 點解係今個月

- Apple 有個漏洞，美國網絡安全局（CISA）列明已經被黑客利用（2026 年 9 月 29 日）
- 香港 HKCERT 將 PHP 兩個漏洞評為極高度風險，話正被廣泛利用（2026 年 10 月 22 日）。大約七成網站用緊 PHP（W3Techs）
- 私隱專員公署 2026 年 4 月 23 日嘅報告：有會所嘅外判 IT 冇更新遙距登入軟件，又冇第二步驗證，結果中咗勒索軟件，9,045 人資料外洩
- 有細公司條 Gemini key 俾人偷咗，兩日內俾人用咗 US$82,314（The Register，2026 年 3 月 3 日）

## 講清楚

做齊呢啲係減低風險，唔保證 100% 安全，亦都唔係專業保安審計。

作者 @marcorefusestocode。MIT license。
