# docker-openrtm

OpenRTM-aistのUbuntu用最新バージョンがインストールされているDockerイメージを作成するためのDockerfileを定義

## How to build

ビルドと同時にDocker Hubへアップロードするため、事前にブラウザ[Docker hub](https://hub.docker.com/) とコンソール画面からログインしておく

- コンソール画面の場合
```shell
$ docker login -u ユーザ名
```

OpenRTM 2.1.0版用(Ubuntu24.04、22.04)で、ビルドと同時にDocker hubへアップロードする手順は以下の通り

### C++, Python, Java

```shell
Ubuntu24.04用の手順

$ git clone https://github.com/OpenRTM/docker-openrtm
$ cd docker-openrtm

# c++用
$ docker buildx build --no-cache --pull -f ubuntu_2404/cxx/Dockerfile -t openrtm/cxx:u24.04-2.1.0 --platform linux/amd64,linux/arm64 --push .

# python用
$ docker buildx build --no-cache --pull -f ubuntu_2404/python/Dockerfile -t openrtm/cxx:u24.04-2.1.0 --platform linux/amd64,linux/arm64 --push .

# java用
$ docker buildx build --no-cache --pull -f ubuntu_2404/java/Dockerfile -t openrtm/cxx:u24.04-2.1.0 --platform linux/amd64,linux/arm64 --push .
```

### rtshell

rtshell用のビルド時、Python3で以下に示す既知の相性バグが発生した
- QEMUエミュレーション環境下において特定の命令で高確率でセグメンテーションファルト（-11）を起こしてクラッシュする

これの回避のため、--network=host を指定してビルドする

```shell
Ubuntu24.04用の手順

$ docker buildx build --network=host --no-cache --pull -f ubuntu_2404/rtshell/Dockerfile -t openrtm/rtshell:u24.04-4.2.10 --platform linux/amd64,linux/arm64 --push .
```

### imageprocessing

DockerfileではOpenCVをソースからインストールし、ImageProceesingをビルドしてdebパッケージを生成し、これを使ってインストールする処理を定義している

Macのarm64環境でamd64用ビルドを行うと、セグメンテーションファルトを起こすため、arm64とamd64はそれぞれのネイティブ環境で別々にビルドする

```shell
Ubuntu24.04用の手順

# arm64用はタグ名に -arm64 を付けてビルドする
$ docker buildx build --no-cache --pull -f ubuntu_2404/imageprocessing/Dockerfile -t openrtm/imageprocessing:u24.04-2.1.0-arm64 --platform linux/arm64 --push .

# amd64用はタグ名に -amd64 を付けてビルドする
$ docker buildx build --no-cache --pull -f ubuntu_2404/imageprocessing/Dockerfile -t openrtm/imageprocessing:u24.04-2.1.0-amd64 --platform linux/amd64 --push .
```

Docker Hub上にすでに -amd64 と -arm64 のイメージが存在している状態で、これを一つにまとめる
```shell
Ubuntu24.04用の手順

# 1. 適当な空ディレクトリを作成して移動
mkdir build-tmp && cd build-tmp

# 2. Dockerfileの代わりに「Docker Hub上の2つのイメージをただ吸い込んで1つにまとめる」コマンドを実行
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t openrtm/imageprocessing:u24.04-2.1.0 \
  --push - <<EOF
FROM openrtm/imageprocessing:u24.04-2.1.0-\$TARGETARCH
EOF

# 3. 終わったら空ディレクトリを削除
cd .. && rm -rf build-tmp
```
- \$TARGETARCH という変数を使うことで、Docker側が自動的に「amd64のときは -amd64 を、arm64のときは -arm64 をDocker Hubから引っ張ってきて、
1つのマルチアーキテクチャタグ（u24.04-2.1.0）にまとめて直にPushする」という処理をノービルド（一瞬）で行ってくれる

Docker Hubで１つのイメージにまとまっていることを確認できたら、ブラウザで個別タグを削除しておく
```shell
# 1. 対象のリポジトリ（openrtm/imageprocessing）を開く
# 2. 「Tags」 タブをクリック
# 3. タグの一覧から、削除したい個別タグ（例：u24.04-2.1.0-amd64、u24.04-2.1.0-arm64）を探す
# 4. 右側にある ゴミ箱アイコン（Delete）をクリック
```
