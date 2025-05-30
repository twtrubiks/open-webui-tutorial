# Open WebUI 手把手教學

* [Youtube Tutorial - AI 模型太多管理不易？ Open WebUI 幫你搞定！從安裝到企業級應用](https://youtu.be/qdgwWvpGvDc)

[open-webui](https://github.com/open-webui/open-webui) 多功能 AI 模型整合平台,

Open WebUI 是一個可自架的 AI 平台，能幫您**統一管理並輕鬆切換多種 AI 模型**，無論是本地的 Ollama 還是外部的 Gemini、OpenAI API 等等.

**核心亮點：**

* **集中管理：** 單一介面存取您所有的 AI 模型 API Key。
* **企業適用：** 支援自架、離線運作，並提供帳號與權限管理，確保資料安全。
* **靈活擴展：** 功能豐富且具可擴展性。

簡單來說，Open WebUI 讓您在公司內安全、方便使用多樣化的 AI 模型服務，並能有效管理使用者權限。

## 安裝方法

建議用 docker, 我寫了一板 [docker-compose.yaml](docker-compose.yaml),

執行 `docker compose up -d`, 接著瀏覽 [http://localhost:3000](http://localhost:3000)

這個版本要安裝 Ollama(安裝在本機, 不是 docker ), 之前有介紹過 [Ollama 簡介](https://github.com/twtrubiks/dify-ollama-docker-tutorial/blob/main/ollama.md)

文件中有提供非常多種的選項,

像是有 GPU 版本

`docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama`

只有 CPU 版本

`docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama`


甚至你不需要 Ollama, 也可以安裝沒有 Ollama 的版本.

for OpenAI API Usage Only

`docker run -d -p 3000:8080 -e OPENAI_API_KEY=your_secret_key -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main`

## Project workflow

**順序圖 (Sequence Diagram)** [Project workflow](https://github.com/open-webui/open-webui/tree/main/docs)

![alt tag](https://i.imgur.com/Gn9ZNcm.png)

整個架構使用 前端(Frontend) Svelte + 後端 (Backend) Python FastAPI 完成

1.  **使用者開啟網頁 (User opens browser, visits localhost:8080)**：
    * 使用者在他們的瀏覽器中輸入 Open WebUI 的網址。

2.  **Open WebUI 伺服器回應 (Returns welcome page from svelte build)**：
    * Open WebUI 伺服器（用 Python 編寫，並使用 Svelte 進行前端建構）收到請求後，會回傳預先建構好的頁面給使用者的瀏覽器。

3.  **網頁介面載入 (WebUI is loaded to browser)**：
    * 瀏覽器成功載入 Open WebUI 的前端介面。

4.  **前端發送 API 請求給 Open WebUI 伺服器 (Sends API request to /api/v1)**：
    * 使用者在 WebUI 上進行操作（例如輸入問題、選擇模型等），前端會將這些操作轉換成一個 API 請求，發送給 Open WebUI 伺服器的 `/api/v1` 端點。

5.  **Open WebUI 伺服器回應前端 (Returns API response data)**：
    * Open WebUI 伺服器處理這個內部 API 請求（可能涉及一些自身邏輯，如使用者驗證、對話管理等），然後回傳相應的資料給前端 WebUI。

6.  **前端處理 Open WebUI 伺服器的回應 (WebUI receives and processes API response data)**：
    * 前端 WebUI 接收並處理來自 Open WebUI 伺服器的回應資料，可能會更新介面顯示。

5 和 6 這部份, 可以把它們理解為 Open WebUI 應用程式內部的通訊和準備階段.

(例如 新的聊天訊息、載入歷史紀錄、顯示錯誤提示等)

7.  **前端發送請求給 Ollama (Sends API request to /ollama/api)**：
    * 接下來，如果需要與語言模型互動，前端 WebUI（或者是透過 Open WebUI 伺服器代理）會將請求發送給一個指向 Ollama API 的路徑（例如 `/ollama/api`）。
    * **圖中顯示的是 WebUI 直接發送給 Open WebUI Server，然後 Open WebUI Server 再代理這個請求給 Ollama Server。**

如果今天是呼叫外部的 Gemini, 這邊的 Ollama 就是變成 Gemini.

8.  **Open WebUI 伺服器代理請求至 Ollama 伺服器 (Proxies request to ollama server)**：
    * Open WebUI 伺服器將收到的請求轉發 (代理) 給實際運行的 Ollama 伺服器。

9.  **Ollama 伺服器處理並回應 (Responds with data)**：
    * Ollama 伺服器（運行大型語言模型）處理這個請求（例如，生成文字回應），然後將結果資料回傳給 Open WebUI 伺服器。

10. **Open WebUI 伺服器將 Ollama 的回應轉發給前端 (Returns API response data)**：
    * Open WebUI 伺服器再將從 Ollama 收到的回應資料回傳給前端 WebUI。

11. **使用者接收並處理最終回應 (User receives and processes API response data)**：
    * 前端 WebUI 接收到來自 Ollama（經由 Open WebUI 伺服器）的回應資料，並將其顯示給使用者。

**總結**

* **使用者 (User)**：透過瀏覽器與 Open WebUI 互動。
* **Open WebUI 伺服器 (Python)**：作為中間層，處理前端請求、管理應用程式邏輯，並作為代理與後端的 Ollama 溝通.
* **Ollama 伺服器 (Ollama Server)**：實際運行大型語言模型並生成回應的核心後端。

## 設定各家模型以及 key

![alt tag](https://i.imgur.com/dwjBy8Z.png)

Ollama `http://host.docker.internal:11434` 的部份不用輸入 key

我這邊是使用 docker 的特殊網址 (可參考 [Docker container 內如何連接到本機 localhost 服務](https://github.com/twtrubiks/docker-tutorial?tab=readme-ov-file#docker-container-%E5%85%A7%E5%A6%82%E4%BD%95%E9%80%A3%E6%8E%A5%E5%88%B0%E6%9C%AC%E6%A9%9F-localhost-%E6%9C%8D%E5%8B%99))

![alt tag](https://i.imgur.com/bux4JoT.png)

各家連線網址(URL)

Gemini `https://generativelanguage.googleapis.com/v1beta/openai`

github `https://models.inference.ai.azure.com` (但這個有 bug, 文章後面使用 [Pipe Function](#pipe-function) 解決)

OpenAI `https://api.openai.com/v1`

OpenRouter `https://openrouter.ai/api/v1`

這邊可以下 TAG

![alt tag](https://i.imgur.com/7ZqFjqA.png)

這樣你就可以在這邊區分, 相同的 api 也可以放多個

![alt tag](https://i.imgur.com/piOv8rt.png)

## 功能簡介

因為他的功能實在太多了, 我只挑一些 [官方文件](https://docs.openwebui.com/features/)

🛡️ 細緻的權限與使用者群組

📱 行動版漸進式網頁應用程式 (PWA)。

✒️🔢 完整的 Markdown 和 LaTeX 支援。

🎤📹 免持語音/視訊通話。

🐍 原生 Python 函數呼叫工具：在工具工作區中透過內建的程式碼編輯器支援來增強您的 LLM。

📚 本地 RAG 整合：透過突破性的檢索增強生成 (RAG) 支援，深入探索未來聊天互動的可能性。

🔍 用於 RAG 的網頁搜尋：使用如 DuckDuckGo、TavilySearch、SearchApi ... 等供應商執行網頁搜尋，並將結果直接注入您的聊天體驗中。

TavilySearch 範例

![alt tag](https://i.imgur.com/NTVVn4r.png)

![alt tag](https://i.imgur.com/sWyJ9Jw.png)

🌐 網頁瀏覽功能：在 `#` 命令後加上 URL，即可將網站無縫整合到您的聊天體驗中。

🎨 圖片生成整合：透過 AUTOMATIC1111 API 或 ComfyUI (本地)，以及 OpenAI 的 DALL-E (外部) 等選項，無縫整合圖片生成功能。

設定方式, 模型我選 Gemini `imagen-3.0-generate-002`

url 輸入 `https://generativelanguage.googleapis.com/v1beta`

![alt tag](https://i.imgur.com/sOAFqrd.png)

實際效果

![alt tag](https://i.imgur.com/O4mr88e.png)

⚙️ 多模型對話：輕鬆同時與多個模型互動，利用它們各自的獨特優勢以獲得最佳回應。

🧩 Pipelines，它的核心作用是在使用者（透過 Open WebUI 前端）與最終的大型語言模型 (LLM) 後端之間，提供一個可以進行額外處理、過濾、增強或路由請求和回應的環節。

### 資料夾管理

[Organizing Conversations](https://docs.openwebui.com/features/chat-features/conversation-organization/)

![alt tag](https://i.imgur.com/AgJUoqd.png)

### 知識庫

[Knowledge](https://docs.openwebui.com/features/workspace/knowledge)

您可以在聊天中直接參考 Knowledge，使用 `#` + 知識庫名稱，以便在需要時隨時引入儲存的資料。

![alt tag](https://i.imgur.com/ojWplKE.png)

### Prompts

[Prompts](https://docs.openwebui.com/features/workspace/prompts)

可以自己定義 prompt 並且設定快捷按鍵, 然後在對話中使用 `/summarize`

![alt tag](https://i.imgur.com/Jd3RQnB.png)

### 筆記區

![alt tag](https://i.imgur.com/OU5oYHL.png)

### MermaidJS

支援 MermaidJS, 如果模型生成了 MermaidJS 語法，但沒有渲染出來，這通常表示程式碼中存在語法錯誤。

[MermaidJS Rendering Support in Open WebUI](https://docs.openwebui.com/features/code-execution/mermaid)

![alt tag](https://i.imgur.com/sL7SBEq.png)

### Artifacts

[What are Artifacts and how do I use them in Open WebUI?](https://docs.openwebui.com/features/code-execution/artifacts)

檢視： Artifacts 會顯示在主聊天介面右側的專用視窗。

編輯與迭代： 使用者可以透過在聊天中向 LLM 發出指令來修改 Artifact，更新會即時反映在 Artifact 視窗中。

版本控制： 每次編輯都會創建一個新版本，使用者可以透過版本選擇器在不同版本間切換和追蹤修改歷史。

操作： 提供複製內容、全螢幕檢視等功能。

![alt tag](https://i.imgur.com/nUHkISq.png)

![alt tag](https://i.imgur.com/36C9eEV.png)

## Pipe Function

[Pipe Function: Create Custom "Agents/Models"](https://docs.openwebui.com/features/plugin/functions/pipe)

因為預設的 azure 似乎格式不吻合, 這邊使用 Function 的方式來串 github 免費模型(有限制).

使用 [Open-WebUI-Functions](https://github.com/owndev/Open-WebUI-Functions) 去修改,

### 教學

申請 github key, 要記得打開設定 `models:read` permissions

![alt tag](https://cdn.imgpile.com/f/dVzqcsQ_xl.png)

你也可以在 github 上測試一下你的 Token

![alt tag](https://cdn.imgpile.com/f/8rws7M2_xl.png)

然後到函式 (function) 這邊直增加一個, 複製貼上我的 code [azure_ai_github.py](azure_ai_github.py)

![alt tag](https://cdn.imgpile.com/f/4AXsntn_xl.png)

填上你的資料

你的 github api key

AI Endpoint 可不填, 已經預設 `https://models.inference.ai.azure.com/chat/completions`

model 可不填, 預設會列出全部可用的

記得要打勾 `Use Predefined Azure Ai Models`

![alt tag](https://cdn.imgpile.com/f/tgyw2bm_xl.png)

全部的 github 模型可以使用的在這邊 [https://github.com/marketplace?type=models](https://github.com/marketplace?type=models)

接著你這邊就可以使用了

![alt tag](https://cdn.imgpile.com/f/CY3EVod_xl.png)

![alt tag](https://cdn.imgpile.com/f/M2nVI5K_xl.png)

如果你想看 Pipe Function 如何撰寫, 可以到 [官網](https://openwebui.com) 免費註冊看別人怎麼寫

![alt tag](https://cdn.imgpile.com/f/xXwcWuT_xl.png)

### MCP Support

[MCP Support](https://docs.openwebui.com/openapi-servers/mcp)

### Exporting and Importing Database

[Exporting and Importing Database](https://docs.openwebui.com/tutorials/database)

## 其他

一般來說, 除非你用了大量的 RAG 或是 Ollama, 否則 openwebui 其實不會很吃資源,

官網這裡也有說明如何降低資源 [reduce-ram-usage](https://docs.openwebui.com/tutorials/tips/reduce-ram-usage)

也就是盡量使用外部 API Key (例如 OpenAI, Anthropic, Google Cloud AI 等),

不要使用本地的服務 (但這樣感覺又有點本末倒置)

## Donation

文章都是我自己研究內化後原創，如果有幫助到您，也想鼓勵我的話，歡迎請我喝一杯咖啡 :laughing:

綠界科技ECPAY ( 不需註冊會員 )

![alt tag](https://payment.ecpay.com.tw/Upload/QRCode/201906/QRCode_672351b8-5ab3-42dd-9c7c-c24c3e6a10a0.png)

[贊助者付款](http://bit.ly/2F7Jrha)

歐付寶 ( 需註冊會員 )

![alt tag](https://i.imgur.com/LRct9xa.png)

[贊助者付款](https://payment.opay.tw/Broadcaster/Donate/9E47FDEF85ABE383A0F5FC6A218606F8)

## 贊助名單

[贊助名單](https://github.com/twtrubiks/Thank-you-for-donate)

## License

MIT license