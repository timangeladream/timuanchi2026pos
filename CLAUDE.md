# TimuAnchi POS 銷售系統

## 專案概要
Firebase + 純 HTML/JS 的單頁 POS 應用，用於珠寶飾品銷售（市集、門市）。

## 技術架構
- **主檔案：** `index.html`（所有功能集中）、`customer.html`（顧客顯示螢幕）
- **後端：** Firebase Auth（帳號）+ Firestore（資料庫）
- **函式庫：** ZXing（QR Code 掃碼）、SheetJS/xlsx（Excel 匯入匯出）、html2pdf.js（收據轉 PDF）
- **PWA：** manifest.json + sw.js

## Firestore 資料結構
| Collection | 用途 |
|---|---|
| `settings/system` | 商品、分類、折扣、贈品、加購品、系統設定 |
| `sales` | 銷售記錄 |
| `customers` | 顧客資料 |
| `stockMovements` | 庫存異動記錄 |
| `users` | 帳號（admin / cashier） |
| `activeOrders` | 進行中訂單（供顧客螢幕即時同步） |

## 重要全域變數
- `products` — 所有商品的扁平陣列
- `productData` — 巢狀結構 `{brand: {mainCat: {subCat: [products]}}}`
- `categories` — 品牌/大分類/小分類（獨立儲存）
- `brandOrder` — 品牌排序陣列（Firestore 不保證物件 key 順序）
- `orderItems` — 當前訂單項目
- `discounts` — 當前已套用的折扣
- `discountButtons` — 折扣設定（六種類型）
- `giftRules` — 贈品規則
- `addOnItems` — 加購項目
- `salesHistory` — 銷售記錄（本次 session）

## 商品規格系統
- `variants` — `{規格名稱: [選項陣列]}`
- `variantKeyOrder` — 規格名稱順序陣列（確保 Firestore 讀寫後順序一致）
- `variantPrices` — `{comboKey: 價格}`，comboKey 格式為 `選項1,選項2`
- `variantStock` — `{comboKey: 庫存}`
- `variantSkus` — `{comboKey: 貨號}`

## 折扣類型
1. `percent` — 全單百分比折扣
2. `fixed` — 固定金額折扣
3. `minAmountPercent` — 滿額百分比折扣
4. `minAmountFixed` — 滿額固定金額折扣
5. `buyNPercent` — 任選N件百分比折扣
6. `buyNFixed` — 任選N件固定金額折扣

## 開發注意事項
- Firestore 不允許空字串作為 field key → 使用 `encodeProductDataKeys()` / `decodeProductDataKeys()` 轉換
- 商品 ID 使用 UUID（`generateProductId()`）避免碰撞
- 彈窗層級使用 `bringModalToFront()` 動態管理
- 庫存異動透過 `logStockMovement()` 統一記錄
- `normalizeVariants()` 確保規格選擇的 key 順序一致

---

## 開發歷程統整

### 一、匯入匯出功能修復
- 修正重複 HTML input ID、單一品牌匯出欄位缺失
- 商品 ID 改用 UUID 解決碰撞問題
- 修正 Firestore 不允許空字串作為 key 的問題
- 修正匯入後資料不同步需重整的問題

### 二、商品/加購品/贈品管理
- 修正刪除商品時誤刪分類、品牌的問題
- 各管理功能獨立選單頁面
- 上架/下架功能（`status: 'inactive'`）
- 加購項目「只更新價格和庫存」匯入功能

### 三、彈窗層級與互動體驗
- 動態彈窗層級系統 `bringModalToFront()`
- 修正編輯彈窗誤關背景管理列表的問題
- 修正選單未正確收合的 bug

### 四、庫存異動記錄系統
- `logStockMovement()` 記錄所有庫存變動
- 補貨管理頁面（商品/加購品/贈品三分頁）
- 「已刪除」標示與篩選功能
- 單品庫存異動查詢

### 五、銷售頁與訂單功能
- 單規格商品庫存上限控管
- 贈品數量可手動增減（含庫存上限檢查）
- 訂單明細標題數量統計
- 銷售員姓名、時間格式修正

### 六、優惠折扣系統
- 六種折扣類型完整支援
- 「自動套用最優折扣組合」開關（動態規劃演算法）
- 手動模式支援疊加多個「任選N件」折扣
- 折扣套用次數與顯示同步修正

### 七、顧客管理與雙螢幕同步
- 顧客詳情頁（消費記錄可點擊查看明細）
- QR Code 複製網址功能
- customer.html 顧客端同步修正（規格順序、貨號、數量統計、折扣次數標籤）

### 八、清除資料與權限
- 六種清除功能統一權限檢查（僅 admin）
- 新增「清除庫存異動紀錄」

### 九、系統選單與介面調整
- 選單項目順序、命名統一
- 管理彈窗標題與按鈕位置統一
- 匯出/清除資料彈窗文字、顏色、順序統一
