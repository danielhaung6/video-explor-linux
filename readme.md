# Video Manager

以 FastAPI 與 Jinja2 建立的網頁媒體管理工具，支援本機媒體上傳、播放、YouTube 下載，以及 Google Drive、OneDrive、Dropbox 雲端整合。

## 功能

- 上傳多個媒體檔案，依影片、圖片、音樂分類瀏覽與刪除。
- 在瀏覽器播放影片與音樂、檢視圖片。
- 下載 YouTube 內容為 MP4 或 MP3。
- 指定影片縮圖與背景音樂。
- 連結雲端硬碟，瀏覽、播放、上傳檔案，或儲存至本機媒體庫。

| 類型 | 支援的副檔名 |
| --- | --- |
| 影片 | `.mp4`、`.mov`、`.mkv` |
| 圖片 | `.jpg`、`.jpeg`、`.png`、`.webp` |
| 音樂 | `.mp3` |

影片能否直接播放取決於瀏覽器與檔案的編碼格式。

## Linux 安裝

需要 Python 3.10 以上版本，以及可連網下載套件的環境。以下指令皆在專案根目錄執行。

### 使用系統 Python

Debian / Ubuntu 可先安裝 Python 與虛擬環境工具：

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-venv
```

建立虛擬環境並安裝相依套件：

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

`imageio-ffmpeg` 提供下載後處理所需的 FFmpeg；一般不需另外安裝系統版 FFmpeg。

### 此工作目錄已配置的環境

目前已建立 Python 3.12 虛擬環境並安裝套件，可直接使用下方啟動指令。

- `.venv/`：專案虛擬環境。
- `.runtime/python/`：獨立 Python 執行環境。
- `.runtime/bin/uv`：套件與 Python 環境管理工具。

以上目錄已加入 Git 忽略設定，不會隨原始碼提供。搬移專案到其他路徑或機器時，請重新建立環境。

如需在此環境重新安裝套件：

```bash
UV_CACHE_DIR="$PWD/.runtime/cache" .runtime/bin/uv pip install \
  --python .venv/bin/python -r requirements.txt
```

## 啟動

```bash
.venv/bin/python -m uvicorn main:app --host 127.0.0.1 --port 8000
```

啟動後開啟：

- [媒體庫](http://127.0.0.1:8000/)
- [雲端硬碟](http://127.0.0.1:8000/cloud)

按 `Ctrl+C` 停止服務。開發時可在指令末尾加上 `--reload`，修改程式後會自動重啟。

若需讓同一網路的其他裝置連線：

```bash
.venv/bin/python -m uvicorn main:app --host 0.0.0.0 --port 8000
```

其他裝置請使用 `http://<Linux 主機 IP>:8000`，並確認防火牆允許該連接埠。本機媒體庫沒有登入與權限控管，請僅在可信任的網路開放。

## 使用方式

1. 在首頁上傳媒體，或將檔案放入專案的 `media/` 目錄後重新整理頁面。
2. 在 YouTube 下載表單輸入網址，選擇 MP4 或 MP3。
3. 上傳圖片或音樂後，可在影片卡片指定縮圖與背景音樂。
4. 刪除操作會移除本機媒體檔案。

## 雲端硬碟設定

本機媒體功能不需要雲端憑證。若要連結雲端硬碟，請依 [CLOUD_SETUP.md](CLOUD_SETUP.md) 建立 OAuth 應用程式並設定 `cloud_config.json`。

首次開啟雲端頁面時會產生預設設定檔。填入對應服務的 `client_id` 與 `client_secret` 後，在頁面按「連結帳號」完成授權。

本機預設 OAuth callback 網址：

| 服務 | Callback URL |
| --- | --- |
| Google Drive | `http://127.0.0.1:8000/cloud/google/callback` |
| OneDrive | `http://127.0.0.1:8000/cloud/onedrive/callback` |
| Dropbox | `http://127.0.0.1:8000/cloud/dropbox/callback` |

請使用 `127.0.0.1` 開啟本機頁面，讓登入過程維持相同主機的 session。若更換服務位址或連接埠，也需要更新 OAuth 應用程式中的 callback URL。

雲端授權資料存於 `cloud_tokens.json`，依瀏覽器 session 分開儲存；本機媒體庫則由所有使用者共用。`cloud_config.json` 與 `cloud_tokens.json` 含敏感資料，請勿提交至版本控制。

## 測試

```bash
.venv/bin/python -m unittest discover -s tests -v
```

現有測試驗證頁面模板、表單欄位與前端行為約定，不需雲端憑證或網路。測試不包含實際 YouTube 下載或雲端 API 操作。

確認 FFmpeg 可用：

```bash
.venv/bin/python -c 'import subprocess; from imageio_ffmpeg import get_ffmpeg_exe; subprocess.run([get_ffmpeg_exe(), "-version"], check=True)'
```

## 專案結構

```text
main.py             FastAPI 應用程式與本機媒體操作
cloud/              雲端服務、OAuth 與設定管理
templates/          HTML 模板、CSS 與 JavaScript
media/              本機媒體檔案
thumbnail_map.py    縮圖對應資料管理
tests/              模板與前端回歸測試
requirements.txt    Python 相依套件
CLOUD_SETUP.md      雲端 OAuth 設定說明
```

影片背景音樂與縮圖對應分別存於 `bgm_map.json` 與 `thumbnail_map.json`。備份時請一併保存媒體及需要的設定檔。

## 常見問題

- **找不到 `python3` 或無法建立虛擬環境**：先安裝 `python3` 與 `python3-venv`，或使用本工作目錄已建立的 `.venv/bin/python`。
- **出現 `ModuleNotFoundError`**：確認使用 `.venv/bin/python` 啟動，並已安裝 `requirements.txt`。
- **8000 埠已被占用**：將啟動指令改為 `--port 8001`，並同步調整雲端 callback 設定。
- **雲端顯示尚未設定**：檢查 `cloud_config.json` 的對應服務憑證，詳細步驟見雲端設定文件。
- **YouTube 下載失敗**：確認網址、網路與 FFmpeg 可用；部分內容受登入、地區或平台限制。可依上方安裝方式加上 `--upgrade` 更新套件後再試。
