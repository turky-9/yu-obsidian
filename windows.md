# プロセス(グループ毎)のメモリ使用量
``` powershell
Get-Process | Group-Object -Property ProcessName |Select-Object "Name", @{n="Memory"; e={($_.Group | Measure-Object "WS" -Sum).Sum}}
```