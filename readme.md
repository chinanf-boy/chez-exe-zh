# gwatt/chez-exe [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 ChezScheme 自足可执行文件 」

[中文](./readme.md) | [english](https://github.com/gwatt/chez-exe)

---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- repo = 'gwatt/chez-exe' -->
<!-- commit = '04b8ce8f4181919328d41d4815108fdf305104f0' -->
<!-- time = '2019-05-17' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2019-05-17 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/gwatt/chez-exe.svg
[commit]: https://github.com/gwatt/chez-exe/tree/04b8ce8f4181919328d41d4815108fdf305104f0

<!-- doc-templite END generated -->

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[hIf help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

## ChezScheme 自足可执行文件

该项目的目标是生成独立的可执行文件：一个完整的 ChezScheme 系统,并包含一个 scheme 程序。这通过将 ChezScheme 引导(boot)文件和 scheme 程序，嵌入到可执行文件中来实现。

支持系统：

- Linux
- MacOSX
- Windows

### 要求

`chez-exe` 需要一个 C 编译器，`make` 和 `ChezScheme` 来构建。在 unix 系统上，gcc 或兼容版本都没问题。而在 Windows，这需要`cl.exe`。同样，在 unix 系统上需要 GNU `make`，而在 Windows 上则需要使用 `nmake`。

### 构建

构建此项目需要 ChezScheme 的工作副本。`Chez` 不需要安装在任何特定的地方，即使在存储库内部构建也应该有效。一旦 ChezScheme 准备就绪，你可以像构建`so`一样，构建`chez-exe`：

```bash
scheme --script gen-config.ss [--prefix prefix] [--bindir bindir] \
    [--libdir libdir] [--bootpath bootpath] [--scheme scheme] \
    [...]
make [bootpath=...] [libpath=...] [incdir=...] [scheme=...]
```

在 unix 上，运行 `gen-config.ss` 将创建两个文件：`config.ss`和`make.in`，或者在 Windows 上，`config.ss`和`tools.ini`。这些文件在编译和安装时，简化了构建**chez-exe**的过程。**gen-config**能用的选项，如下面所述：

| 名           | 曰                                 |
| ------------ | ---------------------------------- |
| `--prefix`   | 用于安装，库和二进制文件的基本目录 |
| `--bindir`   | 用于安装，二进制文件的目录         |
| `--libdir`   | 用于安装，库的目录                 |
| `--bootpath` | 包含`.boot`和`scheme.h`文件的目录  |
| `--scheme`   | scheme 可执行文件的名称或命令行    |

`--bindir`和`--libdir`在 unix 和 Windows 上的表现不同。在 Unix 上，`--bindir`默认为`$prefix/bin`，和`--libdir`为`$prefix/lib`，但在 Windows 上，他们都默认为`$prefix`。另外，Unix 上，`prefix`默认值是`/usr/local`，而在 Windows 上则是`%LOCALAPPDATA%\chez-exe`。`gen-config.ss`尾随的参数会存储起来，传递给 C 编译器。

如果使用`gen-config`指定`bootpath`，则可以在调用 `make`时，从命令行中省略。不然的话，它是必需的。`bootpath`指示在哪里，可以找到`petite.boot`和`scheme.boot`。

而`libpath`，`incdir`和`scheme`都是可选项。

- `libpath` 控制在哪里，找到`kernel.o`文件。默认为`$bootpath`。
- `incdir` 控制在哪里，找到`scheme.h`。默认为`$bootpath`。
- `scheme` 控制所用可执行文件的名称，这默认为“scheme”。

一旦构建(完成)，有两个重要文件：**compile-chez-program**和`chez.a`。

**注意**：在 Windows 上构建时，请确保 MSVC 编译器和 Chez Scheme 中，使用匹配`bitsize`(位大小)。如果您看到类似的错误：

- unresolved external symbol `_Sscheme_init`
- unresolved external symbol `_Sregister_boot_file`
- unresolved external symbol `_Sbuild_heap`
- unresolved external symbol `_Sscheme_program`

仔细检查，您是否从正确的“本机工具命令提示符”，运行“make”。

`chez.a`是一个静态库，其包含来自 ChezScheme 树的`embed_target.o`和`kernel.o`。**compile-chez-program**是一个程序，它将获取一个 scheme 程序，并生成一个可执行程序，用来运行（获取）的 scheme 程序。它使用 ChezScheme 的 `compile-whole-program`，生成一个已编译的文件。该单个文件包含给定程序，以及所有可访问源库。用`(load ...)`访问的任何 scheme 文件，或是以源(库)的形式不可用的(文件)，需要与生成的可执行文件，一起分发。

### 运行：

```
    compile-chez-program [--libdirs ...] [--libexts ...] [--srcdirs ...]
        [--chez-lib-dir /path/to/chezlib] [--optimize-level 0|1|2|3]
        program-file.ss [...]
```

**compile-chez-program**理解`CHEZSCHEMELIBDIRS`和`CHEZSCHEMELIBEXTS`会与 ChezScheme 可执行文件，理解它们的方式相同。**compile-chez-program**还可以识别以下命令行参数：

- `--libdirs`（与 ChezScheme 相同）
- `--libexts`（与 ChezScheme 相同）
- `--srcdirs`
- `--optimize-level`（与 ChezScheme 相同）
- `--chez-lib-dir`

`--srcdirs`参数改变了`source-directories`参数，完全就和`--libdirs`改变了`library-directories`参数一样。其余参数的行为与 ChezScheme 可执行文件中的行为，对应相同。所有这些参数都是可选的。**compile-chez-program**假设第一个未知参数，是要编译的文件名。任何额外的参数都传递给 C 编译器。

例如：

```
compile-chez-program foo.ss -lGL -lGLU -lGLEW
```

还会链接到 OpenGL 库，通过允许 scheme 源代码，调用`(load-shared-object #f)`访问共享库，而不是单独加载(load )每个目标文件。
