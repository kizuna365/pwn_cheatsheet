# pwn備忘録
気が向いたら追記する、未完

## docker関連
### docker-composeがある場合
```sudo docker-compose up -d```
### 配布されたdockerからlibcとldを取得
```sudo docker cp -L $(container):$(libc.so.6) .```

```sudo docker cp -L $(container):$(ld-linux-x86-64.so.2) .```

### patchelf
```patchelf --set-rpath ./libc.so.6 $binary```

```patchelf --set-interpreter  ./ld-linux-x86-64.so.2 $binary```


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
