# 工單與型別資料結構

## 使用者自訂資料結構

### 工單 Issues

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界
  - `issueSetId`: String, UUID/GUID - Foreign Key to IssueSets, Not Null, Index, 所屬工單集；工單可移至其他工單集
  - `issueTypeId`: String, UUID/GUID - Foreign Key to IssueTypeDefinitions, Not Null, 工單型別；決定欄位組配方、狀態流程與結案原因選項
  - `issueKey`: String - Not Null, 工單編號；產生後永不改，移動保留原號；同 Company 內唯一

**與欄位系統的關係**: 工單實體只持最小識別欄位，內容欄位不落本實體。
- 單值與多筆欄位的值存工單欄位單值與工單欄位多筆記錄兩實體
- 關聯欄位的值存工單關聯記錄，由關聯資料模型承載
- 工單帶哪些欄位，由工單型別的欄位組配方決定，工單不逐張宣告

### 工單型別定義 IssueTypeDefinitions

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；型別定義收在 Company 定義區，每 Company 一份
  - `name`: String - Not Null, 識別名稱，同 Company 內唯一，程式依此尋找型別
  - `label`: String - Not Null, 顯示名稱，可改，供團隊用語使用
  - `fieldSets`: Object - Not Null, JSON, 欄位組配方清單；記該型別採用的欄位組名稱，指向欄位組定義
  - `system`: Boolean - Not Null, 系統旗標；為真代表系統內建型別，不可刪除

### 流程狀態 WorkflowStates

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；流程定義收在 Company 定義區
  - `issueTypeId`: String, UUID/GUID - Foreign Key to IssueTypeDefinitions, Not Null, Index, 所屬工單型別；流程定義綁工單型別，一種型別一套
  - `name`: String - Not Null, 狀態名稱；與 `companyId`、`issueTypeId` 組成複合 Primary Key
  - `sortOrder`: Number - Not Null, 狀態排列位置；狀態清單與看板欄序依此排序
  - `isInitial`: Boolean - Not Null, 起始狀態旗標；新工單落在起始狀態
  - `isTerminal`: Boolean - Not Null, 終止狀態旗標；為真代表流程終點

### 流程轉換 WorkflowTransitions

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `issueTypeId`: String, UUID/GUID - Foreign Key to IssueTypeDefinitions, Not Null, Index, 所屬工單型別
  - `fromState`: String - Foreign Key to WorkflowStates, Not Null, 來源狀態名稱；指向同型別的流程狀態
  - `toState`: String - Foreign Key to WorkflowStates, Not Null, 目標狀態名稱；與 `companyId`、`issueTypeId`、`fromState` 組成複合 Primary Key
  - `requiredRole`: String | Null - Nullable, 轉換條件的執行者角色；Null 代表不限角色
  - `requiredFields`: Object - Not Null, JSON, 轉換必填欄位名稱清單；空清單代表無必填

### 結案原因選項 ResolutionOptions

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `issueTypeId`: String, UUID/GUID - Foreign Key to IssueTypeDefinitions, Not Null, Index, 所屬工單型別；選項清單隨工單型別各一份，與狀態流程同一層
  - `value`: String - Not Null, 選項值；與 `companyId`、`issueTypeId` 組成複合 Primary Key
  - `sortOrder`: Number - Not Null, 同型別內從一開始的選項排列位置
  - `system`: Boolean - Not Null, 系統種子旗標；為真代表由最小流程種子帶入，仍可改可刪

**與欄位系統的關係**: Status 與 Resolution 皆為選項值型別的內建欄位，兩欄分離。
- Status 的值域為同型別的流程狀態清單，回答工單走到哪一格
- Resolution 的值域為同型別的結案原因選項，回答工單的下場
- 兩欄回答不同問題，不可互相取代

---

## App 標準定義資料

### 標準流程狀態 StandardWorkflowStates

- **說明:**
  - 建立工單型別時帶入的最小流程狀態；載入為該型別的流程狀態，全部可改可刪可加
- **檔案:**
  - `assets/definitions/StandardWorkflowStates.json`
- **欄位:**
  - `name`: `String` - 狀態名稱，值為待處理、處理中、已關閉之一
  - `sortOrder`: `Number` - 狀態排列位置
  - `isInitial`: `Boolean` - 起始狀態旗標
  - `isTerminal`: `Boolean` - 終止狀態旗標
- 空白流程會卡死第一次使用 → 建立工單型別即帶入種子
- 三個狀態幾乎所有工作皆適用，需細分再自行追加

| name | sortOrder | isInitial | isTerminal |
|---|---|---|---|
| 待處理 | 1 | 是 | 否 |
| 處理中 | 2 | 否 | 否 |
| 已關閉 | 3 | 否 | 是 |

### 標準流程轉換 StandardWorkflowTransitions

- **說明:**
  - 最小流程的允許轉換；已關閉為終止狀態，轉入時必填結案原因
- **檔案:**
  - `assets/definitions/StandardWorkflowTransitions.json`
- **欄位:**
  - `fromState`: `String` - 來源狀態名稱
  - `toState`: `String` - 目標狀態名稱
  - `requiredFields`: `Array<String>` - 轉換必填欄位名稱，空陣列代表無必填
- 種子轉換不限執行者角色

| fromState | toState | requiredFields |
|---|---|---|
| 待處理 | 處理中 | 無 |
| 處理中 | 已關閉 | Resolution |

### 標準結案原因 StandardResolutionOptions

- **說明:**
  - 最小結案原因清單；載入為該型別的結案原因選項，可改可刪可加
- **檔案:**
  - `assets/definitions/StandardResolutionOptions.json`
- **欄位:**
  - `value`: `String` - 選項值，值為已完成、不做之一
  - `sortOrder`: `Number` - 從一開始的選項排列位置

| value | sortOrder |
|---|---|
| 已完成 | 1 |
| 不做 | 2 |

---

## 工單編號標準

- **儲存標準:**
  - 編號格式為前綴加流水號，例如 `LSPEC-142`
  - 前綴取自所屬工單集的 KEY
  - 流水號在該工單集內遞增，不跨工單集
  - 編號產生後永不改，工單移至其他工單集仍保留原號
  - 刪除的號不回收
- **計算與顯示標準:**
  - 移動後的編號可能與現況不符，工單另外顯示目前所屬的工單集
