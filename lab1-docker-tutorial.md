# Docker for Beginners - Linux

在本實驗中，我們將介紹一些基本的Docker命令和一個簡單的構建運行工作流程。我們首先運行一些簡單的容器，然後我們將使用Dockerfile來構建自定義應用程序。最後，我們將看看如何使用綁定掛載修改正在運行的容器，如果您正在使用Docker進行積極開發的話。

---

## Tasks：

* Task 0：環境建置
* Task 1：運行一些簡單的Docker容器
* Task 2：使用Docker打包並運行自定義應用程序
* Task 3：修改正在運行的網站

> 請盡可能自行輸入指令，增加印象

---

## Task 0：環境建置

您將需要以下所有內容完成，才能順利進行

### 建立雲端虛擬機

如果您尚未建立雲端虛擬機

* Google Cloud Platform

    登入 GCP 在 Google Cloud Shell 下輸入以下指令

```
bash <(curl -L http://tiny.cc/systex-devops01-install)
```

    完成後，請依照指示登入虛擬機，並換為 root user，後續示例皆以 root 設計

```
sudo su 
```

### 安裝 Docker

在您首次登入虛擬機，或尚未安裝 Docker 請依照以下指示執行

拷貝以下指令，貼上虛擬機命令列上執行 
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl start docker
```

為確認是否安裝完成，執行hello-world

```
docker run hello-world
```

### 從 GitHub clone Lab 原始碼

使用以下命令從GitHub clone Lab的原始碼。

```
cd ~
git clone https://github.com/bryanwu66/devops-hands-on.git
```

若您以上指令無法正確執行，請先確認是否已安裝 `git`

```
yum install -y git
```

---

## Task1：運行一些簡單的Docker容器

有不同的方法來使用容器。這些包括：

`SingleTask`：這可以是shell腳本或自定義應用程序。
`Interactively`：這將您連接到容器，類似於SSH到遠程服務器的方式。
`Background`：對於長期運行的服務，如網站和數據庫。

在本節中，您將嘗試其中的每個選項，並了解Docker如何管理工作負載。

---

## 在Alpine Linux容器中運行 SingleTask 容器 

在這一步中，我們將啟動一個新容器並告訴它執行`hostname`指令。容器將啟動，執行`hostname`指令，然後退出。

在Linux console中執行以下指令。

```
docker run alpine hostname
```

第一次執行時，會無法在本地端找到 `alpine:lastest` 映像檔。發生這種情況時， Docker 會自動至 Docker Hub 找尋，並拉回(`pull`)映像檔。

拉回映像檔後，會顯示容器內的 `hostname` (如下示例中的 `888e89a3b36b`)

```
 Unable to find image 'alpine:latest' locally
 latest: Pulling from library/alpine
 88286f41530e: Pull complete
 Digest: sha256:f006ecbb824d87947d0b51ab8488634bf69fe4094959d935c0c103f4820a417d
 Status: Downloaded newer image for alpine:latest
 888e89a3b36b
```
只要在容器內啟動的程序(process)仍在運行，Docker就會使容器保持運行。
在本例案中，`hostname` 是我們啟動的程序，當顯示輸出完成後便結束；這意味著容器也會跟著停止。
但是 Docker 預設情況下並不會刪除容器的資源，你可以觀察到容器仍然存在，但狀態顯示為 `Exited`

列出所有容器。

```
docker ps --all
```

請注意您的alpine 容器目前處於該`Exited`狀態。

```
 CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS            PORTS               NAMES
 888e89a3b36b        alpine              "hostname"          50 seconds ago      Exited (0) 49 seconds ago                       awesome_elion
```

`SingleTask`模式的容器非常好用，您可以構建一個映像檔，執行腳本配置某些內容 ___(例如Java的mvn、python的pip等等)___ 。之後任何人都可以執行容器(`docker run`)來執行該任務，他們不會碰到任何腳本配置或環境的問題。

---

## 在 Ubuntu 容器運行 Interactively 容器

您可以使用 Docker 運行不同版本 Linux，例如在下方示例中，我們將在CentOS Linux 主機上運行Ubuntu Linux容器

運行Docker容器並訪問其shell。

```
docker run --interactive --tty --rm ubuntu bash
```

在這個例子中，我們給Docker三個參數：

`--interactive` 指定使用互動式的 session。
`--tty` 分配一個偽tty。
`--rm` 告訴Docker在完成執行後，移除容器。

前兩個參數允許您與Docker容器進行互動，我們還告訴容器以`bash`作為主要 process（PID 1）。當容器啟動時，您將使用bash shell進入容器內  `root@<container id>:/# `

在容器中執行以下指令。

`ls /` 將列出容器中根目錄的內容，`ps aux`將顯示容器中正在運行的 process，`cat /etc/issue`將顯示容器正在運行的Linux版本，在本示例中為 `Ubuntu 18.04.2 LTS`

```
ls /
ps aux
cat /etc/issue
```

輸入 `exit` 退出 shell。這將終止bash process，導致容器退出 (`Exited`)。

```
exit
```

這時您可以再執行一次 `docker ps --all` 您會發現看不到Ubuntu容器。因為參數 `--rm` 會在容器退出時，同時執行刪除。

為了好玩，我們來檢查VM主機的版本。

```
cat /etc/issue
```

你應該看到：

```
\S
Kernel \r on an \m
```

請注意，我們的VM主機正在運作的是 CentOS Linux，但我們能夠運行不同發行版本的 Ubuntu 容器。

但是，Linux容器需要Docker主機運行Linux內核。例如，Linux容器無法直接在Windows主機上運行。Windows容器也是如此 - 它們需要在具有Windows內核的Docker主機上運行。

當您將自己的圖像放在一起時，交互式容器非常有用。您可以運行容器並驗證部署應用程序所需的所有步驟，並在Dockerfile中捕獲它們。

你可以 提交一個容器來製作一個圖像 - 但是你應該盡可能地避免使用它。使用可重複的Dockerfile來構建圖像要好得多。你很快就會看到。

---

## 運行後台MySQL容器

後台容器是您運行大多數應用程序的方式。這是一個使用MySQL的簡單示例。

使用以下命令運行新的MySQL容器。

```
docker run \
  --detach \
  --name mydb \
  -e MYSQL_ROOT_PASSWORD=my-secret-pw \
  mysql:latest
```

`--detach` 將在後台運行容器。
`--name` 將它命名為mydb。
`-e` 將使用環境變量來指定root密碼

由於MySQL映像在本地端不存在，Docker會自動至Docker Hub中拉取映像檔。

```
 Unable to find image 'mysql:latest' locallylatest: Pulling from library/mysql
 aa18ad1a0d33: Pull complete
 fdb8d83dece3: Pull complete
 75b6ce7b50d3: Pull complete
 ed1d0a3a64e4: Pull complete
 8eb36a82c85b: Pull complete
 41be6f1a1c40: Pull complete
 0e1b414eac71: Pull complete
 914c28654a91: Pull complete
 587693eb988c: Pull complete
 b183c3585729: Pull complete
 315e21657aa4: Pull complete
 Digest: sha256:0dc3dacb751ef46a6647234abdec2d47400f0dfbe77ab490b02bffdae57846ed
 Status: Downloaded newer image for mysql:latest
 41d6157c9f7d1529a6c922acb8167ca66f167119df0fe3d86964db6c0d7ba4e0
```

只要MySQL進程正在運行，Docker就會讓容器在後台運行。

列出正在 __運行中__ 的容器。

```
docker ps
```

請注意您的容器正在運行

```
 CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS            NAMES
 3f4e8da0caf7        mysql:latest        "docker-entrypoint..."   52 seconds ago      Up 51 seconds       3306/tcp            mydb
```

您可以使用幾個內建的Docker命令來檢查容器中發生的事情：`docker logs`和`docker top`。

```
docker logs mydb
```

這顯示了來自MySQL Docker容器的日誌。

```
Initializing database
2019-03-28T08:14:49.127365Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
2019-03-28T08:14:49.127466Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.15) initializing of server in progress as process 27
```

讓我們看一下容器內運行的 process。

```
docker top mydb
```

您應該看到MySQL守護程序（mysqld）正在容器中運行。

```
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
polkitd             4434                4418                1                   08:14               ?                   00:00:01            mysqld
```

雖然MySQL正在運行，但它在容器中是隔離的，因為沒有指定輸出的網絡端口。除非明確發布端口，否則網絡流量無法從主機到達容器。這在後續的項目中會進行。

使用列出MySQL版本docker container exec。

`docker exec`允許您在容器內執行指令。在這個例子中，我們將使用`docker exec`執行`mysql --user=root --password=$MYSQL_ROOT_PASSWORD --versionMySQL`容器內部的命令行等效命令。

```
docker exec -it mydb \
mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
```

您將看到MySQL版本號，以警告。

```
mysql: [Warning] Using a password on the command line interface can be insecure.
mysql  Ver 8.0.15 for Linux on x86_64 (MySQL Community Server - GPL)
```

您還可以使用`docker exec`連接到運行中的容器內執行 shell 。執行以下命令將為`sh`您提供MySQL容器內的交互式shell（）。

```
docker exec -it mydb sh
```

請注意，您的shell提示已更改。這是因為您的shell現在已連接到`sh`容器內。

讓我們通過再次運行相同的命令來檢查版本號，只是這次是在容器中的shell session中。

```
mysql --user=root --password=$MYSQL_ROOT_PASSWORD --version
```

輸出會與先前的相同

鍵入`exit`以退出交互式shell會話。

```
exit
```

---

## 課堂練習-01

執行以下指令，可以列出您目前本地端 repo 的所有映像檔

```
docker images
```

您的畫面輸出應該會類似以下所示

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              7bb2586065cd        28 hours ago        477MB
ubuntu              latest              94e814e2efa8        2 weeks ago         88.9MB
alpine              latest              5cb3aa00f899        2 weeks ago         5.53MB
```

* 要如何刪除本地端 repo 的映像檔？

> 提示：請試著看 docker help

---