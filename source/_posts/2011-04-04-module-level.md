---
layout: post
title: "浅谈代码分层：构建模块化程序"
category: module
tags: module
comments: true
---

模块化的程序是怎样的程序？我们可以说一个具有明显物理结构的软件是模块化的，例如带
插件的软件，一个完整的软件由若干运行时库共同构建；也可以说一个高度面向对象的库是
模块化的，例如图形引擎OGRE；也可以说一些具有明显层次结构的代码是模块化的。

模块化的软件具有很多显而易见的好处。在开发期，一个模块化的设计有利于程序员实现，
使其在实现过程中一直保持清晰的思路，减少潜伏的BUG；而在维护期，则有利于其他程序 员的理解。

<!-- more -->
在我看来，具有良好模块设计的代码，至少分为两种形式：

-   整体设计没有层次之分，但也有独立的子模块，子模块彼此之间耦合甚少，这些子模块 构成了一个软件层，共同为上层应用提供服务；
-   整个库/软件拥有明显的层次之分，从最底层，与应用业务毫无相关的一层，到最顶层，
    完全对应用进行直接实现的那一层，每一个相对高层的软件层依赖于更底层的软件层， 逐层构建。

上述两种形式并非完全分离，在分层设计中，某一层软件层也可能由若干个独立的模块构成
。另一方面，这里也不会绝对说低层模块就完全不依赖于高层模块。这种双向依赖绝对不是 好的设计，但事实上我们本来就无法做出完美的设计。

本文将代码分层分为两大类：一是狭义上的分层，这种分层一般伴有文件形式上的表现；一 是广义上的分层，完全着眼于我们平时写的代码。

## 软件分层

软件分层一般我们可以在很多大型软件/库的结构图中看到。这些分层每一层本身就包含大
量代码。每个模块，每一个软件层都可能被实现为一个运行时库，或者其他以文件形式为 表现的东西。

### Example Android

Android是Google推出的智能手机操作系统，在其官方文档中有Android的系统架构图：

![image](/assets/res/module_level/android-architecture.jpg)

这幅图中很好地反映了上文中提到的软件层次。整个系统从底层到高层分为Linux kernel，
Libraries/Runtime，Application
Framework，Applications。最底层的Kernel可以说与应
用完全不相关，直到最上层的Applications，才提供手机诸如联系人、打电话等应用功能。

每一层中，又可能分为若干相互独立（Again，没有绝对）的模块，例如Libraries那一层 中，就包含诸如Surface
manager/SGL等模块。它们可能都依赖于Kernel，并且提供接口给 上层，但彼此独立。

### Example Compiler

在编译器实现中，也有非常明显的层次之分。这些层次可以完全按照编译原理理论来划分。 包括：

-   词法分析：将文本代码拆分为一个一个合法的单词
-   语法分析：基于 *词法分析* 得到的单词流构建语法树
-   语义分析：基于 *语法分析* 得到的语法树进行语义上的检查等
-   生成器：基于 *语义分析* 结果（可能依然是语法树）生成中间代码
-   编译器：基于 *生成器* 得到的中间代码生成目标机器上的机器代码
-   链接器：基于 *编译器* 生成的目标代码链接成最终可执行程序

**软件分层的好处之一就是对任务(task)的抽象，封装某个任务的实现细节，提供给其他 依赖模块更友好的使用接口。隔离带来的好处之一就是可轻易替换某个实现。**
例如很 多UI库隔离了渲染器的实现，在实际使用过程中，既可以使用Direct X的渲染方式，也可 以使用OpenGL的实现方式。

但正如之前所强调，凡事没有绝对，凡事也不可过度。很多时候无法保证软件层之间就是单
向依赖。而另一些时候过度的分层也导致我们的程序过于松散，效率在粘合层之间绕来绕去 而消失殆尽。

## 代码分层

如果说软件分层是从大的方面讨论，那么本节说的代码分层，则是从小处入手。而这也更是
贴近我们日常工作的地方。本节讨论的代码分层，不像软件分层那样大。每一层可能就是 百来行代码，几个接口。

### Example C中的模块组织

很多C代码写得少的C++程序员甚至对一个大型C程序中的模块组织毫无概念。这是对其他技 术接触少带来的视野狭窄的可怕结果。

在C语言的世界里，并不像某些C++教材中指出的那样，布满全局变量。当然全局变量的使
用也并不是糟糕设计的标志(goto不是魔鬼)。一个良好设计的C语言程序懂得如何去抽象、 封装模块/软件层。我们以Lua的源代码为例。

lua.h文件是暴露给Lua应用（Lua使用者）的直接信息源。接触过Lua的人都知道有个结构体
叫lua\_State。但是lua.h中并没有暴露这个结构体的实现。因为一旦暴露了实现，使用者就
可能会随意使用其结构体成员，而这并不是库设计者所希望的。 **封装数据的实现，也算 是构建模块化程序的一种方法。**

大家都知道暴露在头文件中的信息，则可能被当作该头文件所描述模块的接口描述。所以， 在C语言中任何置于头文件中的信息都需要慎重考虑。

相对的，我们可以在很多.c文件中看到很多static函数。例如lstate.c中的stack\_init。
static用于限定其修饰对象的作用域，用它去修饰某个函数，旨在告诉：这个函数仅被当前文件（
模块）使用，它仅用于本模块实现所依赖，它不是提供给模块外的接口！
**封装内部实现 ，暴露够用的接口，也是保持模块清晰的方式之一。**

良好的语言更懂得对程序员做一种良好设计的导向。但相对而言，C语言较缺乏这方面的语
言机制。在C语言中，良好的设计更依赖于程序员自己的功底。

### Example Java中的模块组织

相较而言，Java语言则提供了模块化设计的语法机制。在Java中，如同大部分语言一样，一
般一个代码文件对应于一个代码模块。而在Java中，每个文件内只能有一个public class。 public
class作为该模块的对外接口。而在模块内部，则可能有很多其他辅助实现的class
，但它们无法被外部模块访问。这是语言提供的封装机制，一种对程序员的导向。

### Example OO语言中类接口设计

无论在C++中，还是在Java中，一个类中的接口，都大致有各种访问权限。例如public、
private、protected。访问权限的加入旨在更精确地暴露模块接口，隐藏细节。

在C中较为缺乏类似的机制，但依然可以这样做。例如将结构体定义于.c文件中，将非 接口函数以static的方式实现于.c文件中。

OO语言中的这些访问权限关键字的应用尤为重要。C++新手们往往不知道哪些成员该public
，哪些该private。C++熟手们在不刨根挖底的情况下，甚至会对每个数据成员写出get/set
接口（那还不如直接public）。在public/private之间，我们需要做的唯一决策就是，哪些
数据/操作并非外部模块所需。如果外部模块不需要，甚至目前不需要，那么此刻，都不要
将其public。一个public信息少的class，往往是一个被使用者更喜欢的class。

（至于protected，则是用于继承体系之间，类之间的信息隐藏。）

### Example Lisp中的模块设计

又得提提Lisp。

基于上文，我们发现了各种划分模块、划分代码层的方式，无论是语言提供，还是程序员自 己的应用。但是如何逐个地构建这些层次呢？

Lisp中倡导了一种更能体现这种将代码分层的方式：自底而上地构建代码。这个自底而上，
自然是按照软件层的高低之分而言。这个过程就像上文举的编译原理例子一样。我们先编写
词法分析模块，该模块可能仅暴露一个接口：get-token。然后可以立马对该模块进行功能
测试。然后再编写语法分析模块，该模块也可能只暴露一个接口：parse。语法分析模块建
立于词法分析模块之上。因为我们之前已经对词法分析模块进行过测试，所以对语法分析的 测试也可以立即进行。如此下去，直至构建出整个程序。

每一个代码层都会提供若干接口给上层模块。越上层的模块中，就更贴近于最终目标。每一
层都感觉是建立在新的“语言“之上。按照这种思想，最终我们就可以构建出DSL，即Domain Specific Language。

### 分层的好处

基于以上，我们可以总结很多代码分层的好处，它们包括（但不限于）：

-   隐藏细节，提供抽象，隐藏的细节包括数据的表示（如lua\_State）、功能的实现
-   在新的一层建立更高层的“语言”
-   接口清晰，修改维护方便
-   方便开发，将软件分为若干层次，逐层实现

### 一个问题的解决

有时候，我们的软件层很难做到单向依赖。这可能是由于前期设计的失误导致，也可能确实
是情况所迫。在很多库代码中，也有现成的例子。一种解决方法就是通过回调。回调的实现
方式可以是回调函数、多态。多态的表现又可能是Listener等模式。

所有这些，主要是让底层模块不用知道高层模块。在代码层次上，它仅仅保存的是一个回调
信息，而这个信息具体是什么，则发生在运行期（话说以前给同事讲过这个）。这样就简单 避免了底层模块依赖高层模块的问题。

## END

精确地定义一个软件中有哪些模块，哪些软件层。然后再精确地定义每个模块，每个头文件
，每个类中哪些信息是提供给外部模块的，哪些信息是私有的。这些过程是设计模块化程 序的重要方式。

但需要重新强调的是，过了某个度，那又是另一种形式的糟糕设计。但其中拿捏技巧，则只 能靠实践获取。



