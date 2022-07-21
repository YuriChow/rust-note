目前为止，我们仅仅使用了 Cargo 最基础的特性来 构建、运行 和 测试我们的代码。本章中，我们会讨论更高级的特性。但是 Cargo 还能做更多，最完整的特性，访问 [它的文档](https://doc.rust-lang.org/cargo/)。


# 使用 发布配置 进行客制化构建
1. 在 ***Rust*** 中，***release profiles(发布配置文件)*** 是预定义的，并且，拥有不同配置的客制化配置文件允许一个程序拥有更多对 编译代码的不同选项 的控制。每一个配置文件相对于其他配置文件都是独立配置的。
2. Cargo 有两种主要的配置文件：
   1. **dev 配置文件**：当你使用 `cargo build` 命令的时候 Cargo 使用的配置文件，定义的是**利于开发**的预设值
   2. **release 配置文件**：当你使用 `cargo build --release` 命令的时候 Cargo 使用的配置文件，定义的是**利于发布**的预设值
3. 上面的名字可能你会很眼熟，因为 build 的输出中就有：
> $ cargo build
> Finished dev [unoptimized + debuginfo] target(s) in 0.0s
> $ cargo build --release
> Finished release [optimized] target(s) in 0.0s
4. Cargo 为每个配置文件有默认的设置，当在项目的 *Cargo.toml* 中没有任何 `[profile.*]` 节的时候应用。通过为你想客制化的任何配置文件增加 `[profile.*]` 节，你可以 覆写 默认设置的任何子集。例如，*Cargo.toml* 文件中，这里有一个 `opt-level` 的默认值，用作 dev 和 release 配置文件：
   ```toml
   [profile.dev]
   opt-level = 0
   
   [profile.release]
   opt-level = 3
   ```
5. `opt-level` 设置控制了 ***Rust***  将会对你的代码应用的 优化数量，可选数值是 0 ~ 3。**应用更多优化增加了编译时长**，所以如果你正在开发阶段且经常编译你的代码，会希望编译的快点，即使最终生成的代码运行的会慢一些。而当你准备发布，则刚好相反，最好多花点时间来(优化)编译，毕竟理论上发布只需要编译一次，而生成的程序却需要运行非常多次，所以，发布模式用更长的编译时间来换取程序更快的运行速度。
6. 你可以覆写任何默认设置，通过在 *Cargo.toml* 中为其增加一个不同的值，例如，如果想把开发模式的优化等级 `opt-level` 设置为 `1`：
   ```toml
   [profile.dev]
   opt-level = 1
   ```
7. 查阅配置选项的完整列表和每个配置项的默认值，访问 [Cargo's Documentation](https://doc.rust-lang.org/cargo/reference/profiles.html)。


# 发布一个箱 到 [Crates.io][crates.io]

^Publish-a-Crate-to-Crates-io

我们使用 [crates.io][crates.io] 中的包作为我们项目的依赖，但是你也可以和其他人分享你的代码，通过发布你自己的包。[crates.io][crates.io] 的 ***crate registry(箱注册处)*** 将会分发你的包的源代码，因此他主要存储开源代码。

## 撰写有用的文档注释
精确的文档化你的包可以帮助其他使用者知道 **如何** 和 **什么时候** 使用他们，因此花点时间研究如何写文档是值得的。除了常见的代码注释，***Rust*** 也有文档注释，会生成 ***HTML*** 文档，展示公有 API 项目的文档注释内容，用来给其他程序员了解如何**使用**你的箱，而不是你的箱是如何**实现**的。
1. ***Rust*** 的文档注释使用 `///`，并且支持 ***Markdown*** 标记来排版文字，将文档注释放在被解释代码前，例如：
   ```rust
   /// Adds one to the number given.   
   ///
   /// # Examples
   ///
   /// ```
   /// let arg = 5;
   /// let answer = package_name::add_one(arg);
   ///
   /// assert_eq!(6, answer);
   /// ```
   pub fn add_one(x: i32) -> i32 {
      x + 1
   }
   ```
2. 通过使用 `cargo doc` 命令可以生成 这个文档注释 的 ***HTML*** 文档。这个命令运行了 ***Rust*** 分发的 ***rustdoc*** 工具，并输出到 *target/doc/package_name* 文件夹中的 *all.html*，可以访问。更简洁的，使用 `cargo doc --open` 可以直接 **生成 + 打开**：
> cargo doc 
> cargo doc --open   
3. 注意：无论是否是 `pub` 的函数，都是可以有文档注释的，但是，私密的函数根本无法在 ***HTML*** 文件中生成、访问，因为它是私密的。所以，**不要给私密函数增加文档注释，没有意义，有需要的话普通注释用于代码理解即可**。
### 常用节
我们使用 `/// # Examples` 来在生成的 ***HTML*** 文档中创建一个标题为“Examples”的节，这里有一些箱作者在它们自己的文档中常用的其他节，这些节并不是全都需要，`/// # Examples` 推荐一定要有，其他的如果有需要就要添加，可以作为一个点检表：
1. Examples：创建一个 "Examples" 节 - **推荐必有**
2. Panics：什么情况下函数会 panic
3. Errors：如果函数返回了一个 `Result`，描述可能发生的错误和其可能的原因，方便使用者进行异常处理
4. Safety：如果函数调用是 `unsafe` 的(我们会在 [[17-高级功能]] 中讨论)，那么应该有这节来解释为什么函数是不安全的，并且涵盖 此函数所有希望调用者维持的 不变量。 
### 文档注释作为测试

^Documentation-Comments-as-Tests

`/// # Examples` 节后的代码块不仅可以帮助使用者知道如何使用你的库，而且还有附加的好处：当使用 `cargo test` 命令的时候，将会将示例代码块作为测试进行测试，这也就是测试输出中的 "Doc-tests package_name" 节的测试。没有比包含 Examples 的文档注释更好的了，但是也没有比 Examples 由于写完后代码又进行了更改而导致的无法通过测试 更糟的了。
### 包含项目的注释
另一种 文档注释 `//!`，为 包含此注释的项目 增加文档注释，而不是为 注释之后的项目 增加文档注释，典型的应用这种文档注释是在 箱根文件(通常上是：*src/lib.rs*)中 或者 在一个模块中来整体地注释 箱 或者 模块。例如，如果我们想要增加文档注释来表述 含有 `add_one` 函数的 `my_crate` 箱 的目的，我们在 *src/lib.rs* 文件的开偷增加 `//!` 文档注释：
```rust
//! # Crate.io Learn
//!
//! `crate_io_learn` is a collection of utilitiees to make performing certain
//! calculations more convenient

/// Adds one to the number given.
```   
注意：最后一行以 `//!` 开头的行的后面没有任何代码，因为我们这是 `//!` 而不是 `///`，我们正在给 包含这个文档注释的项目 写文档注释，例子中的文档注释，描述的是整个箱。   
这种文档注释的作用是：**描述箱/模块本身**，主要是传达被描述项目的意义，来帮助你的用户理解箱的组织。

## 使用 `pub use` 导出一个方便的公有 API

^Export-a-Convenient-Public-API-with-pub-use

公开 API 的结构是需要仔细考虑的，相较于 `my_crate::some_moduel::another_module::UsefulType`，人们更喜欢使用 `my_crate::UsefulType`。因为使用者并不像你一样了解模块间的层级架构，而且输入的代码更少不是吗。好消息是，可以在不破坏辛苦整理的模块结构的前提下解决这个问题，通过使用 ***re-export(重导出)*** 技术，代码 `pub use`。这个技术在一个地方获取公有项目，在另一个地方重新让其公有就像我们在这个地方定义的一样。     
使用这个技术可以 “除掉公开 API 中的内部层级”，并且更好的是：在生成的 ***HTML*** 文档中，有单独的 “Re-exports” 节可以区分出那些是重导出的，那些是本就在最上级下定义的。

## 设置一个 [Crates.io][crates.io] 账户
在发布任何 crate 之前，需要在 [crates.io][crates.io] 创建一个账户并获得一个 ***API token(令牌)***。访问网站，并登录 ***GitHub*** 账户，然后访问 [account settings](https://crates.io/me/) 并获取 API key。然后运行 `cargo login api_key` 命令，如：
> cargo login abcdefghijklmnopqrstuvwxyz012345
 
这个命令会知会 Cargo 你的 API 令牌，并且将其存储在本地 *~/.cargo/credentials*。   
注意，令牌是 **机密的：不要和其他人分享！** 如果不小心泄露或者给别人用了，应该撤回并生成一个新的令牌来使用。

## 给一个新箱增加元数据
拥有了账户，如果有一个箱想要发布，在发布前需要增加一些 ***Metadata(元数据)*** 到你的箱中，方法是在 *Cargo.toml* 中的 `[package]` 节中增加。大概过程如下：
1. 箱需要一个**独一无二的名字**。当你在本地编码的时候，可以随便命名，但是 [crates.io][crates.io] 上的箱名都是 **先到先得** 的，一旦一个名字被占用，其他能就不可以发布同名箱。在发布自己的箱之前，在 [crates.io][crates.io] 网站上搜索一下你想用的那个名字，如果名字已经占用，你需要换一个没人用过的名字，并且在 *Cargo.toml* 中的 `[package]` 节中的 `name` 字段进行修改。如下：
   ```toml
   [package]
   name = "another_name"
   ```   
2. 即使你选择了一个独一无二的名字，当使用命令 `cargo publish` 来发布，还是会得到一个警告，然后程序错误：
> $ cargo publish
>       Updating crates.io index
> warning: manifest has no description, license, license-file, documentation, homepage or repository.
> See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
> --snip--
> error: failed to publish to registry at https://crates.io
> 
> Caused by:
> the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang. org/cargo/reference/manifest.html for how to upload metadata   
3. 原因是缺少了一些关键的信息：**一个描述** 和 **开源许可** 是强制要求的，这样人们就知道你的箱是做什么的 和 他们可以使用哪些条款。一样，为了修正，需要在 *Cargo.toml* 中增加这些信息
4. 增加一个**一两句话**的描述，因为他会和箱一起显示在搜索结果中；   
5. 而对于 `license` 字段，需要一个开源许可标识值，可以在 [Linux Foundation’s Software Package Data Exchange (SPDX)](http://spdx.org/licenses/) 中查阅。例如，***MIT*** 协议：
   ```toml
   [package]
   name = "unique_name"
   license = "MIT"
   ```   
5. 如果希望使用 ***SPDX*** 中没有的协议，需要将 含有开源协议文本的文件 包含在你的项目中，并且使用 `license-file` 来明确指出文件名字，而不是使用 `license` 键值。
6. 许多 ***Rust*** 社区的程序员喜欢使用和 ***Rust*** 相同的开源协议：双许可 - `MIT OR Apache-2.0`。这也提示了我们可以使用 `OR` 来引用多个许可。
7. 拥有 独一无二的名、版本号、描述和开源协议，一个可发布的项目的 *Cargo.toml* 看起来像下面这样:
   ```toml
   [package]
   name = "unique_name"
   version = "0.1.0"
   edition = "2022"
   description = "A fun game where you guess what number the computer has chosen."
   license = "MIT OR Apache-2.0"
   ```   
8. 其他的可配置的元数据，请查阅 [Cargo's documentation](https://doc.rust-lang.org/cargo/)。

## 发布到 [Crates.io][crates.io] 
现在你已经拥有了一个账户、保存了 API 令牌、选择了一个名字、并且定义了必须的元数据，可以发布了！     
1. 发布一个箱要考虑清楚，因为一个发布是 ***permanent(永久的)***，版本无法被重写，代码无法被删除。[crates.io][crates.io] 的一个主要的目的就是作为一个永久的存档，这样所有使用其中依赖的项目都能持续工作(编译)，允许删除代码就无法保证这个目的。但是，并不会限制你发布多少个版本。   
2. 发布使用命令 `cargo publish`，如果版本号新于已经存档的版本号，则此命令也可自动更新版本:
> cargo publish

## 为现存箱发布一个新版本
当你修改了你的箱，并且准备发布一个新的版本，你修改 *Cargo.toml* 中的 `version` 值，使用 [***SemVer***](http://semver.org/) 依据你改动的大小来决定下一个版本号。最后，运行 `cargo publish` 来上传新版本。

## 使用 `cargo yank` 从 [Crates.io][crates.io] 移除版本
虽然你无法删除之前的版本，但是**可以禁止任何未来的项目使用之前的版本作为自己新的依赖**。当一个箱版本由于各种原因坏掉了的时候很有用，这种情况，Cargo 支持 ***yanking(猛拉)*** 一个箱版本。Yank 一个版本，将会禁止新项目开始依赖这个版本，同时，允许已经存在的、依赖此版本的项目继续下载并依赖此版本。本质上，yank 意味着所有拥有 *Cargo.lock* 的项目将不会损坏，并且任何未来的 *Cargo.lock* 文件生成将不会使用 yanked 版本。   
1. yank 一个版本，使用 `cargo yank` 命令，并明确想要 yank 的版本，例如：   
> cargo yank --vers 1.0.1
2. 通过使用 `--undo` 选项，可以撤回 yank，重新允许新项目依赖此版本：   
> cargo yank --vers 1.0.1 --undo   
3. 注意：yank 不会删除任何代码。例如，yank 的特性不是为了删除 一不小心上传的秘密，如果发生了，你就需要赶紧重置那些秘密，因为 **[crates.io][crates.io] 没有任何办法能够撤回发布的操作**。


# Cargo ***workspaces(工作区)***

^Cargo-Workspaces

随着项目的增大，你可能希望将包分成多个库箱，这时，Cargo 提供了一个特性叫做 ***workspaces(工作区)***，可以帮助管理多个相关的串联开发的包。

## 创建一个工作区
**一个工作区是一系列 分享同一个 *Cargo.lock* 文件和输出文件夹 的包**。创建工作区有很多方式，一个常见的方式：
1. 创建一个新文件夹(例 *add*)，并进入:
2. 在 *add* 文件夹中，创建一个 *Cargo.toml* 文件用来设置整个工作区。这个文件**没有** `[package]` 节或者我们在其他 *Cargo.toml* 文件中常见的元数据。作为替代的，以 `[workspace]` 节作为开头，允许我们通过 指定包含二进制箱的包 的路径来 增加成员。在这个例子中，path 是 *adder*，所以 *Cargo.toml* 文件如下：
   ```toml
   [workspace]
   
   members = ["adder",]
   ```
3. 接下来创建 adder 二进制箱，通过在 *add* 文件夹中使用 `cargo new adder` 命令
4. 这时我们就可以使用 `cargo build` 命令来构建 工作区   
5. 创建完成，目录应该如下：
   ```
   ├── Cargo.lock
   ├── Cargo.toml
   ├── adder
   │   ├── Cargo.toml
   │   └── src
   │       └── main.rs
   └── target
   ```
6. 一个 workspace 只有一个顶层的 *target* 文件夹用来放置编译后文件，而 adder 没有自己的 *target* 文件夹，即使我们在 *adder* 文件夹内 使用 cargo build 命令，生成的文件还是会被存储在 *add/target* 中 而不是 *add/adder/target* 中。这样做是因为同一个 workspace 中的箱本就需要相互依赖，如果每个箱都有自己的 *target* 文件夹，每个箱都需要在自己的 *target* 文件夹中重新编译，而共享一个 *target* 文件夹避免了这种不必要的重新编译

## 在工作区内创建第二个包
1. 增加一个新的箱，首先需要修改顶层 *Cargo.toml*，将箱名增加到 `members` 字段的数组中。如 add_one：`member = ["adder", "add_one", ]`，然后创建一个名为 add_one 的库箱
> cargo new add_one --lib
2. 这时我们就可以使 adder 依赖 add_one 包，首先在 *adder/Cargo.toml* 中增加 路径依赖 (因为 Cargo 不会默认所有同一工作区的箱都是相互依赖的)：
   ```toml
   [dependencies]
   add_one = {path = "../add_one"}
   ```
   之后就可以在 adder 中使用 `use add_one;` 来导入了。
3. 通过在工作区文件夹(暨顶部 *add* 文件夹)中使用 `cargo build` 命令来构建工作区。
4. 想要在工作区文件夹中运行二进制箱，我们可以通过使用 `cargo run` 配合 `-p` 参数和包名，来明确指出我们想要运行工作区中的哪个包。
> cargo run -p adder
5. 如果没有参数只有 `cargo run`，并且**工作区只有一个二进制箱，则默认执行这个唯一的二进制箱**。
6. 一个 workspace 中并**不限制 库 和 二进制 的数量**，只不过多个二进制箱 不能直接使用 `cargo run` 命令来运行，必须增加 `-p` 参数 并指定要运行的包。
### 在工作区依赖一个外部包
可以注意到，工作区只有根目录有一个 *Cargo.lock* 文件，而不是每个箱里面有一个，这保证了 **工作区中的每一个依赖** 在工作区中不同的箱里版本是一致的，比如，我们增加 `rand` 依赖到 *add/add_one/Cargo.toml* 和 *add/adder/Cargo.toml* 中，Cargo 将会决定他们俩都是用同一个版本的 `rand` ，并且将其记录到 *Cargo.lock* 中。      
1. 注意：工作区根目录下的 *Cargo.toml* 中是不可以增加 `[dependencies]` 节的，会报错，哪个箱使用，就在哪个箱里的 *Cargo.toml* 中增加，就像平常一样。
2. 虽然 *Cargo.lock* 中有了 `rand` 的信息，但是并不代表任何一个工作区的箱都能直接 `use`，除非我们将 `rand` 增加到对应箱中的 *Cargo.toml* 中，才可以在这个箱中使用他。如果单独的箱的 *Cargo.toml* 中没有一个包，则其 *.rs* 文件无法导入、使用这个包，会报错。
3. 虽然在工作区中的多个包中导入，但是并不需要额外下载，因此在工作区中使用同一个版本的包，也更加节省空间，因为我们不需要多个副本来保证工作区中各个箱互相兼容。
### 给工作区增加一个测试
在使用工作区使用 `cargo test` 的时候，会对工作区中所有的箱进行 `cargo test`，并汇总输出。也可以通过 `-p package_name` 选项来在工作区根目录来单独测试一个工作区中的指定项目。       
如果想要将工作区中的箱发布到 [crates.io][crates.io]，每个箱都要单独发布，`cargo publish` 命令没有 `--all` 或 `-p` 选项，因此必须进入到每个箱的文件夹中，然后单独 `cargo publish` 每个箱。      
随着项目的增长，考虑使用 工作区：小的独立的箱 比 一大个箱 更容易理解。并且，将箱(们)放在工作区内可以使他们更方便协作，尤其是他们经常同时改动时。


# 使用 cargo install 从 [Crates.io][crates.io] 安装二进制
`cargo install` 允许在本地安装和使用 二进制箱。这不是为了替换系统包，而是为了方便 ***Rust*** 开发者安装其他人在 [crates.io][crates.io] 上分享的工具。
1. 注意，只可以安装拥有二进制箱的包，一个 ***binary target(二进制目标)*** 是一个可执行程序，只有 箱包含 *src/main.rs* 或者其他被定义为二进制的文件 时才能创建的程序，(与之相对的就是 *library target*，是不可单独执行的)。通常，箱在它的 *README* 文件中有说明这个箱是一个库，还是一个二进制程序，还是都有。
2. 所有使用 `cargo install` 被安装的二进制都被存储在 安装根目录的 *bin* 文件夹中。如果你的 ***Rust*** 是使用 *rustup.rs* 安装的，并且没有任何自定义设置，二进制安装的目录在 *$HOME/.cargo/bin*，保证这个地址在环境变量中，然后才能使用 `cargo install`
3. 例如，下载 grep 工具：
> cargo install ripgrep
4. 只要它的安装目录在 $PATH 中，就可以使用 `rg --help` 来开始使用一个更快、更 ***Rust*** 的文件内文本检索工具


# 使用客制化命令扩展 Cargo
如果在 $PATH 中有一个名为 cargo-something 的二进制程序，可以将其视为一个 cargo 的子命令来运行：
> cargo something

并且如果使用 `cargo --list` 命令，客制化命令也会被列出


# 代码
- *Rust/crate_io_learn/src*
- *Rust/add*
- *Rust/cargo_learn/src/main.rs*


[crates.io]: https://crates.io/