# Systemd add service
```sh
cd /etc/systemd/system/
#add xxx.service below
```
### service structure
```sh
[Unit]
#service name 
Description=ipmi exporter service


After=network.target syslog.target

[Service]
Type=simple
WorkingDirectory=/tmp

#
# 服務終止時自動重新啟動
Restart=always

RestartSec=1
User=root
SyslogIdentifier=ipmi-exporter
ExecStart=/usr/bin/ipmi_exporter
# 行程類型
Type=simple

# 啟動服務指令
ExecStart=/opt/your_command

# 服務行程 PID（通常配合 forking 的服務使用）
# PIDFile=/run/your_server.pid

# 啟動服務前，執行的指令
# ExecStartPre=/opt/your_command

# 啟動服務後，執行的指令
# ExecStartPost=/opt/your_command

# 停止服務指令
# ExecStop=/opt/your_command

# 停止服務後，執行的指令
# ExecStopPost=/opt/your_command

# 重新載入服務指令
# ExecReload=/opt/your_command



# 重新啟動時間格時間（預設為 100ms）
# RestartSec=3s

# 啟動服務逾時秒數
# TimeoutStartSec=3s

# 停止服務逾時秒數
# TimeoutStopSec=3s

# 執行時的工作目錄
# WorkingDirectory=/opt/your_folder

# 執行服務的使用者（名稱或 ID 皆可）
# User=myuser

# 執行服務的群組（名稱或 ID 皆可）
# User=mygroup

# 環境變數設定
# Environment="VAR1=word1 word2" VAR2=word3 "VAR3=$word 5 6"

# 服務輸出訊息導向設定
# StandardOutput=syslog

# 服務錯誤訊息導向設定
# StandardError=syslog

# 設定服務在 Syslog 中的名稱
# SyslogIdentifier=your-server


[Install]
WantedBy=multi-user.target
```
### systemctl command

```sh
#list all services
sudo systemctl list-units
# status service
systemctl status hello.service
systemctl restart hello.service
# systemctl所指定查看服務的狀態訊息會有以下這幾種輸出：
# loaded，此單位（unit）的設定檔已經被處理了。
# active(running)，運行一個或多個的程序prcoess。
# active(exited)，已經成功地執行一次性的設定。
# active(waiting)，正在運行但是正在等待一個事件。
# inactive，尚未運行。
# enabled，在開機的時候，這個服務會自動帶起來運行。
# disabled，在開機的時候，這個服務不會自動的帶起來運行。
# static，無法被設定成開機自動啟動（enabled），但是可能可以被自動的啟用此單位unit。
```


### Training 
---
**1. 
Create the "Hello BMC" systemd service in openbmc, which prints out "Hello BMC" via shell script, and use the systemctl command to check the status of the service and restart it.**

**2. 
Executing the "Hello BMC" systemd service before systemd-networkd.service.**
.sh
```sh
#bin/bash
echo "Hello BMC"
```

/lib/systemd/system/
add service to change phosphor state manager
# bb file as task 



x.service
```sh
[Unit]
Description=Hello BMC service
Before=systemd-networkd.service
[Service]

[Install]
WantedBy=multi-user.target
```