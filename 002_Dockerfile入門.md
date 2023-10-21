# Dockerfile入門

## Dockerfileとは

Dockerfileにより、ベースとなるイメージを元に必要なパッケージやファイルをインストールし、新たなイメージを作成する作業を自動化することができます。  
(例)centos:7 イメージをベースとして、Apache (httpd) をインストールし、新たなイメージを作成

```dockerfile
# mkdir ./work
# cd ./work
# cat > Dockerfile <<EOF
FROM centos:7
RUN yum install -y httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
EOF
# docker build -t my-app:latest .
# docker images
```

## Dockerfileのコマンド一覧

FROM ... ベースとなるイメージ  
RUN ... docker build 時に実行するコマンド  
CMD ... docker run 時に実行するコマンド  
ENTRYPOINT ... docker run 時に実行するコマンド  
MAINTAINER ... 作者情報  
LABEL ... ラベル情報(メタデータ)  
EXPOSE ... 公開ポート番号  
ENV ... 環境変数  
ARG ... 一時変数  
COPY ... ホストからコンテナへのファイルコピー  
ADD ... ホストからコンテナへのファイルコピー  
VOLUME ... ボリューム  
USER ... 実行ユーザ  
SHELL ... シェル指定  
WORKDIR ... ワークディレクトリ  
ONBUILD ... ビルド時に実行するコマンド  
STOPSIGNAL ... コンテナ終了時に送信されるシグナル  
HEALTHCHECK ... ヘルスチェック

## FROM

FROMはベースとなるイメージを指定します。Dockerfileの先頭に必須です。

```dockerfile
FROM centos:7
```

タグを省略すると:laltestが指定されたとみなされます。

```dockerfile
FROM centos
```

タグの代わりに@でダイジェストを指定することもできます。

```dockerfile
FROM centos@sha256:a799dd8a2ded4a83484bbae769d97655392b3f86533ceb7dd96bbac929809f3c
```

## RUN

ビルド時に実行すｒコマンドを指定します。  
コマンドの呼び出しは「シェル形式」と「exec形式」の2つの形式があります。  
シェル形式の場合は、「/bin/sh -c」にコマンドが渡され、echoなどのシェルコマンド実行や環境変数の展開が可能となります。  
exec形式は、直接そのコマンドが実行されます。

```dockerfile
# シェル形式
RUN yum install -y httpd

# exec形式
RUN ["yum", "install", "-y", "httpd"]
```

RUN が実行される度にイメージのレイヤが増えるため、コマンドを && や ; で連結し、複数のコマンドをひとつの RUN にまとめて記述することが推奨されています。

```dockerfile
RUN yum update -y && \
    yum install -y httpd && \
    wget http://download.exeample.com/centos7/app-1.2.3.tar.gz
```

## CMD

docker run時に実行するコマンド、またはdocker run時に実行するENTRYPOINTコマンドの引数を指定します。  
CMDもまた、シェル形式と exec形式があります。  
シェル形式の場合、シェル機能を利用できる反面、PID=1のプロセスが/bin/sh -cとなり、docker stopを行ってもSIGTERMを受け取るのは/bin/shとなり、コマンドがシグナルを受け取ることができないデメリットがあります。

```dockerfile
# exec形式(推奨)
CMD ["/usr/sbin/httpd", "-DGOREGROUND"]

# シェル形式
CMD /usr/sbin/httpd -DFOREGROUND

# ENTRYPOINTの引数
CMD ["param1", "param2"]
```

## ENTRYPOINT

docker run時に実行するコマンドをシェル形式、またはexec形式で指定します。  
CMDと似てますが、「--entrypoint オプション > ENTRYPOINT > run引数 > CMD」の優先度があります。

```dockerfile
# exec形式
ENTRYPOINT ["/bin/cmd", "arg1", "arg2"]

# シェル形式
ENTRYPOINT /bin/cmd arg1 arg2
```

ENTRYPOINTとCMDがexec形式で指定された場合、docker runを引数無しで実行するとENTRYPOINT＋CMDが、引数有りで実行するとENTRYPOINT＋run引数が実行されます。

```dockerfile
ENTRYPOINT ["/bin/cmd", "arg1", "arg2"]
CMD ["arg3", "arg4"]
```

```dockerfile
#/bin/cmd arg1 arg2 arg3 arg4 が実行される
docker run image1

# /bin/cmd arg1 arg2 arg5 arg6 が実行される
docker run image1 arg5 arg6
```

CMD のみの場合は、CMD の第一引数がコマンドとみなされます。docker run の引数により上書きされます。

```dockerfile
CMD ["/bin/cmd", "arg1", "arg2"]
```

```dockerfile
# /bin/cmd arg1 arg2 が実行される
docker run image1

# /bin/cmd2 arg3 arg4 が実行される
docker run image1 /bin/cmd2 arg3 arg4
```

ENTRYPOINT のみの場合は、ENTRYPOINT＋run引数が実行されます。

```dockerfile
ENTRYPOINT ["/bin/cmd", "arg1", "arg2"]
```

```dockerfile
# /bin/cmd arg1 arg2 が実行される
docker run image1

#/bin/cmd arg1 arg2 arg3 arg4 が実行される
docker run image1 arg3 arg4
```

アルゴリズムとしては  
① --entrypointオプションが指定されていれば、--entrypointオプション値でENTRYPOINT配列を置換する。  
② docker runに引数が指定されていれば、run引数でCMD配列を置換する。  
③ ENTRYPOINT配列にCMD配列をアペンドする。  
④ できあがった配列をコマンド＋引数として実行する。  
といった流れになります。

## EXPOSE

指定したポート番号をコンテナが公開することをDockerに伝えます。

```dockerfile
EXPOSE 80
```

## ENV

環境変数を指定します。レイヤが増えないように可能な限り1行で記述します。

```dockerfile
ENV DB_HOST="192.168.2.201" \
    DB_PORT="3306" \
    DB_USER="myapp" \
    DB_PASSWD="ZbGc7#adG87GBfVC" \
    DB_DATABASE="sample"
```

## ARG

Dockerfile内で使用できる変数を指定します。  
ENVによる環境変数がコマンドやコマンドのサブプロセスに引き継がれるのに対し、ARGによる変数はDockerfileの中のみで使用できます。

```dockerfile
ARG user=unknown
RUN echo $user
=> unknown
```

docker buildの--build-argオプションで値を指定することも可能です。  
この場合、unknownは、--build-argオプションが指定されなかった場合のデフォルト値になります。

```dockerfile
# docker build -t myapp --build-arg user=tanaka .

ARG user=unknown
RUN echo $user
=> tanaka
```

## COPY

ホストからコンテナイメージにファイルやディレクトリをコピーします。

```dockerfile
# ファイルをファイルにコピー
COPY ./file1.conf /etc/file .conf

# ファイルをディレクトリ配下にコピー
COPY ./file2.conf /etc

# ディレクトリをディレクトリにコピー
COPY ./my-app /opt/my-app
```

## ADD

ホストからコンテナイメージにファイルやディレクトリをコピーします。  
COPY とは異なり、転送元に .tar.gz などの圧縮ファイルを指定すると自動的に展開してコピーすることができます。  
また、http:// や https:// で始まる URL を指定することもできます。

```dockerfile
ADD ./my-app.tar.gz /opt
ADD http://www.example.com/file1.txt /etc/file1.txt
```

## VOLUME

コンテナ起動時に永続ボリュームを割り当てます。  
永続ボリュームはホスト側の/var/lib/docker/volumesなどに作成されます。  
docker runの-v オプションでは「-v ボリューム名:マウントポイント」でボリューム名を指定することができました。  
VOLUMEではマウントポイントしか指定することはできず、名前付きボリュームを割り当てることはできません。  
VOLUMEでマウントしたボリュームは、dfコマンドに-aオプションをつけないと表示されないことがあります。  
※docker runの-vより--mountの方が推奨になっている

```dockerfile
VOLUME /var/disk1 /var/disk2 /var/disk3
```

## WORKDIR

RUN, CMD, ENTRYPOINT, COPY, ADD, docker run, exec で実行するコンテナプロセスのワークディレクトリを指定します。

```dockerfile
WORKDIR /tmp
```

複数記述すると直前の WORKDIR が有効となります。

```dockerfile
WORKDIR /tmp
RUN pwd
=> /tmp

WORKDIR /var
RUN pwd
=> /var
```

環境変数を展開することもできます。

```dockerfile
ENV BASE_DIR=/opt/myapp
WORKDIR $BASE_DIR/tmp
```
