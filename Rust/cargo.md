# 一、cargo常用配置

## 国内镜像配置

- 修改$HOME/.cargo/config

```toml
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
# 指定镜像
replace-with = '镜像源名' # 如：tuna、sjtu、ustc，或者 rustcc
 
# 注：以下源配置一个即可，无需全部
 
# 中国科学技术大学
[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index"
# 或者写下面这个地址：
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
 
# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index/"
 
# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"
 
# rustcc社区
[source.rustcc]
registry = "https://code.aliyun.com/rustcc/crates.io-index.git"



#如果有socks代理，也可以配置，下载v8库会很快

[http]
proxy="socks5h://127.0.0.1:13659"
#proxy="socks5h://11.166.91.15:1080"

[https]
proxy="socks5h://127.0.0.1:13659"
#proxy="socks5h://11.166.91.15:1080"

```

可以通过以下方式验证socks5代理是否通的：

curl -x socks5h://11.166.91.15:1080 http://httpbin.org/ip



## cargo env配置

Q: 如何设置 RUSTY_V8_ARCHIVE=/work/mochen.zb/edgeworker2-deps/librusty_v8.a到环境变量，每次cargo run的时候能自动识别？

A: 要在 Rust 环境中设置环境变量，特别是希望该变量在每次运行 `cargo run` 时都能被自动识别，你可以采用几种方法。这里提供两种常见且推荐的设置方法：

### 方法1：在 shell 配置文件中设置

对于大多数 Unix/Linux shell（如 Bash, Zsh），你可以在你的 shell 配置文件（如 `~/.bashrc`, `~/.zshrc` 或 `~/.profile`）中添加以下行来设置环境变量：

```
export RUSTY_V8_ARCHIVE=/work/mochen.zb/edgeworker2-deps/librusty_v8.a
```

然后，通过执行 `source ~/.bashrc` （或相应的配置文件名）命令使更改生效。这样设置后，每次你打开新的终端会话时，这个环境变量都会自动加载。

### 方法2：使用 `.cargo/env` 文件

Cargo 允许项目级别的环境变量配置，你可以在项目根目录下创建一个名为 `.cargo` 的目录（如果尚不存在），并在其中创建一个名为 `env` 的文件。在这个 `env` 文件中，你可以设置你的环境变量，如下所示：

```
RUSTY_V8_ARCHIVE=/work/mochen.zb/edgeworker2-deps/librusty_v8.a
```

Cargo 在构建和运行项目时会自动加载此文件中的环境变量。这意味着每次你执行 `cargo run` 时，`RUSTY_V8_ARCHIVE` 都会被正确设置，而不需要在命令行中手动设置。

### 注意事项

- 确保路径 `/work/mochen.zb/edgeworker2-deps/librusty_v8.a` 是正确的并且该静态库文件确实存在。
- 使用第二种方法时，只有在通过 Cargo 运行或构建项目时这些环境变量才会生效。直接运行二进制文件不会应用这些设置。



# 二 、cargo常用命令

使用文档： https://rustwiki.org/zh-CN/cargo/getting-started/first-steps.html

## 常用命令

### new

- cargo new <project_name>

### add

- cargo add deno_core
- cargo add ferris-says@0.2

### build：安装依赖+编译

- cargo build / cargo build --debug / cargo build --release

### run

在 Rust 项目中，当你使用 `cargo run` 命令运行某个二进制文件时，你可以在命令后加上 `--` 并带上所需的启动参数。`--` 用于指示 `cargo run` 命令之后的参数都将传递给生成的可执行文件，而不是 Cargo 本身。

如果你的项目的 `Cargo.toml` 中已经设置了 `default-run` 为 `edgeworker2`，你可以像下面这样执行命令，并传递参数：

- cargo run [CARGO_OPTIONS] -- [PROGRAM_OPTIONS]， 其中 `[CARGO_OPTIONS]` 是可选的，属于 Cargo 命令的选项，如 `--bin <name>`、`--release` 等。`[PROGRAM_OPTIONS]` 是传递给程序的运行时选项。选项之间用 `--` 分隔。
  - cargo run --  -v --config_file /work/mochen.zb/edgeworker2/conf/config-local.toml
  - 如果你想执行一个不是 `default-run` 的二进制文件，你可以使用 `--bin` 参数来指定：cargo run --bin edgeworker2 -- -v --config_file /work/config.toml

- cargo run / cargo run --debug / cargo run --release
- cargo run --example mutlithread

### check 

- cargo check



# 常见报错

A: rust执行cargo run时一直卡在 Blocking waiting for file lock on package cache上

Q: 当你在使用 Rust 的 Cargo 工具时，遇到 "Blocking waiting for file lock on package cache" 消息通常意味着 Cargo 正在等待一个文件锁，这个锁通常是由另一个同时运行的 Cargo 进程持有的。这种情况可能发生在多个 Cargo 命令试图同时运行时，例如构建、测试或更新依赖。

如果你遇到了这个问题，以下是一些可能的解决步骤：

1. **检查是否有其它正在运行的 Cargo 进程：**
   有可能你已经在另一个终端或背景中启动了 Cargo 命令。检查系统中所有的 Cargo 进程，并在不需要的话终止它们。

2. **检查编辑器或 IDE 是否正在使用 Cargo：**
   某些编辑器或 IDE 会在背景中使用 Cargo 来检查代码、获取代码完成提示等。确保编辑器不是正在使用 Cargo。

3. **删除 Cargo 的锁文件：**
   如果你确认没有其它进程正在使用 Cargo，那么锁文件可能是由于之前的 Cargo 命令异常终止而留下的。锁文件通常位于你的 Cargo 仓库目录中，例如 Unix 系统下的 `~/.cargo/.package-cache`。删除这个锁文件后重试。

4. **检查文件系统权限：**
   确保你有权限写入 Cargo 仓库目录。权限问题可能会导致 Cargo 无法获取所需的锁。

5. **重启计算机：**
   如果上述步骤都没有帮助，尝试重启计算机，这可以清除挂起的进程和锁文件。

6. **网络问题：**
   如果锁是由于正在下载依赖包而产生的，网络问题可能会导致 Cargo 命令阻塞。确保你的网络连接是稳定的。

7. **Cargo 的 bug：**
   尽管不常见，但有可能是 Cargo 自身的 bug 造成的问题。如果怀疑是这种情况，可以尝试更新到最新版本的 Cargo 和 Rust，或者查看 Cargo 的 issue 跟踪器看是否有类似的已知问题。

一般情况下，上述步骤中的一个或多个应该能帮你解决问题。如果问题仍然存在，你可能需要更深入地调查或寻求社区的帮助。
