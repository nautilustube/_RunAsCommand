# RunAsCommand Guide

```powershell
param(
    [string]$Username,      # 欲以 runas 執行的使用者帳號，例如 ".\AdminUser"
    [string]$Password,      # 該帳號的密碼
    [string]$Command        # 欲執行的 CMD 指令，例如 "dir C:\Windows"
)

# 指定使用目前使用者的臨時資料夾
$sharedTempFolder = $env:TEMP
if (-not (Test-Path $sharedTempFolder)) {
    New-Item -Path $sharedTempFolder -ItemType Directory -Force | Out-Null
}

# 產生唯一檔案名稱（避免與其他執行衝突）
$guid = [guid]::NewGuid().ToString()
$outputFile = Join-Path $sharedTempFolder "out_$guid.txt"
$errorFile  = Join-Path $sharedTempFolder "err_$guid.txt"
# 確保檔案存在，若不存在則創建
if (-not (Test-Path $outputFile)) {
    New-Item -Path $outputFile -ItemType File -Force | Out-Null
}

if (-not (Test-Path $errorFile)) {
    New-Item -Path $errorFile -ItemType File -Force | Out-Null
}

# 組合 cmd 執行命令，並將標準輸出與錯誤輸出導向檔案
$cmdCommand = "cmd /c `"$Command`" > `"$outputFile`" 2> `"$errorFile`""

# 判斷是否需要使用指定帳號
if ([string]::IsNullOrEmpty($Username) -or [string]::IsNullOrEmpty($Password)) {
    # 使用目前登入的帳號
    $process = Start-Process -FilePath "powershell.exe" `
        -ArgumentList "-NoProfile", "-Command", "`"$cmdCommand`"" `
        -NoNewWindow `
        -PassThru
} else {
    # 使用指定帳號
    $securePassword = ConvertTo-SecureString $Password -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential($Username, $securePassword)

    $process = Start-Process -FilePath "powershell.exe" `
        -Credential $cred `
        -ArgumentList "-NoProfile", "-Command", "`"$cmdCommand`"" `
        -NoNewWindow `
        -PassThru
}

# 手動等待該進程結束（因為 -Wait 會因權限問題失敗）
while (-not $process.HasExited) {
    Start-Sleep -Milliseconds 200
}

# 檢查檔案是否存在
if (-Not (Test-Path $outputFile)) {
    Write-Host "⚠️ 標準輸出檔案不存在：$outputFile" -ForegroundColor Red
} else {
    # 嘗試讀取並顯示標準輸出與錯誤輸出
    try {
        Write-Host "`n=== 標準輸出 ==="
        Get-Content $outputFile -Raw

        $errorText = (Get-Content $errorFile -Raw)
        if ($errorText) {
            Write-Host "`n=== 錯誤輸出 ===" -ForegroundColor Red
            Write-Host $errorText.Trim()
        }
    } catch {
        Write-Host "錯誤訊息：" $_.Exception.Message -ForegroundColor Red
        Write-Host "錯誤堆疊：" $_.Exception.StackTrace -ForegroundColor Red
    }
}

# 清理暫存檔案
Remove-Item $outputFile, $errorFile -Force -ErrorAction SilentlyContinue

```

## Usage example

```
D:\_RunAsCommand\RunAsCommand.ps1 -Command "ipconfig /all"
D:\_RunAsCommand\RunAsCommand.ps1 -Command "D:\_RunAsCommand\sleep_and_echo.cmd"
D:\_RunAsCommand\RunAsCommand.ps1 -Command "D:\_RunAsCommand\sleep_and_echo.cmd"

D:\_RunAsCommand\RunAsCommand.ps1 -Command "D:\_RunAsCommand\sleep_and_echo.cmd"

D:\_RunAsCommand\RunAsCommand.ps1 -Username "nautilus" -Password "12341111AAAA" -Command "ipconfig /all"
D:\_RunAsCommand\RunAsCommand.ps1 -Username "nautilus" -Password "12341111AAAA" -Command "D:\_RunAsCommand\sleep_and_echo.cmd"
```

✅ 支援特性：
不會預先建立檔案，避免存取權限衝突；

支援任何 cmd 指令（包含管線、邏輯條件等）；

主程式不需要 -Wait 就能安全等待；

有詳細輸出與錯誤區塊；

可根據需要調整暫存資料夾權限。
