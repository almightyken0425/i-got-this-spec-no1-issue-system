# 組織管理與建單: ContainerManagementLogic

## readOrganization 讀取組織

- **執行:**
  - 讀取帳號所屬 Company 的 Team、Product、Mgmt 與 IssueSet
  - 保留各層名稱與歸屬
  - 依有效權限提供各層可執行操作
- **回傳:**
  - 組織樹與操作權限

---

## createContainer 建立組織項目

- **輸入:**
  - 項目種類、名稱與所屬容器
  - 建立 Mgmt 時提供初始 IssueSet 名稱與 KEY
  - 建立 IssueSet 時提供 KEY
- **執行:**
  - 名稱去除首尾空白後不得為空
  - 所屬容器必須存在於帳號所屬 Company
  - 建立 Team 或 Product 需要 orgAdmin
  - 建立 Mgmt 需要所屬 Product 的 canStructure
  - 建立 IssueSet 需要所屬 Mgmt 的 canStructure
  - KEY 遵循工單編號規則
  - Mgmt 與初始 IssueSet 同時建立
  - 初始 IssueSet 成為 Mgmt 的 containerIssueSetId
  - 驗證失敗時不保留部分新增資料
- **回傳:**
  - 新增項目或具體失敗原因

---

## renameContainer 重新命名組織項目

- **輸入:**
  - 目標項目與新名稱
- **執行:**
  - 目標必須存在於帳號所屬 Company
  - 名稱去除首尾空白後不得為空
  - Team 與 Product 改名需要 orgAdmin
  - Mgmt 與 IssueSet 改名需要所屬 Mgmt 的 canStructure
  - 僅更新名稱
  - 保留歸屬、KEY 與既有工單編號
- **回傳:**
  - 更新項目或具體失敗原因

---

## readIssueCreationOptions 讀取建單選項

- **輸入:**
  - 帳號與當前檢視
- **執行:**
  - 檢視必須屬於當前帳號
  - 依 sourceMgmtIds 取出來源管理域
  - 僅提供具 canCreate 與 canRead 的來源管理域
  - 提供各工單集的完整組織歸屬與 KEY
  - 提供 Company 的工單型別
- **回傳:**
  - 可選工單集與工單型別

---

## createWorkspaceIssue 建立指定工單

- **輸入:**
  - 標題、目標工單集與工單型別
- **執行:**
  - 未指定目標或型別時沿用預設工作區
  - 已指定的目標與型別必須存在於帳號所屬 Company
  - 無效選擇不得回退至預設值
  - 目標 Mgmt 必須具有 canCreate
  - 工單編號使用目標 IssueSet 的 KEY 與序列
  - 未指定狀態時使用所選型別的 isInitial 狀態
  - 工單與初始欄位值同時建立
- **回傳:**
  - 已建立工單或具體失敗原因
