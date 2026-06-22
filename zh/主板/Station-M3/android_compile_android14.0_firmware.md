

## 常见问题

### git clone 远程 bundle 仓库失败

Q：[更新SDK](android_compile_Android14.0_firmware.html#geng-xin-sdk)时，git clone 远程 bundle 仓库出错：

```
$ git clone https://gitlab.com/T-Firefly/rk3588-Android14.0-bundle.git .bundle
Cloning into '.bundle'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (18/18), done.
error: RPC failed; curl 56 GnuTLS recv error (-9): A TLS packet with unexpected length was received.
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: unpack-objects failed
```

A：缓冲区大小不够，需要扩大缓冲区：

```
git config --global https.postBuffer 1048576000
```