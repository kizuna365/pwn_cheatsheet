# pwn備忘録
気が向いたら追記する、未完
何かあれば X垢:@k1_zunaまで

## docker関連
### docker-composeがある場合
```sudo docker-compose up -d```
### 配布されたdockerからlibcとldを取得

作成したcontainerを確認し、libcとldのパスを把握
```
sudo docker ps
sudo docker exec -it <コンテナID> /bin/sh
```
Hostにコピー
```
sudo docker cp <コンテナID>:/path/to/libc.so.6 .
sudo docker cp <コンテナID>:/path/to/ld-linux-x86-64.so.2 .
```

### patchelf
dockerからコピーしたLibcはroot所有のため所有権の変更

```sudo chown $(whoami):$(whoami) ./libc.so.6```

パーミッションも変更

```sudo chmod 644 ./libc.so.6```

patchelfでlibcとldを付け替え

```patchelf --set-interpreter  ./ld-linux-x86-64.so.2 chall```

libcはディレクトリ指定なのに注意

```patchelf --set-rpath . chall```

正しくpatchできてるかチェック

```ldd chall```

これで本番環境とほぼ同じbinaryが手に入った

## 解析

### checksec
```checksec $binary```
### 文字列
```rabin2 -z $binary```
### one_gadget
```one_gadget -l 1 -f```
### ROPgadget
```rp -r 1 -f $binary```
### symbolアドレス
```nm -D $binary | grep "symbol_name"```
