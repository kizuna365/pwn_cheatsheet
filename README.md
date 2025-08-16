# Pwn備忘録

気が向いたら追記する予定。未完。  
何かあれば、X (旧Twitter) アカウント: [@k1_zuna](https://x.com/k1_zuna) まで。

## Docker関連

### docker-composeがある場合
`docker-compose.yml` が提供されている場合は以下を実行:  
```bash
sudo docker-compose up -d
```

### 配布されたDockerからlibcとldを取得

1. 作成されたコンテナを確認し、`libc` と `ld` のパスを確認:  
   ```bash
   sudo docker ps
   sudo docker exec -it <コンテナID> /bin/sh
   ```
   例
   ```bash
   $ ldd binary
	linux-vdso.so.1 
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 
	/lib64/ld-linux-x86-64.so.2 
   $ ls -l /lib/x86_64-linux-gnu/ 
   -rwxr-xr-x  1 root root 2000000 Apr 30  2024 libc-2.31.so
   lrwxrwxrwx  1 root root      12 Apr 30  2024 libc.so.6 -> libc-2.31.so
   $ ls -l /lib64/
   lrwxrwxrwx 1 root root 32 Apr 30  2024 ld-linux-x86-64.so.2 -> /lib/x86_64-linux-gnu/ld-2.31.so
   $exit
   ```

2. ホストにコピー:  
   ```bash
   sudo docker cp <コンテナID>:/path/to/libc.so.6 .
   sudo docker cp <コンテナID>:/path/to/ld-linux-x86-64.so.2 .
   ```
   例
   ```
   sudo docker cp <コンテナID>:/lib/x86_64-linux-gnu/libc-2.31.so .
   sudo docker cp <コンテナID>:/lib/x86_64-linux-gnu/ld-2.31.so .
   ```

### patchelfの使用手順

#### 1. 所有権の変更
Dockerからコピーした `libc` は root 所有のため、現在のユーザーに変更:  
```bash
sudo chown $(whoami):$(whoami) ./libc.so.6
```

#### 2. パーミッションの変更
適切な権限を設定:  
```bash
chmod 644 ./libc.so.6
```

#### 3. バイナリの修正
`patchelf` を使用してバイナリに `libc` と `ld` を適用:

1. **インタープリタ（`ld`）の付け替え**:  
   ```bash
   patchelf --set-interpreter ./ld-linux-x86-64.so.2 chall
   ```

2. **`libc` を指定するディレクトリを追加**:  
   ```bash
   patchelf --set-rpath . chall
   ```

#### 4. 修正内容の確認
以下のコマンドで適用結果を確認:  
```bash
ldd chall
```

これで本番環境に近いバイナリを再現できる

## 解析

### セキュリティ情報の確認
```bash
checksec $binary
```

### 文字列情報の抽出
```bash
rabin2 -z $binary
```

### one_gadgetの利用
```bash
one_gadget -l 1 -f
```

### ROPgadgetの利用
```bash
rp -r 1 -f $binary
```

### シンボルアドレスの確認
```bash
nm -D $binary | grep "symbol_name"
```

