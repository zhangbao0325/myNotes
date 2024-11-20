# 一、deno安装

## Mac

- curl -fsSL https://deno.land/x/install/install.sh | sh

可能会遇到报错： curl: (28) Failed to connect to github.com port 443: Operation timed out

解决办法: 手动下载下来脚本，按照脚本内容下载zip包进行安装。

​	curl -fsSL https://deno.land/x/install/install.sh > aaa.sh	

安装成功信息:

```	
❯ sh aaa.sh
https://github.com/denoland/deno/releases/latest/download/deno-x86_64-apple-darwin.zip
####                                                                                                                              3.5%#################                                                                                                                13.7%################################                                                                                                 25.3%###########################################                                                                                      34.2%############################################################                                                                     47.8%##########################################################################                                                       58.3%#######################################################################################                                          68.5%############################################################################################################################### 100.0%
Archive:  /Users/mochen/.deno/bin/deno.zip
  inflating: /Users/mochen/.deno/bin/deno
Deno was installed successfully to /Users/mochen/.deno/bin/deno
Manually add the directory to your $HOME/.zshrc (or similar)
  export DENO_INSTALL="/Users/mochen/.deno"
  export PATH="$DENO_INSTALL/bin:$PATH"
Run '/Users/mochen/.deno/bin/deno --help' to get started

Stuck? Join our Discord https://discord.gg/deno
```

- 配置环境变量

```
vim ~/.zshrc

export DENO_INSTALL="/Users/mochen/.deno"
export PATH="$DENO_INSTALL/bin:$PATH"
```

本地切换版本，可能会报错：

```
[ERROR rust_analyzer::main_loop] FetchBuildDataError:
error: linking with `cc` failed: exit status: 1
```

解决办法：将.cargo/config.toml里的配置注释掉。

```
# [target.x86_64-pc-windows-msvc]
# rustflags = ["-C", "target-feature=+crt-static"]

# [target.'cfg(all(windows, debug_assertions))']
# rustflags = [
#   "-C",
#   "target-feature=+crt-static",
#   "-C",
#   # increase the stack size to prevent overflowing the
#   # stack in debug when launching sub commands
#   "link-arg=/STACK:4194304",
# ]

# [target.aarch64-apple-darwin]
# rustflags = ["-C", "link-arg=-fuse-ld=lld"]

# [target.'cfg(all())']
# rustflags = [
#   "-D",
#   "clippy::all",
#   "-D",
#   "clippy::await_holding_refcell_ref",
#   "-D",
#   "clippy::missing_safety_doc",
#   "-D",
#   "clippy::undocumented_unsafe_blocks",
#   "--cfg",
#   "tokio_unstable",
# ]

```



## linux

1、centos8 安装官网安装即可

2、centos7安装可能会报错，需要glibc2.18，升级glibc比较危险，可以采用源码安装。参考： https://deno.js.cn/t/topic/611

可能遇到的问题：

- protobuf-compiler

```SHELL
报错1：

Caused by:
  process didn't exit successfully: `/work/mochen.zb/deno/target/release/build/deno_kv-293db57d11008501/build-script-build` (exit status: 101)
  --- stdout
  cargo:rerun-if-changed=./proto

  --- stderr
  thread 'main' panicked at 'Could not find `protoc` installation and this build crate cannot proceed without
      this knowledge. If `protoc` is installed and this crate had trouble finding
      it, you can set the `PROTOC` environment variable with the specific path to your
      installed `protoc` binary.If you're on debian, try `apt-get install protobuf-compiler` or download it from https://github.com/protocolbuffers/protobuf/releases

  For more information: https://docs.rs/prost-build/#sourcing-protoc
  ', /root/.cargo/registry/src/index.crates.io-6f17d22bba15001f/prost-build-0.11.9/src/lib.rs:1457:10
  note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
  
报错2：

  --- stderr
  Error: Custom { kind: Other, error: "protoc failed: datapath.proto:3:10: Unrecognized syntax identifier \"proto3\".  This parser only recognizes \"proto2\".\n" }
warning: build failed, waiting for other jobs to finish...

  
```

这两个报错都是因为缺少protobuf-compiler，deno需要依赖protobuf-compiler 3+以上的版本，安装方法为：

```SHELL
- wget https://github.com/protocolbuffers/protobuf/releases/download/v3.15.8/protobuf-all-3.15.8.tar.gz
- cd protobuf-3.15.8/
- ./configure
- make -j8
- make install
```

- v8 build卡住很久

方法一：配置国内镜像和socks5代理。可参照cargo.md

方法二： 参照https://github.com/denoland/rusty_v8，先手动下载对应的v8库， 然后直接编译时指定v8的静态库:

```
RUSTY_V8_ARCHIVE=/Users/mochen/Documents/mywork/myProject/edgeworker2/target/debug/gn_out/obj/librusty_v8.a cargo build  -vvv
```



# 二、运行参数

### permission

- --allow-read: 打开读权限，可以指定可读的目录。
- --allow-write： 打开写权限。
- --allow-net： 允许网络通信
- --allow-env： 允许读取环境变量

https://github.com/ms-online/ms-deno-rest-api/tree/lesson-1

### run

- 直接run某个地址： deno run https://examples.deno.land/hello-world.ts



# 三、debugger调试器

- --inspect-brk

  示例：deno run --inspect-brk --allow-read --allow-net /Users/mochen/Documents/mywork/myProject/deno/cli/bench/cache_api.js







