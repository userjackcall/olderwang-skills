# Pexels Media Sourcing

從 Pexels API 搜尋、下載高品質免費圖片與影片，用於設計、佔位圖或內容素材。

## 這是什麼？

這個 skill 讓 AI 能直接透過 Pexels API 搜尋圖庫、下載素材，並自動建立附帶的**中繼資料檔（sidecar metadata）**，方便你追溯來源與授權資訊。

所有下載的素材均為 **Pexels 授權**，可免費使用於個人與商業專案。

## 前置準備

需要設定 `PEXELS_API_KEY` 環境變數。可至 [Pexels API](https://www.pexels.com/api/) 免費申請。

```bash
# 確認 API Key 已設定
echo $PEXELS_API_KEY
```

## 怎麼用

### 搜尋圖片

```bash
# 基本搜尋
curl -H "Authorization: $PEXELS_API_KEY" \
  "https://api.pexels.com/v1/search?query=office+workspace&per_page=5"
```

### 搜尋影片

```bash
curl -H "Authorization: $PEXELS_API_KEY" \
  "https://api.pexels.com/videos/search?query=nature&min_width=1920&per_page=5"
```

### 熱門／精選

| 類型 | API 端點 |
|------|----------|
| 精選照片（每小時更新） | `GET /v1/curated` |
| 熱門影片 | `GET /videos/popular` |
| 精選收藏集 | `GET /v1/collections/featured` |

### 可用篩選條件

**圖片搜尋參數：**

| 參數 | 說明 | 範例值 |
|------|------|--------|
| `query` | 搜尋關鍵字（必填） | `mountain sunset` |
| `orientation` | 方向篩選 | `landscape`, `portrait`, `square` |
| `size` | 尺寸等級 | `large` (24MP), `medium` (12MP), `small` (4MP) |
| `color` | 主色調 | `blue`, `green`, `#ffffff` |
| `locale` | 搜尋語言 | `zh-TW`, `en-US`, `ja-JP` |

**影片額外參數：**

| 參數 | 說明 |
|------|------|
| `min_width` / `min_height` | 最小解析度 |
| `min_duration` / `max_duration` | 最短／最長秒數 |

## 圖片可用尺寸

| 尺寸 | 說明 |
|------|------|
| `original` | 原始上傳解析度 |
| `large2x` | 寬 940px（高度 2x） |
| `large` | 寬 940px |
| `medium` | 高 350px |
| `small` | 高 130px |
| `portrait` | 800×1200 |
| `landscape` | 1200×627 |
| `tiny` | 280×200 |

## 中繼資料檔（Sidecar Metadata）

**每個下載的檔案都會自動產生一個 `.meta.json` 附檔**，內容包含：

- 來源（Pexels）
- 攝影師名稱與連結
- 下載尺寸與解析度
- 授權條款
- 完整 API 回應原始資料

範例：下載 `sunset.jpg` 後，自動產生 `sunset.jpg.meta.json`。

## 檔案結構

```
pexels-media/
└── SKILL.md        # 技能主說明（API 端點、使用方式、完整 curl 範例）
```

## 速率限制

- 預設：每小時 200 次請求、每月 20,000 次
- 可在回應標頭查看剩餘次數：`X-Ratelimit-Remaining`

## 授權與姓名標示

Pexels 授權允許免費使用，無需姓名標示即可用於個人與商業用途。但建議：
- 一般使用：`Photo by John Doe on Pexels`
- 附連結：`Photo by John Doe (https://www.pexels.com/@johndoe) on Pexels (https://www.pexels.com)`

中繼資料檔中已自動包含標準的標示文字與 HTML 格式。

## 觸發時機

- 需要找免費商用圖片／影片素材
- 設計中需要高品質佔位圖
- 需要特定主題、色調或方向的圖庫照片
