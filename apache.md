# 設定
###### log ratate
``` ini
LoadModule log_config_module modules/mod_log_config.so
    #CustomLog "logs/access.log" common
    CustomLog "|bin/rotatelogs.exe -l logs/access_%Y-%m-%d.log 86400" common
```
