---
layout: article
title: TFTP
key: 20181209
tags: 嵌入式 
---

在開發嵌入式系統裡，我們在開發的過程中，往往會接觸一些傳輸或是連接的協定，
而TFTP很明顯的就是其中特別常見的一種，以下我們將會稍微簡介一下TFTP的特性，
與一步步帶領讀者如何在Ubuntu上架設自己的tftp server。


<!--more-->

## 1. 簡介
小型檔案傳輸協定TFTP(Trivial File Transfer Protocol, TFTP)，，是於1981年基於檔案傳輸協定（FTP）定義的簡化版協定，
因為它傳輸的方式簡單且需要的資源不多，故利用來在可以連網嵌入式系統上來做網路開機或是傳輸映像檔(image)開機，其傳輸有下列幾個特性：
* 使用UDP(port=69)來當作其傳輸的協定
* 不能列出目錄內容
* 無驗證或加密機制
* Client可以被於在遠端伺服器上讀取或寫入檔案
* 一般用於區域網路(LAN)之上

## 2. 如何在Ubuntu架設自己的tftp server
### 1.安裝tftp server(tftpd-hpa)與tftp client(tftp-hpa)
這邊我們安裝的是(-hpa)的版本，相較於無後綴的版本它修掉了一些奇怪的bug

```bash
$ sudo apt-get install tftp-hpa tftpd-hpa
```

### 2. 修改組態文件
1.  server 組態(/etc/default/tftpd-hpa)

```bash
# /etc/default/tftpd-hpa

TFTP_USERNAME="nobody" # 運行 tftpd 的user，預設值為nobody 
TFTP_DIRECTORY="/tftpboot" # tftp server分享的根資料夾路徑
TFTP_ADDRESS=":69" # 
TFTP_OPTIONS="-l -c -s
```

### 3. 建立所需的資料夾與更動權限
```bash
$ sudo mkdir /tftpboot
$ sudo chmod -R 777 /tftpboot
$ sudo chown -R nobody:nogroup /tftpboot
```

### 4. 開啟tftp的服務
```bash
$ sudo service tftpd-hpa restart
```

### 5. 測試tftp
1. 檢查服務有無開啟
可以透過下面兩種方式來做檢查：
* 檢查process有沒有運作
  ```bash
  $ ps aux |grep tftp
  root     15480  0.0  0.0  15352   152 ?        Ss    5月05   0:00 /usr/sbin/in.tftpd --listen --user nobody --address :69 -l -c -s /tftpboot
  tobygao  20002  0.0  0.0  21544  1088 pts/7    S+   00:34   0:00 grep --color=auto tftp
   ```
* 檢查網路的狀態
  ```
  $ netstat -a | grep tftp
  udp        0      0 0.0.0.0:tftp            0.0.0.0:*                          
  udp6       0      0 [::]:tftp               [::]:*   
  ```

2. 利用tftp client來上下傳檔案
   ```
   $ tftp tftp 127.0.0.1
   tftp> get downloadTest
   tftp> put uploadTest
   ```


---
參考資料：

[Trivial File Transfer Protocol(wiki)](https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol)

[Ubuntu community:TFTP](https://help.ubuntu.com/community/TFTP)

[TFTP](https://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.sdk5.v15.0.0%2Fiot_sdk_app_tftp.html)

[Ubuntu / Debian Linux: Install and Setup TFTPD Server](https://www.cyberciti.biz/faq/install-configure-tftp-server-ubuntu-debian-howto/)

[使用TFTP进行文件传输](https://www.jianshu.com/p/b6afddb05cdc)
