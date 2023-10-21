# docker run/create

docker run: コンテナを起動状態で作成  
docker create: コンテナを停止状態で作成

## docker run

httpd:latestイメージからconst1コンテナを作成

```sh
# docker run -d --name cont1 httpd:latest
```

## docker run -d -it

-d (--detach) オプションは、コンテナを作成後、コンテナプロセス(PID=1のプロセス)の標準入出力をコンソールからデタッチします。  
アタッチしたままコンテナを起動した場合は、デタッチキー (通常は Ctrl-P Ctrl-Q) を押すことでデタッチすることができます。  
-i (--interactive) オプションは、コンテナプロセスの標準入力を開いたままにします。  
-t (--tty) オプションは、コンテナプロセスに擬似TTYを割り当てます。

```sh
# docker run -d -it --name cont1 centos:7
54c2db8ce51774dedbf74d03f302257635b38518e7de24198b752c9bb07fccfd
# デタッチしているのでホストのプロンプトに戻る
# docker run -it --name cont2 centos:7
[root@f85deacb79f1 /]# デタッチしていないのでコンテナのプロンプトになる
[root@f85deacb79f1 /]# Ctrl-P Ctrl-Q    デタッチキーを押す
# ホストのプロンプトに戻る
```

## docker run -v

-v (--volume) オプションは、ホストのボリュームまたはディレクトリをコンテナに割り当てます。複数指定可能です。  
-v ホスト側:コンテナ側 で指定しますが、ホスト側が/で始まる名前の場合はディレクトリを指定します。

```sh
# mkdir -p /var/docker/cont1/htdocs
# echo '<div>Hello!!</div>' > /var/docker/cont1/htdocs/index.html
# docker run -d -p 8080:80 -v /var/docker/cont1/htdocs:/usr/local/apache2/htdocs --name cont1 httpd
# curl http://localhost:8080/
<div>Hello!!</div>
```

ホスト側が / で始まらない場合は名前付きのボリュームを指定します。

```sh
# docker volume create disk1
# docker run -d -it --name cont1 -v disk1:/disk1 centos:7
# docker volume ls
```

ホスト側を省略した場合は、名前無しのボリュームが自動的に作成されます。  
/var/lib/docker/volumes などの下に名前無しボリュームが作成されます。  
名前無しボリュームを作成した場合は、docker rm コマンドで -v オプションを指定しないと、名前無しボリュームが残ってしまいます。

```sh
# docker run -d -it --name cont1 -v /disk1 centos:7
# docker volume ls
```

## docker run -h

-h (--hostname) オプションは、コンテナのホスト名を指定します。

```sh
# docker run -d -it --name cont1 -h cont1 centos:7
```

## docker run -p

-p (--port) オプションは、ホストのポート番号とコンテナのポート番号をマッピングします。複数指定可能です。  
下記の例では、ホストの8080番ポートを、コンテナの80番ポートにマッピングしています。

```sh
# docker run -d --name cont1 -p 8080:80 httpd
# curl http://localhost:8080/
<html><body><h1>It works!</h1></body></html>
```

## docker run --add-host

--add-host はホスト名とIPアドレスをマッピングします。複数指定可能です。コンテナ内の /etc/hosts に書き込まれます。

```sh
# docker run -d -it --name cont1 --add-host host1:192.168.0.1 centos:7
# docker exec cont1 cat /etc/hosts | grep host1
192.168.0.1 host1
```

## docker run --net

--net オプションは、docker network コマンドで作成した Dockerネットワークにコンテナを接続します。  
--net を省略した場合はデフォルトで bridge ネットワークに接続します。

```sh
# docker network create network1 --subnet 192.168.0.0/24
# docker run -d -it --name cont1 --net network1 centos:7
```

## docker run --ip

--ip オプションは、IPアドレスを指定します。

```sh
# docker run -d -it --name cont1 --net network1 --ip 192.168.0.200 centos:7
```

## docker run -w

-w (--workdir) オプションは、コンテナ内のプロセスのワークディレクトリを指定します。

```sh
# docker run -d -it --name cont1 -w /opt/workdir centos:7
# docker exec cont1 pwd
/opt/workdir
```

## docker run -e

-e (--env) オプションは、コンテナの環境変数を設定します。複数指定可能です。

```sh
# docker run -d --name cont1 -e TZ=Aaia/Tokyo centos:7
# docker exec cont1 env | grep TZ
TZ=Asia/Tokyo
```

## docker run --env-file

--env-file オプションは、環境変数をファイルで指定します。

```sh
# cat ./envfile
TZ=Asia/Tokyo
LANG=en_US.UTF-8
# docker run -d --name cont1 --env-file ./envfile centos:7
```
