有许多互相竞争的 OOP 定义，有些会将 ***Rust*** 划为面向对象的，有些不会。


# 面向对象语言的特征
一个编程语言必须要有什么特性才能被视为是面向对象的 在编程社区中没有共识。***Rust*** 被很多编程范式影响，包括 OOP。例如我们 [[11-函数式编程语言特性：迭代器和闭包]] 中探索了 来源于函数式编程 的特性。按理，OOP 语言共享一些特定的公用特性，暨：对象、封装、继承。我们来看看是否 ***Rust*** 支持它们。

## 对象包含数据和行为
***《Design Patterns: Elements of Reusable Object-Oriented Software》*** *by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides (Addison-Wesley Professional, 1994)*，aka ***The Gang of Four*** book - **四人帮书**，是一个面向对象设计模式的目录，它是这样定义 OOP 的：面向对象程序是由对象组成的。一个 ***object(对象)*** 同时将 数据 和 对数据进行操作的程序 包装，这个程序一般叫做 ***method(方法)*** 或 ***operations(操作)***。参考这个定义，***Rust*** 是面向对象的：`struct` 和 `enum`，并且 `impl` 块提供了 `struct` 和 `enum` 上的方法。虽然拥有方法的结构体和枚举不称呼为对象，但是他们提供了相同的功能，依据四人帮对对象的定义。

## 隐藏实现细节的封装
另一个通常和 OOP 关联的方面是 ***encapsulation(封装)*** 概念，意味着一个对象的实现细节对使用这个对象的代码来说是不可见的。因此，与对象交互的唯一方法是通过它的 公有的 API，代码不应该能够接触到对象的内部并且直接修改数据或行为。这使程序员可以 在不需要修改使用对象的代码 的前提下 修改和重构对象的内部。
1. 我们在 [[6-使用包&箱&模块管理增长的项目]] 讨论了如何控制封装：使用 `pub` 关键字来决定哪个模块、类型、函数和方法应该为公有的，并且默认所有都是 私有的。
2. 在我们实现一个类型的时候，由于封装特性，我们仅暴露一些 API，只要这些 API 没有变，内部是随意修改/重构的，外部的代码由于使用的 API 没有任何变更，因此不需要做任何改变。
3. 如果 封装 是面向对象编程语言的一个要求，那么 ***Rust*** 是满足要求的。

## 作为类型系统和代码共享的继承
***Inheritance(继承)*** 是一个机制，通过这个机制，一个对象可以继承自另一个对象的定义，因此获得父对象的数据和行为，而不需要你重新定义一次。
1. 如果一个语言必须要有继承才能被称之为面向对象的，那么 ***Rust*** **不是**，没有任何方式来定义一个 继承父结构体的字段和其所实现的方法的 子结构体。然而，如果你习惯在你的编程工具中有继承功能，在 ***Rust*** 中你可以使用其他解决方案，取决于你第一时间需要继承的原因。
2. 你选择继承有两个主要原因：
   1. 代码复用。在 ***Rust*** 中，你可以使用 **特性方法的默认实现 来分享代码，代替继承的复用作用**。只要实现了这个特性的类型，就拥有这个特性方法的默认实现，和继承一样，不需要多余的代码，而且，你还可以 override(重写)，覆盖掉默认实现。
   2. 允许子类被用于本该是父类的位置，也称为 ***polymorphism(多态)***，意味着运行时，如果几个类型共享特定的特点，你可以互相替换他们。对许多人来说，多态和继承紧密联系。但是其实它是一个更通用的概念，指代码可以配合多种类型的数据进行工作，对继承来说，那些类型通常是子类。***Rust***，作为替代的，**使用泛型**来抽象多种可能的类型，并且**使用特性约束**来强制限定那些类型必须提供的东西。这有时被称为 ***bounded parametric polymorphism(约束/有界参数多态)***。
3. **继承 作为很多编程语言的一个编程设计解决方案 最近失宠了**，因为它经常冒着分享过多不必要的代码的风险。子类不应该总是分享所有其父类的特征，但是继承会导致这样。这会导致一个程序设计缺少灵活性，而且还引入了在子类上调用方法没意义或者造成错误的可能性，因为那个方法不应该应用于子类。另外，一些语言仅允许单继承(如 ***JAVA***)，会进一步限制程序设计的灵活性。因此，***Rust*** 换了条路走，使用 trait(特性) 代替继承。让我们来看看在 ***Rust*** 中特性对象如何允许多态。


# 使用允许不同类型值的特性对象

^Using-Trati-Objects-That-Allow-for-Values-of-Different-Types

在 [[7-通用集合]] 中，我们提及了向量(数组)的限制 - 他们只能存储单一类型的数据。我们创建了一个迂回，定义一个 `SpreadsheetCell` 枚举，其中的变体持有 整形、浮点、文本，意味着我们可以在每个格中存储不同类型的数据，并且获得了一个 代表了一排这种格的 向量。当我们的可互换项目在编译时是固定的一套类型，这是一个完美的解决方案。      
然而有时我们希望使用我们库的程序员可以拓展 类型的集合(在特定情况下有效)。为了展示我们如何达成这个功能，我们创建一个示例 GUI 工具，迭代一个项目列表，在每个项目上调用 `draw` 方法来将其打印到屏幕 - GUI 工具中一个常用的技术。      
如果用拥有继承的编程语言来实现，我们会定义一个类名 `Component`，其拥有一个方法 `draw`，其他如 `Button`，`Image` 等会继承 `Component`。这样它们就可以重写 `draw` 来定义它们自己的行为，但是框架仍会将它们视为 `Component`，并且调用他们的 `draw` 方法。

## 为共有行为定义一个特性
为了实现上面的需求，我们定义一个名为 `Draw` 的特性，其拥有一个 `draw` 方法。然后我们可以定义一个存储 ***trait object(特性对对象)*** 的向量，**一个特性对象既指向实现指定特性的类型的实例，也指向了一个用于在运行时查找该类型上的特性方法的表**。我们通过指定某种指针来创建一个特性对象，例如一个 `&` 引用，或者一个 `Box<T>` 智能指针，接下来是 `dyn` 关键字，然后指定相关特性(为什么特性对象一定要使用指针我们会在 [[17-高级功能#^Dynamically-Sized-Types-and-the-Sized-Trait]] 讨论)。我们可以使用特性对象代替泛型或者具体类型。不论我们在哪使用特性对象，***Rust*** 的类型系统会保证编译时上下文中使用的任何值都会实现特性对象的特性，因此我们不需要在编译时知道所有可能的类型。例如：
```rust
pub trait Draw {
   fn draw(&self);
}
pub struct Screen {
   pub components: Vec<Box<dyn Draw>>,
}
impl Screen {
   pub fn run(&self) {
      for component in components {
         component.draw();
      }
   }
}
```
1. 我们之前提过，我们克制称呼 `struct` 和 `enum` 为 “object”，来将其和其他语言的 对象 进行区分。在一个 `struct` 或 `enum` 中，结构体字段中的数据和 `impl` 块中的行为是分开的，而其他语言，数据和行为合并的概念被称之为一个对象。然而，特性对象**更像是**其它语言中的对象，就 他结合了数据和行为 而言。但是特性对象区别于传统对象在于：我们不能在特性对象中增加数据。特性对象并不像对象在其他语言中一样 通用：它的目的仅是抽象出共有行为。
2. 特性对象的定义使用 和 泛型配合特性约束 并不相同。一个泛型参数在同一时间只能被一种具体类型代替，而特性对象 允许在运行时为 特性对象 填充多种具体类型。例如如果我们按照泛型配合特性约束来定义：
   ```rust
   pub struct<T> Screen {
      pub components: Vec<T>,
   }
   impl<T> Screen<T> 
   where 
   T: Draw,
   {
      pub fn run(&self) { ... }
   }
   ```   
   这时候我们还是只能接受一种类型添加到 `components` 种，也就是全都是 `Button` 类型 或者 全都是 `TextField` 类型。如果你需要的是同种类的集合，才应该使用这种定义。      
   而另一方面，通过使用特性对象的方法，一个 `Screen` 示例可以持有一个向量，既可以包含 `Box<Button>` 也可以包含 `Box<TextFiled>`。

## 实现特性
实现一些类型。我们不会具体实现 GUI 库，主要是看概念。每一个类型的都可能不一样，包括字段、方法的数量/名字/功能都可能不同，但是他们都要实现自己的 `draw` 方法。如：
```rust
pub struct Button {
   pub width: u32,
   pub height: u32,
   pub label: String,
}
impl Draw for Button {
   fn draw(&self) {
      // 实际的输出代码
   }
}
```
1. 当我们写这个库的时候，我们无法知道是否会有人增加 `SelectBox` 类型，但是我们的 `Screen` 实现能够操作新的类型，并且绘制它，因为 `SelectBox` 实现了 `Draw` 特性。
2. 只关心一个值响应的消息而不是这个值具体是什么类型的，这种概念和 动态类型语言中的 ***duck typing(鸭子类型)*** 概念十分相似：如果它走路像鸭子并且叫的像鸭子，那它就是个鸭子！比如我们实现的 `Screen` 中的 `run` 函数，只是调用组件的 `draw` 方法，不去检查它是什么类型的。通过指定 `Box<dyn Draw>` 作为 `components` 中向量中的值的类型，我们定义了 `Screen` 需要的值是我们能在其上调用 `draw` 方法的值。
3. 使用 特性对象 和 ***Rust*** 类型系统来写代码 的优势类似于使用鸭子类型：我们永远不需要检查一个值在运行时是否实现了一个特定给定的方法 或者 担心如果一个值没有实现一个方法但我们却调用了这个方法导致出错，如果值没有实现特性对象需要的特性，那么 ***Rust*** 根本不会编译通过。

## 特性对象执行动态分配
回忆起 [[9-泛型&特性&生命周期#^Performance-of-Code-Using-Generics]]，我们讨论了当我们在泛型上使用特性约束的时候由 编译器 执行的 ***monomorphization process(单态化处理)***：编译器 为我们用来代替泛型类型参数的每个具体类型 生成函数和方法的非泛型实现。单态化处理生成的代码执行 ***static dispatch(静态分配)***，暨编译器在编译的时候就知道你将调用什么方法，与之相反的是 ***dynamic dispatch(动态分配)***，暨编译器在编译时不知道你将调用什么方法。在动态分配的情况下，编译器会发出代码，在运行时才确定要调用哪个方法。   
当我们使用特性对象的时候，***Rust*** 必须使用动态分配。编译器无法知道 可能与使用特性对象的代码 一起使用的所有类型，因此它不知道哪个方法实现在哪个类型上来供调用。作为替代的，运行时，***Rust*** 使用特性对象中的指针来知道调用哪个方法。当发生这种检索行为的时候，**会带来额外的运行时性能损耗**，而静态分配不会。动态分配还会阻止编译器决定 ***inline***(***内联***：源于 ***C++***，用来解决一些频繁调用的函数大量消耗栈内存的问题) 一个方法的代码，因此导致阻碍了一些优化。      
虽然使用特性对象来处理多态会由于动态分配带来些性能损耗(相较于使用泛型)，但是我们也确实获得了些代码灵活性，这是一个 权衡


# 实现一个面向对象的设计模式
***state pattern(状态模式)*** 是一个面向对象的设计模式。模式的难点在于一个值有一些内部状态，表现为一组 ***state objects(状态对象)***，并且对象的行为会依据内部状态进行改变。状态对象共享功能：在 ***Rust*** 中，当然，我们使用结构体和特性，而不是对象和继承。每个状态对象负责自己的行为，并负责控制何时应更改为另一个状态，而持有这个状态对象的值对这些一无所知。      
使用状态模式意味着当业务要求程序改变，我们不需要改变 持有状态的值的代码 或者 使用这个值的代码。我们只需要更新 其中一个状态对象中的代码 来改变它的规则，或 可能增加更多状态对象。   
我们以递增的方式实现一个帖子发布流程，最终行为如下：
1. 开始为空草稿；
2. 当草稿完成，要求审核帖子的发布；
3. 当发布审核通过，帖子可以发布；
4. 只有发布了的帖子返回可打印内容，这样没通过的帖子不能意外发布。

## 定义 `Post` 并创建一个在草稿状态的新实例
```rust
pub struct Post {
   state: Option<Box<dyn State>>,
   content: String,
}
impl Post {
   pub fn new() -> Post {
      Post {
         state: Some(Box::new(Draft {})),
         content: String::new(),
      }
   }
}
trait State {}
struct Draft {}
impl State for Draft {}
```

## 存储帖子内容的文本
```rust
impl Post {
   pub fn add_text(&mut self, text: &str) {
      self.content.push_str(text);
   }
}
```

## 保证草稿帖子的内容是空的
即使我们已经调用了 `add_text` 增加了一些内容到我们的帖子对象中，我们仍然希望 `content` 方法返回空字符串，因为帖子还在草稿阶段。先使用一个永远返回空字符串的方法来占位，后面再进行修改：
```rust
impl Post {
   pub fn content(&self) -> &str {
      ""
   }
}
```

## 要求一个帖子审核来改变它的状态
```rust
impl Post {
   pub fn request_review(&mut self) {
      if let Some(s) = self.state.take() {
         self.state = Some(s.request_review())
      }
   }
}
trait State {
   fn request_review(self: Box<Self>) -> Box<dyn State>;
}
struct Draft {}
impl State for Draft {
   fn request_review(self: Box<Self>) -> Box<dyn State> {
      Box::new(PendingReview {})
   }
}
struct PendingReview {}
impl State for PendingReview {
   fn request_review(self: Box<Self>) -> Box<dyn State> {
      self
   }
}
```
1. 特性中的 `request_review` 方法消耗当前 `state` 的所有权，夺走了 `Box<Self>` 的所有权，并返回一个新状态的所有权，由于是加在特性定义中的无默认实现的方法，因此每个实现都需要实现这个函数。
2. 注意：特性中的 `request_review` 参数没有使用 `self`，`&self`，`&mut self` 作为方法的第一个参数，而是使用了 `self: Box<Self>` 作为第一个参数，记得 `self` 是 `self: Self` 的缩写。这个语法意思是：这个方法只有在 当持有类型的 `Box` 调用时 这个方法才是有效的。
3. 为了消耗掉旧状态，`request_review` 需要夺走 ownership，这也是 `Post` 定义中使用 `Option<T>` 的原因：我们调用 `take` 方法来将 `Some` 值拿出 `state` 字段，然后在这个位置留下了一个 `None`，因为 ***Rust*** 禁止字段无值。这样操作，我们能够将 `state` 的值移出 `Post` 对象而不是借用它。
4. 我们需要将 `state` 暂时设置为 `None` 而不是使用类似于 `self.state = self.state.request_review();` 这样的代码直接设置它来获得 `state` 值的所有权，这保证了 `Post` 在我们转换他到一个新状态后无法使用旧的 `state` 值。
5. 现在我们就看到了状态模式的优势了：`Post` 上的 `request_review` 方法无论 `state` 是什么 它都是相同的，每个 `state` 对自己的规则负责

## 增加修改 `content` 行为的批准方法
```rust
impl Post {
   pub fn approve(&mut self) {
      if let Some(s) = self.state.take() {
         self.state = Some(s.approve())
      }
   }
}
trait State {
   fn request_review(self: Box<Self>) -> Box<dyn State>;
   fn approve(self: Box<Self>) -> Box<dyn State>;
}
struct Draft {}
impl State for Draft {
   fn approve(self: Box<Self>) -> Box<dyn State> {
      self
   }
}
struct PendingReview {}
impl State for PendingReview {
   fn approve(self: Box<Self>) -> Box<dyn State> {
      Box::new(Published {})
   }
}
struct Published {}
impl State for Published {
   fn request_review(self: Box<Self>) -> Box<dyn State> {
      self
   }
   fn approve(self: Box<Self>) -> Box<dyn State> {
      self
   }
}
```
过程和添加 `request_review` 类似，不赘述。   
修改 `content` 方法：
```rust 
impl Post {
   pub fn content(&self) -> &str {
      self.state().as_ref().unwrap().content(self);
   }
}
trait State {
   fn request_review(self: Box<Self>) -> Box<dyn State>;
   fn approve(self: Box<self>) -> Box<dyn State>;
   fn content(&self, post: &Post) -> &str {
      ""
   }
}
struct Published {}
impl State for Published {
   fn content<'a>(&self, post: &'a Post) -> &'a str {
      &post.content
   }
}
```
1. 我们在 `&Box<dyn State>` 上调用 `content()`，由于解引用强制多态对 `&` 和 `Box` 的生效，`content` 方法最终会在实现了 `State` 的类型上被调用，这意味着，我们需要在 `State` 中增加这个方法。我们可以给 `content` 一个默认的实现，暨安全地返回 `""`
2. 由于 `content()` 在 `State` 上有默认实现，因此仅 `Published` 需要重写该方法
3. 注意：我们需要生命周期标记，我们使用了一个 `Post` 对象的引用作为参数 `post`  并且 返回这个对象一部分的引用，所以返回值的生命周期和 `post` 实参相关联。
4. 注意 `&post.content`，`&` 的优先级在 `.` 之后。例如：
   ```rust
   let s = String::from("hello");
   let a = (&s).len(); // usize
   let b = &s.len(); // &usize
   ```

## 状态模式的权衡
我们已经展现了：***Rust*** 能够实现面向对象的状态模式 来封装一个 `Post` 对象在不同状态下的不同类型的行为。`Post` 的方法对于不同行为一无所知(`Post` 对象调用的代码都是相同的)，这种代码结构，我们只需在一个地方就能找到一个 发布状态的 `Post` 对象可能表现的不同行为：实现了 `State` 特性的 `Published` 结构体。        
1. 如果我们换另一种方式来实现这个功能，我们可能会在 `Post` 中使用 `match` 表达式 甚至在  `main` 代码中检查对象的状态然后执行不同的代码。这意味着我们 必须查看很多个地方 才能理解一个发布状态的帖子的所有实现，随着我们状态的增加 这只会增加更多：每一个 `match` 表达式需要另一个分支。      
2. 使用状态模式，`Post` 的方法和我们使用 `Post` 的位置不需要 `match` 表达式，要添加一个新状态，我们只需要添加一个新结构并在该结构上实现 `trait` 方法。
3. 使用状态模式的实现，**更易于拓展和增加更多功能**
4. 一个状态模式的缺点就是：由于状态中实现了状态间的转换，一些状态之间是互相耦合的。如果我们在 `PendingReview` 和 `Published` 之间增加另一个状态，例如 `Scheduled` ，我们必须将 `PendingReview` 中的代码修改来转换至 `Scheduled`，如果 `PendingReview` 不需要 由于新增状态 来改变，工作量会少很多，但是那就意味着我们切换到了另一种设计模式。
5. 另一个状态模式的缺点就是：我们重复了一些逻辑。为了避免重复，你可能会尝试为一些 特性定义 上的方法增加 默认实现 返回 `self`，然而，这会违反对象安全，因为特性并不知道实际的 `self` 是什么类型，我们希望将 `State` 用作特性对象，因此我们需要它的方法是**对象安全的**。   
6. 另一个重复是 `Post` 的 `request_review` 和 `approve` 方法有相似的实现，如果在 `Post` 上有很多这种模式的方法，我们应该考虑定义一个**宏**，来减少重复(19 章会提到)。
7. 这只是利用 ***Rust*** 实现了状态模式，我们还没完全利用上 ***Rust*** 的长处呢，接下来我们将修改代码使 非法状态 和 转变 变成**编译时**错误。
### 将状态和行为编译为类型
我们将会展示给你如何重新思考状态模式来获得一组不同的权衡。我们将会将状态编码进不同的类型，而不是完全的封装状态和转换好让外部代码对其一无所知。这样 ***Rust*** 的类型检查系统将会通过报出编译时错误，来阻止我们尝试 在只有已发布博客被允许使用的地方 使用草稿博客。我们希望有的状态下某些方法就不应该存在，比如：不是 `Published` 状态下的 `Post` 对象根本就应该没有 `content` 方法，一旦调用直接编译器报错。这样我们就不会误调用一些没意义的方法。
```rust
pub struct Post {
   content: String,
}
pub struct DraftPost {
   content: String,
}
impl Post {
   pub fn new() -> DraftPost {
      DraftPost { content: String::new(), }
   }
   pub fn content(&self) -> &str {
      &self.content
   }
}
impl DraftPost {
   pub fn add_text(&mut self, text: &str) {
      self.content.push_str(text);
   }
}
```
1. 不再有 `state` 字段了，因为我们将 `state` 的编码移动到了结构体的类型(如 `DraftPost`)中了
2. 由于 `content` 是 private 的，`new` 方法又是返回的 `DraftPost`，目前没有方法能创建 `Post` 实例
3. 虽然在 `DraftPost` 中我们增加了 `add_text` 方法，但注意，这里可没有 `content()` 方法。现在程序保证了所有 `Post` 对象都是从 `DraftPost` 对象开始的，而且 `DraftPost` 对象没有 `content()` 方法，所以如果尝试在草稿阶段就调用根本不存在的 `content()`，直接会编译错误
### 实现 转换为不同类型的 变换
```rust
impl DraftPost {
   pub fn request_review(self) -> PendingReviewPost {
      PendingReviewPost { content: self.content, }
   }
}
struct PendingReviewPost {
   content: String,
}
impl PendingReviewPost {
   pub fn approve(self) -> Post {
      Post { content: self.content, }
   }
}
```
1. 类似的，我们继续完成其他状态类型，这个类型一样没有 `content` 方法，因此调用也会直接编译器报错
2. 注意，我们的转换函数的参数参数是 `self`，会夺走 ownership，也就是消耗掉调用的对象，并生成新的 `PendingReviewPost` 或 `Post` 对象
3. 修改了实现方式，我们的 *main.rs* 文件也需要一些小修改，比如每次返回类型都是不同的，因此需要多次 shadow。这种我们需要在 `main` 中重新赋值 `post` 的改变，意味着这个实现不再完全遵循面向对象的状态模式：状态间的转换不再完全被封装进 `Post` 的实现中。然而我们的收获是，由于类型系统和编译时发生的类型检查，现在不可能出现无效状态，这保证了一些特定的 bug，例如显示出未发布的 post，会在程序做成产品前就被发现。
4. 我们已经看到了，虽然 ***Rust*** 能够实现面向对象的设计模式，其他模式如将状态编码进类型系统，依旧是可用的。这些模式有不同的权衡。虽然你可能很熟悉面向对象的设计模式，重新思考问题利用 ***Rust*** 的特性能够提供好处，例如在编译阶段阻止 bug。
5. ***Rust*** 中面向对象模式不总是最好的解决方案，由于一些其他面向对象的语言没有的特性，如 ownership。


# 代码
- *Rust/oop_rust_learn/src/lib.rs*
- *Rust/gui_simpel_learn/src/lib.rs*
- *Rust/gui_simpel_learn_binary/src/lib.rs*
- ***Rust/blog_lib/src/lib.rs***
- ***Rust/blog_lib_rethink/src/lib.rs***
- ***Rust/blog_bin/src/main.rs***