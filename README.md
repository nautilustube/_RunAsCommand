# RunAsCommand Guide

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
