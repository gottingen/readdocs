# ninja

## reference

ninja building home [ninja build](https://ninja-build.org)

ninja build manual [ninja manual](https://ninja-build.org/manual.html)

ninja github repo [github ninja](https://github.com/ninja-build/ninja)

## over view

Ninja是一个注重速度的小型构建系统。它与其他构建系统在两个主要方面不同：
1. 它被设计为使其输入文件由更高级别的构建系统生成。
2. 被设计为尽可能快地运行构建。

## Why yet another build system

在其他构建系统是高级语言的地方，Ninja旨在成为代理程序。

ninja是对人类来说，可读性比较好，但是手写起来并不是很方便。

## Introduction

Ninja是另一个构建系统。它以文件（通常是源代码和输出可执行文件）的相互依赖性为输入，并快速地构建它们。

Ninja加入了其他构建系统。它的独特目标是要快速。它源于我在[Chromium](http://neugierig.org/software/chromium/notes/2011/02/ninja.html)浏览器项目中的工作，该项目具有30,000多个源文件，并且其他构建系统（
包括从自定义非递归Makefile构建的一个）在更改一个文件后将需要十秒钟才能开始构建。Ninja不到一秒钟。

## Philosophical overview(设计理念)

在其他构建系统是高级语言的地方，Ninja的目标是成为粘合剂。

构建系统在需要做出决策时会变慢。当你处于编辑-编译循环中时，你希望它尽可能快地运行-希望构建系统完成所需的最少工作，以找出需要立即构建的内容。

Ninja包含描述任意依赖关系图所需的最重要功能。由于缺乏语法，因此无法表达复杂的决定。

相反，Ninja旨在与生成其输入文件的单独程序一起使用。生成器程序（如在autotools项目中找到的./configure一样）可以分析系统依赖关系并预先做出
尽可能多的决定，从而使增量构建保持快速。超越自动工具，甚至是诸如“我应该使用哪个编译器标志？”之类的构建时决策。或“我应该构建调试或发布模式的
二进制文件吗？”.ninja文件生成器的任务。

## Design goals

以下是Ninja设计的目标：

* 即使是非常大的项目，也可以非常快速的增量构建。

* 关于如何构建代码的政策很少。对于应该如何构建代码，不同的项目和更高级别的构建系统有不同的设置。例如，在源码目录构建对象，还是所有
的构建输出都应进入单独的目录？是否存在构建项目的可分发软件包的“软件包”规则？通过尝试允许实施而不是选择来回避这些决定，即使这会导致更多的冗
长。

* 获得正确的依赖关系，以及在某些情况下难以正确使用Makefile的情况（例如，输出需要在生成它们的命令行上隐式依赖；要生成C源代码，你需要使用
gcc的-M标志用于标头依赖）。

* 当方便和速度冲突时，最好选择速度。

 
一些明确的非目标：
* 手动编写构建文件的便捷语法。你应该使用其他程序生成Ninja文件。这就是我们可以回避许多政策决定的方式。
* 内置规则。 Ninja没有开箱即用的规则，例如编译C代码。
* 构建时的构建时定制。选项属于生成Ninja文件的程序。
* 建立时间的决策能力，例如条件或搜索路径。

重申一下，Ninja比其他构建系统要快，因为它非常简单。创建项目的.ninja文件时，你必须确切告诉Ninja该怎么做。

## Comparison to Make

Ninja的精髓和功能与Make最接近，它依赖于文件时间戳之间的简单依赖关系。

但从根本上说，make具有很多功能：后缀规则，函数，内置规则，例如在构建源代码时搜索RCS文件。 Make的语言是专为人类编写的。许多项目发现单独解决
它们的构建问题就足够了。

相反，Ninja几乎没有功能。在构建Ninja输入文件最复杂的基础上，获得正确构建所需的文件。Ninja本身不太可能对大多数项目有用。

以下是Ninja添加到Make中的一些功能。 （这些功能通常可以使用更复杂的Makefile来实现，但它们本身并不是make的一部分。）

* Ninja对在构建时发现额外的依赖项提供了特殊的支持，从而使获取针对C/C++代码的标头依赖项变得容易。

* 生成边界可能有多个输出。

* 输出隐式依赖于用于生成它们的命令行，这意味着更改例如编译标志将导致输出重建。

* 在运行依赖输出目录的命令之前，总是会隐式创建输出目录。

* 规则可以提供正在运行的命令的简短说明，因此你可以打印例如CC foo.o，而不是在构建时使用较长的命令行。

* 默认情况下，构建始终并行运行，具体取决于系统具有的CPU数量。未指定的构建依赖项将导致错误的构建。

* 命令输出始终被缓冲。这意味着并行运行的命令不会交错输出，当命令失败时，我们可以在导致失败的完整命令行旁边打印其失败输出。

## Using Ninja for your project

Ninja当前可在类似Unix的系统和Windows上运行。它在Linux上测试最多（并且性能最高），但在Mac OS X和FreeBSD上运行良好。

如果你的项目很小，那么Ninja的速度影响可能不会引起注意。 （但是，即使对于小型项目，有时Ninja的有限语法也会强制采用更简单的构建规则，从而加
快构建速度。）另一种说法是，如果你对项目的编辑-编译周期感到满意，那么Ninja帮不上忙。

还有许多其他构建系统比Ninja本身更具用户友好性或功能性。对于某些建议，Ninja作者发现[tup](http://gittup.org/tup)构建系统对Ninja的设计
具有影响力，并认为[redo](https://github.com/apenwarr/redo)的设计非常聪明。

Ninja的好处来自将其与更智能的元构建系统结合使用。

[gn](https://gn.googlesource.com/gn/)
    
    
    元生成系统用于生成Google Chrome浏览器和相关项目（v8，node.js）以及Google Fuchsia的生成文件。 gn可以为Chrome支持的所有平台生成
    Ninja文件。

[cmake](https://cmake.org/)

    
    从CMake 2.8.8版开始，可以广泛使用的元构建系统可以在Linux上生成Ninja文件。较新版本的CMake也支持在Windows和Mac OS X上生成Ninja文
    件。
    
[other](https://github.com/ninja-build/ninja/wiki/List-of-generators-producing-ninja-build-files)

    
    Ninja应该完全适合其他预构建之类的元构建软件。如果你从事这项工作，请告诉我们！
    
## Running Ninja

运行Ninja。默认情况下，它将在当前目录中查找名为build.ninja的文件，并构建所有过期的目标。你可以指定要构建的目标（文件）作为命令行参数。

还有一个特殊的语法target ^，用于将目标指定为某些规则的第一个输出，该规则包含你在命令行中放置的源（如果存在）。例如，如果你将目标指定为
foo.c ^，则将构建foo.o（假设你的构建文件中包含这些目标）。

ninja -h打印帮助输出。Ninja的许多旗帜故意与Make的旗帜匹配；例如ninja -C build -j 20进入build目录并并行运行20个build命令。 （请注意，
Ninja始终默认并行运行命令，因此通常不需要传递-j。）

## Environment variables

Ninja支持一个环境变量来控制其行为：NINJA_STATUS，在运行规则之前打印进度状态。


有几个占位符：

* %s 起始边的数量。
* %t 完成构建必须运行的边总数。
* %p 起始边的百分比。
* %r 当前运行的边数。
* %u 要开始的剩余边数。
* %f 已完成的边缘数量。
* %o 每秒完成边缘的总速率。
* %c 当前每秒完成边缘的速率（由-j或其缺省值指定的构建的平均值）。
* %e 经过的时间（以秒为单位）。 （自Ninja 1.2起可用。）
* %% 一个普通的％字符。


默认进度状态为“ [％f /％t]”（请注意要与构建规则分开的尾随空格）。可能的进度状态的另一个示例可以是“ [％u /％r /％f]”。

## Extra tools

Ninja命令行上的-t标志运行一些我们发现在Ninja开发过程中有用的工具。当前的工具是：

* query 转储给定目标的输入和输出。

* browse 在Web浏览器中浏览依赖关系图。单击文件可将视图聚焦在该文件上，显示输入和输出。此功能需要安装Python。默认情况下，使用端口8000，将
打开Web浏览器。可以如下更改：
    
   
    ninja -t browse --port=8000 --no-browser mytarget
    
* graph 使用自动图形布局工具graphviz使用的语法输出文件。像这样使用它：

    
    ninja -t graph mytarget | dot -Tpng -ograph.png
    
在Ninja源代码树中，ninja graph.png会为Ninja本身生成一个图像。如果没有给出目标，则为所有根目标生成一个图形。

* targets 按规则或按深度输出目标列表。如果像ninja -ttargets规则名称那样使用，它将使用要构建的给定规则打印目标列表。如果没有给出规则，它
将打印源文件（图形的叶子）。如果像Ninja-t目标深度数字那样使用，它将以深度优先的方式打印目标列表，从根目标（没有输出的目标）开始。缩进用于标
记依赖关系。如果深度为零，则打印所有目标。如果未提供任何参数，则假定ninja -t目标深度为1。在此模式下，目标可能会列出多次。如果像这样使用
ninja -t target，它将打印所有可用的目标而不会缩进，并且比深度模式要快。

* commands 在给定目标列表的情况下，打印命令列表，如果按顺序执行，则可以假定所有输出文件已过时，用于重建那些目标。

* clean 删除内置文件。默认情况下，它将删除除生成器创建的文件以外的所有构建文件。添加-g标志还会删除由生成器创建的构建文件（请参见生成器属性
的规则参考）。其他参数是目标，它删除给定的目标并递归地为它们建立的所有文件。如果像ninja -t clean -r规则一样使用，它将删除使用给定规则构建
的所有文件。在图形中创建但未引用的文件不会被删除。该工具考虑了-v和-n选项（请注意，-n表示-v）。

* cleandead 删除清单中不再存在的由先前版本生成的文件。自Ninja 1.10起可用。

* compdb 给定一个规则列表，每个规则应该是C族语言编译器规则，其第一个输入是源文件的名称，并在标准输出上以Clang工具接口期望的JSON格式打印
一个编译数据库。自Ninja 1.2起可用。

* deps 显示存储在.ninja_deps文件中的所有依赖项。指定目标后，仅显示目标的依赖项。自Ninja 1.4起可用。

* recompact 重新压缩.ninja_deps文件。自Ninja 1.4起可用。

* restat 更新.ninja_log文件中所有记录的文件修改时间戳。自Ninja 1.10起可用。

* rules 输出所有规则的列表（如果有一个规则，最后输出其描述）。它可以用来知道将哪个规则名称传递给ninja -t目标规则名称或ninja -t compdb。

## Writing your own Ninja files

本手册的其余部分仅在你自己构造Ninja文件时才有用：例如，如果你正在编写元构建系统或支持新语言。

### Conceptual overview

Ninja会评估文件之间的依赖关系图，并运行所需的任何命令以使构建目标保持最新状态（由文件修改时间决定）。如果你熟悉Make，Ninja就非常相似。

生成文件（默认名称：build.ninja）提供规则列表-较长命令的短名称，例如如何运行编译器-以及列出如何使用规则生成文件的生成语句列表适用的规则向
哪个输入产生哪个输出。

从概念上讲，构建语句描述项目的依赖关系图，而规则语句描述如何沿图的给定边缘生成文件。

### Syntax example

这是一个基本的.ninja文件，用于演示大多数语法。以下部分将作为示例。


    cflags = -Wall
    
    rule cc
      command = gcc $cflags -c $in -o $out
    
    build foo.o: cc foo.c


### Variables


尽管不存在手工编写方便的目的，但为了保持构建文件的可读性（可调试性），Ninja也支持声明较短的字符串可重用名称。如下声明

    
    cflags = -g

可以在等号的右侧使用，用美元号取消引用，如下所示：

    rule cc
      command = gcc $cflags -c $in -o $out
  
也可以使用大括号（例如$ {in}）来引用变量。

变量最好称为“绑定”，因为给定的变量不能被更改，只能被遮盖。本文档后面将详细介绍隐式替换的工作原理。

### Rules

规则声明命令行的简称。它们以由rule关键字和规则名称组成的一行开头。然后是一组缩进的变量=值行。

上面的基本示例声明了一个名为cc的新规则以及要运行的命令。在规则的上下文中，command变量定义要运行的命令，$ in扩展为命令的输入文件列表
（foo.c），$ out扩展为命令的输出文件（foo.o）。参考中提供了特殊变量的完整列表。

### Build statements

生成语句声明输入文件和输出文件之间的关系。它们以build关键字开头，格式为build输出：rulename输入。这样的声明表明所有输出文件都来自输入文件。
当输出文件丢失或输入更改时，Ninja将运行规则以重新生成输出。

上面的基本示例描述了如何使用cc规则构建foo.o。

在构建块的范围内（包括对其关联规则的评估），变量$ in是输入列表，变量$ out是输出列表。

生成语句后可以紧跟一组缩进的键=值对，这很像一条规则。在评估命令中的变量时，这些变量将隐藏所有变量。例如：

    
    cflags = -Wall -Werror
    rule cc
      command = gcc $cflags -c $in -o $out
    
    # If left unspecified, builds get the outer $cflags.
    build foo.o: cc foo.c
    
    # But you can shadow variables like cflags for a particular build.
    build special.o: cc special.c
      cflags = -Wall
    
    # The variable was only shadowed for the scope of special.o;
    # Subsequent build lines get the outer (original) cflags.
    build bar.o: cc bar.c

有关范围界定如何工作的更多讨论，请查看[the reference](https://ninja-build.org/manual.html#ref_scope)

如果你需要从build语句传递到规则的更复杂的信息（例如，如果规则需要“第一个输入的文件扩展名”），则将其作为额外的变量传递，就像上面如何传递
cflags一样。

如果将顶级Ninja文件指定为任何构建语句的输出，并且该文件已过期，则Ninja将在构建用户请求的目标之前重建并重新加载它。

### Generating Ninja files from code

Ninja发行版中的misc / ninja_syntax.py是一个微型Python模块，可帮助生成Ninja文件。它允许你进行Python调用，例如ninja.rule
（name ='foo'，command ='bar'，depfile ='$ out.d'），它将生成适当的语法。如果有用，请随时将其内联到项目的构建系统中。

## More details

### The phony rule

特殊的规则名称phony可用于为其他目标创建别名。例如：
    
    build foo: phony some/file/in/a/faraway/subdir/foo

这使得Ninja foo 建立更长的路径。语义上，phony规则等效于一条普通命令，该命令不执行任何操作，但是phony规则经过特殊处理，它们在运行时
不会打印（请参见下文），也不会对作为部分打印的命令计数有所贡献构建过程。

phony还可以用于为构建时可能不存在的文件创建虚拟目标。如果编写的phony build语句没有任何依赖关系，则目标（如果目标不存在）将被视为过时的。
如果没有伪造的构建声明，则Ninja将报告错误，如果该文件不存在并且是构建所必需的。

要创建一个永不重建的规则，请使用不带任何输入的构建规则：

    rule touch
      command = touch $out
    build file_that_always_exists.dummy: touch
    build dummy_target_to_follow_a_pattern: phony file_that_always_exists.dummy

### Default target statements

默认情况下，如果在命令行上未指定目标，Ninja将构建所有未在其他位置命名为输入的输出。你可以使用默认目标语句覆盖此行为。如果在命令行上未指定
任何目标语句，则默认的target语句将使Ninja仅构建给定的输出文件子集。

默认目标语句以default关键字开头，格式为默认目标。默认的目标语句必须出现在将目标声明为输出文件的build语句之后。它们是累积的，因此可以使用
多个语句来扩展默认目标的列表。例如：

    default foo bar
    default baz
默认情况下，这会使Ninja构建foo，bar和baz目标。

### The Ninja log

对于每个生成的文件，Ninja都会保留用于生成该文件的命令的日志。使用该日志，Ninja可以知道何时使用不同于构建文件指定的命令行（即，更改了命令
行）构建了现有输出，并且知道如何重建该文件。

日志文件保存在名为.ninja_log的文件的生成根目录中。如果在最外层范围提供名为builddir的变量，则.ninja_log将保留在该目录中。

### Version compatibility

Available since Ninja 1.2.

Ninja版本标签遵循标准的major.minor.patch格式，其中主版本在向后不兼容的语法/行为更改时增加，次版本在新行为方面增加。你的build.ninja可能会
声明一个名为ninja_required_version的变量，该变量声明使用生成的文件所需的最低Ninja版本。例如，


    ninja_required_version = 1.1

声明构建文件依赖于Ninja 1.1中引入的某些功能（可能是pool语法），并且必须使用Ninja 1.1或更高版本进行构建。与其他Ninja变量不同，解析时会立即
检查此版本要求，因此最好将其放在构建文件的顶部。

如果Ninja的主要版本和ninja_required_version不匹配，Ninja就会发出警告。尚未进行重大版本更改，因此很难预测可能需要的行为。

### C/C++ header dependencies

为了获得C / C ++标头依赖项（或以类似方式工作的任何其他构建依赖项），正确的Ninja具有一些额外的功能。

标头的问题在于，给定源文件所依赖的文件的完整列表只能由编译器发现：不同的预处理器定义并包含路径会导致使用不同的文件。一些编译器可以在构建时
发出此信息，而Ninja可以使用该信息来完善其依赖关系。

考虑：如果从未编译过该文件，则无论如何都必须对其进行构建，从而产生标头依赖关系。如果以后修改了任何文件（即使以更改其依赖的标头的方式），修
改也会导致重新生成，从而使依赖关系保持最新。

加载这些特殊的依赖项时，Ninja会隐式添加额外的构建边界，这样，如果缺少列出的依赖项，则不会出错。这样，你就可以删除头文件并进行重建，而不会
因缺少输入而导致构建中止。

#### depfile

gcc（以及其他类似clang的编译器）支持使用Makefile的语法发出依赖项信息。 （可以使用任何可以这种形式编写依赖关系的命令，而不仅仅是gcc。）

将这些信息带入Ninja需要联合作业。在Ninja方面，构建中的depfile属性必须指向写入此数据的路径。 （Ninja仅支持编译器发出的Makefile语法的有限子集。）
然后，该命令必须知道将依赖项写入depfile路径。在以下示例中使用它：

    rule cc
      depfile = $out.d
      command = gcc -MD -MF $out.d [other gcc flags here]
  
gcc的-MD标志告诉它输出标头依赖性，而-MF标志告诉它在哪里写它们。

##### deps
(Available since Ninja 1.3.)

事实证明，对于大型项目（尤其是在文件系统速度缓慢的Windows上），在启动时加载这些依赖项文件的速度很慢。

Ninja 1.3可以在生成依赖项后立即对其进行处理，并将相同信息的压缩形式保存在Ninja内部数据库中。

Ninja以两种形式支持此处理。

1. deps = gcc指定该工具以Makefiles的形式输出gcc样式的依赖项。将其添加到上面的示例中将使Ninja在编译完成后立即处理depfile，然后删除.d文
件（仅用作临时文件）。
2. deps = msvc指定该工具以Visual Studio编译器的/ showIncludes标志产生的形式输出标头依赖项。简而言之，这意味着该工具将特殊格式的行输出
到其stdout。Ninja然后从显示的输出中过滤这些行。不需要depfile属性，但是在头文件路径前面的本地化字符串。例如，msvc_deps_prefix =注意：包
括文件：对于英文Visual Studio（默认）。应该是全局定义的。

    
    msvc_deps_prefix = Note: including file:
    rule cc
      deps = msvc
      command = cl /showIncludes -c $in /Fo$out
      
 如果include目录指令使用绝对路径，则你的depfile可能会混合使用相对路径和绝对路径。其他构建规则使用的路径需要完全匹配。因此，在这种情况下，
 建议使用相对路径。
 
 ### Pools
 
 Available since Ninja 1.1.
 
 pool 允许你分配一个或多个规则，或者限制有限数量的并发作业，这些并发作业的限制比默认并行性严格。
 
 例如，这对于限制特定的昂贵规则（例如大型可执行文件的链接步骤）或限制特定的构建语句（你知道它们在并行运行时性能较差）很有用。
 
 每个pool都有一个深度变量，该深度变量在构建文件中指定。然后在规则或生成语句中使用pool变量引用该pool。
 
 无论你指定什么pool，Ninja运行的并发作业绝不会超过默认的并行性或命令行上指定的作业数量（使用-j）。
 
    # No more than 4 links at a time.
    pool link_pool
      depth = 4
    
    # No more than 1 heavy object at a time.
    pool heavy_object_pool
      depth = 1
    
    rule link
      ...
      pool = link_pool
    
    rule cc
      ...
    
    # The link_pool is used here. Only 4 links will run concurrently.
    build foo.exe: link input.obj
    
    # A build statement can be exempted from its rule's pool by setting an
    # empty pool. This effectively puts the build statement back into the default
    # pool, which has infinite depth.
    build other.exe: link input.obj
      pool =
    
    # A build statement can specify a pool directly.
    # Only one of these builds will run at a time.
    build heavy_object1.obj: cc heavy_obj1.cc
      pool = heavy_object_pool
    build heavy_object2.obj: cc heavy_obj2.cc
      pool = heavy_object_pool
      
  #### The console pool
  
  Available since Ninja 1.5.

存在一个名为console的预定义pool，深度为1。它具有特殊的属性，即pool中的任何任务都可以直接访问提供给Ninja的标准输入，输出和错误流，这些流
通常连接到用户控制台。 （因此命名），但可以重定向。这对于交互式任务或长时间运行的任务很有用，这些任务会在控制台上产生状态更新（
例如测试套件）。

在控制台pool中的任务运行时，Ninja的常规输出（例如进度状态和并发任务的输出）将被缓冲直到完成。

## Ninja file reference

文件是一系列声明。声明可以是以下之一：

1. 规则声明，以规则规则名开头，然后有一系列定义变量的缩进线。
2. 生成边，类似于生成output1 output2：规则名input1 input2。隐式依赖可以在最后加上|。依赖1依赖2。只能在||处附加仅顺序的依存关系依赖1依
赖2。 （请参阅有关依赖项类型的参考。） 隐式输出（自Ninja 1.7起可用）可以在：之前加上| |。 output1 output2并且不出现在$ out中。 （请参
阅有关输出类型的参考。）

3. 变量声明，看起来像variable = value。
4. 默认目标语句，看起来像默认target1 target2。
5. 对更多文件的引用，这些文件看起来像subninja路径或include路径。
6. pool声明，看起来像 pool poolname。

### Lexical syntax

只要Ninja关心的字节（如路径中的斜杠）是ASCII，Ninja大部分都将编码无关。这意味着UTF-8或ISO-8859-1输入文件应该可以工作。

注释以＃开头，并扩展到该行的末尾。

换行符很重要。诸如build foo bar之类的语句是一组以空格分隔的令牌，它们以换行符结尾。令牌中的换行符和空格必须转义。

只有一个转义字符$，并且具有以下行为：

***$ 跟换行符***

转义换行符（在换行符处继续当前行）。

***$ 文本***

变量

***${varname}***

$varname的另一种语法。

***$ 空格***

空格。 （这仅在路径列表中是必需的，否则路径之间会用空格分隔文件名。请参见下文。）

***$:***
冒号。 （这仅在构建行中是必需的，否则，冒号将终止输出列表。）

***$$***
    
字符$

首先将build或default语句解析为以空格分隔的文件名列表，然后扩展每个名称。这意味着变量中的空格将导致扩展文件名中出现空格。

    spaced = foo bar
    build $spaced/baz other$ file: ...
    # The above build line has two outputs: "foo bar/baz" and "other file".

在name = value语句中，始终删除值开头的空格。在行继续之后的行开头的空白也被剥离。

    two_words_with_one_space = foo $
        bar
    one_word_with_no_space = foo$
        bar
其他空格仅在行首时才有意义。如果缩进的行比上一行缩进，则该行被视为其父级范围的一部分；如果缩进小于前一个缩进，则会关闭前一个作用域。

### Top-level variables

在最外层文件范围中声明时，两个变量很重要。

builddir:
   
    一些Ninja输出文件的目录。请参阅构建日志的讨论。 （你也可以在此目录中存储其他构建输出。）
    
ninja_required_version:
    
    构建所需的最低Ninja版本。
    
### Rule variables

规则块包含影响规则处理的key = value 声明的列表。这是特殊键的完整列表。

* command
    
    
    命令行运行。每个规则可能只有一个命令声明。有关引用和执行多个命令的更多详细信息，请参见下一部分。
 
 * depfile
 
    
    包含额外隐式依赖关系的可选Makefile的路径（请参阅有关依赖关系类型的参考）。明确支持C / C ++标头依赖性

* deps

    
    自Ninja 1.3。起可用）（如果存在），必须是gcc或msvc之一以指定特殊的依赖处理。查看完整的讨论。生成的数据库以.ninja_deps的形式存储在
    builddir中，请参见builddir的讨论。
    
* msvc_deps_prefix

    
    自Ninja 1.5起可用。）定义了应从msvc的/ showIncludes输出中删除的字符串。仅当deps = msvc且不使用英语Visual Studio版本时才需要。
    
* description
    
    
    该命令的简短说明，用于在命令运行时对其进行漂亮打印。 -v标志控制是打印完整命令还是它的描述。如果命令失败，则将始终在命令输出之前打印完
    整的命令行。
    
* dyndep

    
    （自Ninja 1.10起可用。）仅在build语句上使用。如果存在，则必须命名build语句输入之一。动态发现的依赖项信息将从文件中加载。有关详细信
    息，请参见动态依赖项部分。
    
* generator

    
    如果存在，则指定该规则用于重新调用生成器程序。使用生成器规则构建的文件有两种特殊处理方式：首先，如果命令行更改，它们将不会重建。其次，
    默认情况下不会清除它们。
    
* in

    
    引用此规则的，以空格分隔的文件列表，这些文件作为构建行的输入提供，如果在命令中出现，则用shell引用。 （$ in仅出于方便起见而提供；如果
    你需要此文件列表的某些子集或变体，只需使用该列表构造一个新变量，然后使用它即可。）
    
* in_newline

    
    与$ in相同，只是多个输入之间用换行符而不是空格分隔。 （用于$ rspfile_content；这可解决MSVC链接器中的一个错误，该错误使用固定大小
    的缓冲区来处理输入。）

* out
    
    
    用空格分隔的文件列表，这些文件作为引用此规则的构建行的输出提供，如果在命令中出现，则用shell引用。
    
* restat

    
    如果存在，则使Ninja在执行命令后重新统计命令的输出。命令的修改时间未更改的每个输出都将被视为不需要构建。这可能会导致输出的反向依赖性从待
    处理的构建操作列表中删除。
    
* rspfile, rspfile_content


    如果同时存在（两者），Ninja将使用给定命令的响应文件，即在调用命令之前将所选字符串（rspfile_content）写入给定文件（rspfile），并在命
    令成功执行后删除该文件。
    
    这在Windows OS上特别有用，在Windows OS中，命令行的最大长度受到限制，而必须使用响应文件。
    
    在以下示例中使用它：
    
    rule link
      command = link.exe /OUT$out [usual link flags here] @$out.rsp
      rspfile = $out.rsp
      rspfile_content = $in
    
    build myapp.exe: link a.obj b.obj [possibly many other .obj files]
    
解释命令变量

从根本上讲，命令行在Unix和Windows上的行为有所不同。

在Unix上，命令是参数数组。 Ninja命令变量直接传递给sh -c，然后sh -c负责将该字符串解释为argv数组。因此，引用规则是外壳程序的规则，你可以
使用所有常规外壳程序运算符，例如&&链接多个命令，或使用VAR = value cmd设置环境变量。

在Windows上，命令是字符串，因此Ninja将命令字符串直接传递给CreateProcess。 （在简单执行编译器的常见情况下，这意味着较少的开销。）因此，
引用规则由所调用的程序确定，在Windows上通常由C库提供。如果你需要命令的外壳解释（例如使用&&链接多个命令），请通过在命令前面加上cmd / c来
使该命令执行Windows Shell。Ninja可能会显示“无效参数”错误，该错误通常表明已超过命令行长度。

### Build outputs

有两种类型的构建输出，它们完全不同。

* 显式输出，如构建行中所列。这些可以作为规则中的$ out变量使用。
  
  这是用于例如输出的标准形式。编译命令的目标文件。
  
* 隐式输出，如语法| build1的：之前的out1 out2 +（自Ninja 1.7起可用）。语义与显式输出相同，唯一的区别是隐式输出未显示在$ out变量中。
  
  这是用于表达未在命令的命令行中显示的输出。
  
### Build dependencies

有三种类型的构建依赖项，它们有细微不同。

1. 显式依赖项，如构建行中所列。这些可以作为规则中的$ in变量使用。这些文件中的更改将导致重新生成输出。如果这些文件丢失并且Ninja不知道如何
构建它们，则构建将中止。这是要使用的依赖的标准形式，例如用于编译命令的源文件。

2. 隐式依赖关系，可以从规则的depfile属性中获取，也可以从语法|在构建行末尾的dep1 dep2。语义与显式依赖项相同，唯一的区别是隐式依赖项不会
显示在$in变量中.这是用于表达未显示在命令的命令行中的依赖项；例如，对于运行脚本的规则，脚本本身应为隐式依赖项，因为对脚本的更改应导致重新生
成输出。请注意，通过depfiles加载的依赖项的语义略有不同，如规则参考中所述。

3. 仅顺序依赖性，用语法||表示在构建行末尾的dep1 dep2。如果这些内容已过期，则直到生成它们后才重新构建输出，但是仅顺序依赖项的更改不会导致
重新构建输出。仅顺序依赖关系对于引导仅在构建期间发现的依赖关系很有用：例如，在开始后续的编译步骤之前生成头文件。 （一旦在编译中使用了标头，
则生成的依赖项文件将表示隐式依赖项。）

文件路径按原样进行比较，这意味着Ninja认为指向同一文件的绝对路径和相对路径是不同的。

### Variable expansion
变量在路径中（在build或default语句中）以及name = value语句的右侧扩展。

对name = value语句求值时，其右侧将立即扩展（根据以下作用域规则），然后，由于扩展，$name会扩展为静态字符串。永远不会需要对值进行
“double-escape”以防止其两次扩展。

所有变量在解析时都会立即展开，但有一个重要的例外：规则块中的变量在使用规则时（而不是在声明时）扩展。在以下示例中，演示规则将打印
“this is a demo of bar”。

    
    rule demo
      command = echo "this is a demo of $foo"
    
    build out: demo
      foo = bar

### Evaluation and scoping

全局变量声明的作用域是它们所在的文件。

规则声明的范围也仅限于它们所在的文件。（自Ninja 1.6起可用）

subninja关键字（用于包括另一个.ninja文件）引入了一个新范围。包含的子Ninja文件可以使用父文件中的变量和规则，并在文件范围内隐藏它们的值，但
不会影响父文件中变量的值。

要在当前作用域中包含另一个.ninja文件，就像C #include语句一样，请使用include而不是subninja。

在构建块中缩进的变量声明的作用域为该构建块。在构建块（或使用规则）中扩展的变量的完整查找顺序为：

1. 特殊的内置变量（$in，$out）。
2. 来自构建块的构建级别变量。
3. 规则块中的规则级变量（即$ command）。 （从上面有关扩展的讨论中注意到，这些扩展是“后期”扩展的，并且可以使用范围内的绑定，例如$ in。）
4. 构建行所在文件中的文件级变量。
5. 使用subninja关键字包含该文件的文件中的变量。

## Dynamic Dependencies

Available since Ninja 1.10.

某些用例要求在构建过程中从源文件内容中动态发现隐式依赖关系信息，以便在首次运行时正确构建（例如，Fortran模块依赖关系）。这与标头依赖性不同，
标头依赖性仅在第二次运行时才需要，以后才能正确重建。生成语句可能具有dyndep绑定，命名其输入之一来指定必须从文件中加载动态依赖项信息。例如：
    
    build out: ... || foo
      dyndep = foo
    build foo: ...
这指定文件foo是一个dyndep文件。因为它是输入，所以在构建foo之前永远无法执行out的build语句。 foo完​​成后，Ninja将读取它以加载动态发现的依
赖关系信息以供输出。这可能包括其他隐式输入和/或输出。 Ninja将相应地更新构建图，并且构建将继续进行，就像该信息最初是已知的一样。

### Dyndep file reference

dyndep绑定指定的文件使用与ninja构建文件相同的词汇语法，并具有以下布局。

1. 版本号，格式为<major> [。<minor>] [<suffix>]：

    
    ninja_dyndep_version = 1
    
当前版本号必须始终为1或1.0，但可以具有任意后缀。

2. 一个或多个以下形式的构建语句：

    
    build out | imp-outs... : dyndep | imp-ins...
 
 每个语句必须精确地指定一个显式输出，并且必须使用规则名称dyndep。 |出口...和| imp-ins ...部分是可选的。
    
3. 每个构建语句上的一个可选的restat变量绑定。

dyndep文件中的build语句必须具有一对一的对应关系才能在Ninja构建文件中以dyndep绑定命名dyndep文件的构建语句。不能省略dyndep构建语句，也可以
不指定额外的构建语句。

### Dyndep Examples

Dyndep Examples

考虑一个提供模块foo.mod（编译的隐式输出）的Fortran源文件foo.f90，以及另一个使用模块的源文件bar.f90（编译的隐式输入）。为了确保bar.f90
永远不会在foo.f90之前进行编译，并且bar.f90在foo.mod发生更改时重新进行编译，必须在我们编译任何源代码之前先发现此隐式依赖性。我们可以如下
实现：

    
    rule f95
      command = f95 -o $out -c $in
    rule fscan
      command = fscan -o $out $in
    
    build foobar.dd: fscan foo.f90 bar.f90
    
    build foo.o: f95 foo.f90 || foobar.dd
      dyndep = foobar.dd
    build bar.o: f95 bar.f90 || foobar.dd
      dyndep = foobar.dd
  
 在此示例中，仅顺序依赖项确保在任何一个源编译之前生成foobar.dd。假设的fscan工具扫描源文件，假设每个文件都将被编译为同名的.o，然后将
 foobar.dd写入以下内容：
 
    ninja_dyndep_version = 1
    build foo.o | foo.mod: dyndep
    build bar.o: dyndep |  foo.mod


Ninja将加载此文件以将foo.mod添加为foo.o的隐式输出和bar.o的隐式输入。这样可以确保始终按正确的顺序编译Fortran源，并在需要时重新编译。

#### Tarball Extraction

考虑我们要提取的tarball foo.tar。提取时间可以用foo.tar.stamp文件记录，以便在tarball更改时重复提取，但是如果缺少任何输出，我们也想重新
提取。但是，输出列表取决于tarball的内容，不能在Ninja生成文件中明确说明。我们可以如下实现：

    
    rule untar
      command = tar xf $in && touch $out
    rule scantar
      command = scantar --stamp=$stamp --dd=$out $in
    build foo.tar.dd: scantar foo.tar
      stamp = foo.tar.stamp
    build foo.tar.stamp: untar foo.tar || foo.tar.dd
      dyndep = foo.tar.dd
在此示例中，仅订购依赖项确保foo.tar.dd在压缩包提取之前就已构建。假设的scantar工具将读取tarball（例如，通过tar tf），然后将foo.tar.dd写
入以下内容：

    
    ninja_dyndep_version = 1
    build foo.tar.stamp | file1.txt file2.txt : dyndep
      restat = 1

Ninja将加载此文件，以将file1.txt和file2.txt添加为foo.tar.stamp的隐式输出，并将build语句标记为restat。在将来的版本中，如果缺少任何隐
式输出，则将再次提取压缩包。 restat绑定告诉Ninja忍受这样一个事实，即隐式输出的修改时间可能不比tarball本身新（避免在每次构建时重新提取）。

