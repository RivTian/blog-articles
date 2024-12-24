```bash
brew search protobuf
brew install protobuf

# output
==> Installing dependencies for protobuf: abseil
==> Installing protobuf dependency: abseil
==> Pouring abseil-20230125.3.arm64_ventura.bottle.1.tar.gz
🍺  /opt/homebrew/Cellar/abseil/20230125.3: 723 files, 10.6MB
==> Installing protobuf
==> Pouring protobuf-23.4.arm64_ventura.bottle.tar.gz
==> Caveats
Emacs Lisp files have been installed to:
  /opt/homebrew/share/emacs/site-lisp/protobuf
==> Summary
🍺  /opt/homebrew/Cellar/protobuf/23.4: 389 files, 12.2MB
==> Running `brew cleanup protobuf`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
==> Caveats
==> protobuf
Emacs Lisp files have been installed to:
  /opt/homebrew/share/emacs/site-lisp/protobuf
```

```bash
# 查看是否安装成功
$ protoc --version
libprotoc 23.4
```