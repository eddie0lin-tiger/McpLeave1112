# McpServer - 請假系統 MCP 伺服器

這是一個基於 Model Context Protocol (MCP) 的 ASP.NET Core 範例伺服器，提供請假管理相關的工具函數，並使用 Server-Sent Events (SSE) 作為傳輸協定。

## 專案概述

本專案提供的主要功能：
- 查詢員工今年剩餘可請假天數
- 查詢員工今年已請假天數
- 提交請假申請（示範性回傳結果）
- 取得當前日期時間（UTC+8）

## 技術架構

### 開發環境
- **.NET 10.0**
- **ASP.NET Core**
- **ModelContextProtocol** + **ModelContextProtocol.AspNetCore**（用於 MCP 與 SSE 整合）

### 通訊協定
- **Server-Sent Events (SSE)**

## 專案結構

```
McpServer/
├── Program.cs                      # 主程式與 MCP 工具實作 (LeaveRequestTool)
├── McpServer.csproj
├── McpServer.sln
├── appsettings.json
├── appsettings.Development.json
└── Properties/launchSettings.json
```

## 程式碼說明

### Program.cs

1. 建立 `WebApplication` 並設定日誌輸出（將 Trace 等級以上輸出到 stderr）：

```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Logging.AddConsole(consoleLogOptions =>
{
    consoleLogOptions.LogToStandardErrorThreshold = LogLevel.Trace;
});
```

2. 註冊 MCP 伺服器、啟用 HTTP/SSE 傳輸，並使用 `WithToolsFromAssembly()` 自動掃描程式集中標註的工具：

```csharp
builder.Services
    .AddMcpServer()
    .WithHttpTransport()
    .WithToolsFromAssembly();
```

3. 啟動應用並掛載 MCP middleware（`MapMcp()`）：

```csharp
var app = builder.Build();
app.MapMcp();
app.Run();
```

### 工具實作（`LeaveRequestTool`）

目前所有 MCP 工具皆直接定義在 `Program.cs` 的靜態類別 `LeaveRequestTool` 中，並由 `WithToolsFromAssembly()` 自動掃描註冊。主要工具如下：

- `getRemainingLeaveDays(string employeeName)`：取得今年剩餘可請假天數（示範性模擬回傳）。
- `getUsedLeaveDays(string employeeName)`：取得今年已請假的總天數（示範性模擬回傳）。
- `LeaveRequest(string 請假起始日期, int 天數, string 請假事由, string 代理人, string 請假者姓名)`：提交請假申請，若欄位齊全回傳請假摘要字串，否則回傳錯誤訊息。
- `GetCurrentDate()`：回傳目前時間（UTC+8，格式 `yyyy-MM-dd HH:mm:ss`）。

範例（伺服器內部模擬）：
- `david` → 5 天，`eric` → 6 天，其他 → 3 天（此為示範資料，實務應查詢資料庫）。

## 如何使用

### 1. 安裝相依套件

```bash
dotnet restore
```

### 2. 執行專案

```bash
dotnet run
```

預設會在 `http://localhost:5177` 啟動伺服器。

### 3. 設定 MCP 客戶端

在 `.vscode/mcp.json` 中設定連線資訊：

```json
{
  "servers": {
    "leave-request": {
      "type": "sse",
      "url": "https://your-server-url/sse"
    }
  }
}
```

### 4. 使用範例

透過支援 MCP 的客戶端可以自然語言呼叫工具，例如：

- "eric 今年還能請幾天假？" → 呼叫 `getRemainingLeaveDays("eric")` → 回傳示範值 6
- "eric 今年已請了幾天？" → 呼叫 `getUsedLeaveDays("eric")` → 回傳示範值 6
- "幫我從明天開始請 3 天假，事由是家中有事，代理人 John，請假者 Alice" → 呼叫 `LeaveRequest(...)`
- "現在時間是什麼？" → 呼叫 `GetCurrentDate()`

## MCP 註解說明

- `[McpServerToolType]`：標記類別為 MCP 工具容器
- `[McpServerTool]`：標記方法為 MCP 工具函數
- `[Description(...)]`：提供工具或參數的說明，幫助 AI 理解用途

## 開發注意事項

- 日誌：專案設定會將日誌輸出到標準錯誤串流 (stderr)，便於監控與除錯。
- 實務應用建議：將模擬邏輯改為資料庫查詢、加入驗證與授權、實作請假審核流程、加入錯誤處理與單元測試等。

## 相關資源

- [Model Context Protocol 官方文件](https://modelcontextprotocol.io/)
- [ASP.NET Core 文件](https://docs.microsoft.com/aspnet/core)
- [Server-Sent Events 規範](https://html.spec.whatwg.org/multipage/server-sent-events.html)

## 授權

本專案僅供學習與示範用途。
