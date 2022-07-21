# 安装
## ***Linux*** or ***macOS***
1. 命令：
   >$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
2. 如果成功，终端显示：
   >Rust is installed now. Great!
3. 你还需要一个链接器，***Rust*** 用来将它的编译输出链接到一个文件中，如果发生链接器错误，你需要安装一个 ***C*** 编译器，它里面有链接器，在 ***macOS*** 上获得 ***C*** 编译器：
   >$ xcode-select --install

## ***Windows***
访问  [***Rust*** 官方安装页面](https://www.rust-lang.org/tools/install)，遵循指引，可能你会需要 ***C++*** 构建工具，按提示下载。 

## 更新和卸载
1. 更新：
   >$ rustup update
2. 卸载：
   >$ rustup self uninstall
3. 检查版本：
   >$ rustc --version
4. 安装成功，上面的命令显示：
   >rustc x.y.z (abcabcabc yyyy-mm--dd)
5. 本地文档：
   >$ rustup doc


# Hello, World!
## 创建 ***Rust*** 代码
1. 创建 *main.rs* 文件，并编辑，然后保存：
   ```rust
   fn main() {
      println!("Hello, World!");
   }
   ```
2. `main`  函数是特殊的：它总是每个可执行 ***Rust*** 程序中运行的第一个代码 
3. `println!` 是一个输出到终端的宏， ***Rust*** 中有 `!` 跟着的函数都是宏，而不是函数。
4. 注意在 ***Rust*** 中，`;`  在大多数行的结尾都是强制要求的。

## 编译 和 运行 是分开的步骤
### 编译
1. 命令：
   >$ rustc main.rs
2. 编译会生成可执行文件，***Linux*** or ***macOS*** 中是 *main*，或在***Windows*** 中是 *main.exe*。并且 ***Windows*** 中还有一个包含 debug 信息的 *main.pdb* 文件
### 运行
编译完成后可以运行
>$ ./main      # 或者如果在 ***Windows*** 上 $ .\\main.exe


# Hello, Cargo!
***Cargo*** 是 ***Rust*** 的构建系统和包管理器，大多数 ***Rust*** 人使用它管理项目，因为它能够替人完成很多工作，例如：构建、下载依赖、构建依赖。(我们将项目需要的库称之为依赖 - ***dependencies***)。
## 安装
cargo 一般在 [安装](#安装 ) 部分就一同安装了，使用下面命令查看版本：
>$ cargo --version

## 使用 Cargo 创建项目
1. 在文件夹中：
   >$ cargo new hello_cargo
   >$ cd hello_cargo
2. 查看 *Cargo.toml* 文件：
   ```toml
   [package]
   name = "hello_cargo"
   version = "0.1.0"
   edition = "2021"

   [dependencies]
   ```
3. ***TOML(Tom’s Obvious, Minimal Language)*** 文件是 Cargo 的配置格式。
	1. 第一行 `[package]` 是一个节的开头，说明接下来的语句在定义一个包，随着项目信息的增加，我们可能会增加其他的节。节中的三个语句定义了：程序名、版本 和 使用的 ***Rust*** 版本
	2. 最后一行 `[dependencies]` 是依赖节的开头，在这里你列出程序所需要的所有依赖。***Rust*** 中包被成为 ***箱(crate)***。
4. 如果你打开生成的 *hello_cargo/src/main.rs* 文件，你会看到 Cargo 自动生成了一个 Hello, world! 代码
5. Cargo 希望你将源代码文件都放到 *src* 文件夹中，项目根目录仅放置：*README* 文件、协议信息、配置文件 和 任何其他与你代码无关的东西。

## 构建 与 运行 Cargo 项目
### 构建
1. 使用命令来构建：
   >$ cargo build
2. 构建在文件夹 *target/debug/* 中生成一个可执行文件。而不是在你当前的目录，你可以进入到文件夹中运行它。
3. 同时，build 命令首次会生成一个 *Cargo.lock* 文件在项目根目录。这个文件追踪了项目中使用的依赖构建时实际的版本，是自动生成的，不需要手动修改。
### 构建 与 运行
我们不需要进入到 *target/debug/* 文件夹中运行生成的可执行文件，可以直接通过命令 **构建 并 运行**：(注意到，如果代码没有变更，Cargo 不会重复编译)
>$ cargo run
### 检查
如果只希望检查代码的编译，而不希望生成可执行文件，可使用：
>$ cargo check
### 为发布构建
当你的代码可以发布了，可以使用这个命令来进行 **有优化的编译**，将会在 *target/release/* 文件夹中生成最终的可执行文件。debug 模式下，不优化，编译快，运行慢；release 模式下，优化，编译慢，运行快。除非你准备发布或者进行性能基准测试，否则使用 debug 模式来获得更快的编译速度避免影响开发速度：
>$ cargo build --release

## More
1. 创建一个新包：
   >$ cargo new hello_world
   >$ cargo new hello_world --bin
2. 创建一个新库(library)：
   >$ cargo new --lib hello_world_lib
   >$ cargo new hello_world_lib --lib 
3. [crates.io](https://crates.io/) 是 ***Rust*** 社区中心包登记处，是发现和下载包的地方 Cargo 默认使用它来寻找需求的包。
4. 增加 依赖，在 *Cargo.toml* 中的 `[dependencies]` 节中(如果没有就自己添加一个)，增加想用的 名字 和 版本：
   ```toml
   [dependencies]
   time = "0.1.12"
   regex = "0.1.41"
   ```   
5. version 字符串是一个 ***SemVer***：
    1. 主版本号：当你做了不兼容的 API 修改；     - major
    2. 次版本号：当你做了向下兼容的功能性新增；   - minor
    3. 修订号：当你做了向下兼容的问题修正；       - patch
6. build 第一次就会自动下载包，第一次(暨还没有 *Cargo.lock* 文件的时候) build 包的版本是 **可升级的最新版本修订号**，如 `"0.1.1"` 实际首次构建使用的包是  **早(小)于 `"0.2.0"` 的最新版本**，比如 `"0.1.999"`，在项目中使用。暨，比如 `time = "0.1.12"` 看起来是一个确定的版本，实际上是一个版本范围，允许 ***SemVer*** 兼容性更新：暨 当新版本不修改最左边第一位非零数字(以 . 算位) 的版本 才是可更新到的版本。0.1.13 无法升级到 0.2.0；1.0 无法升级到 2.0；0.0.x 不和任何版本兼容。
7. 如果包已经被更新了，我们仍会使用和 *Cargo.lock* 中相同版本来构建，直到我们使用以下命令才会更新到 **相同次版本的最新版本修订号**，如果要使用更新的 **主版本号.此版本号** 则必须修改 *Cargo.toml* 文件
   >$ cargo update
8. ***main.rs*** 存在 ***./src/*** 中，其他的 ***.rs*** 文件可以存放在 ***./src/bin/*** 中
9. 如果创建 库(--lib)，将 *Cargo.lock* 添加到 *.gitignore* 中。如果创建的是 ***命令行工具或者应用*** 则不用。在使用 ***Cargo*** 命令创建的时候，***Cargo*** 会自动默认这样做。因为使用 库 的时候，不会检视 *Cargo.lock* 文件，即使它存在。
10. 使用其他位置的包：
   ```toml
   [dependencies]
   regex = {git = "https://github.com/rust-lang/regex.git"}       
   // 这样是默认 最新的版本
   regex = { git = "https://github.com/rust-lang/regex.git", rev = "9f9f693" }
   // 这样是使用 SHA1 标识，也是不好用的
   // 推荐使用前一个，然后分享的时候，Cargo 会使用最新的版本，当分享给其他人的时候，其他人可以通过 Cargo.lock 的帮助完成编译
   ```
11. [crates.io](https://crates.io/) 不允许通配版本(wildcard) \* , 1.\* , 1.2.\* 。
12. 使用路径依赖，`path` 的值可以是相对路径(常用)或者绝对路径
   ```toml
   [dependencies]
   hello_utils = {path = "hello_utils", version = "0.1.0"}
   ```
13. 关于 Cargo 的更多信息，检索它的 [文档](https://doc.rust-lang.org/cargo/)
14. 编辑器可以使用 ***VSCode***，插件推荐使用 ***rust-analyzer插件***(持续更新中) 替代 ***Rust插件***(2020年停更 - 2022/7/14 已废弃 无法下载)
15. 再推荐一个 ***crates*** 插件，可以显示导入依赖的最新版本。







   

