# 学习依赖注入和PHP

译自/translated from ：[Learning About Dependency Injection and PHP](http://ralphschindler.com/2011/05/18/learning-about-dependency-injection-and-php)

在过去的几年里，有一些编程模式和概念增强了来自其他编程语言和开发社区的PHP开发者的信念。这些概念从MVC应用架构，多种模型技术（ActiveRecord和Data Mapper）到我们思考应用架构的方式的完全转变，如面向切面编程（aspect-oriented programming (AoP)）和事件驱动编程。或许因为PHP由企业级别运营因此增长了开发者称为<i>企业质量编程模式</i>的需求，或许是因为它够简单，因为PHP之前的进化对象模型可以使新事物变得可能。毕竟，谁不喜欢心的闪亮的东西呢？不管是什么原因，最新的概念之一（至少是过去的三年内）已经合并成了我们讨论的主题之一，那就是<i>如果管理对象依赖</i>，有趣的是，管理依赖的争论通常由方案命名，这个方案的支持者给出了解决方案：**依赖注入（dependency injection）**（这个抽象的原则实际上被称为**控制反转(Inversion of control)**）.

在任何面向对象信仰的开发者圈子中，你永远不会听到依赖注入自身是糟糕的的争论。在这些圈子中，通常认为，依赖注入是最好的实现方式。在PHP中注入对象依赖就像这样：

```php
// construction injection
$dependency = new MyRequiredDependency;
$consumer = new ThingThatRequireMyDependency($dependency);
```

基本上就是这样。除了上面的构造器注入之外，还有多个这样的变种：存储器注入，接口注入，调用时注入。这些都是将依赖注入到消耗对象中的有效的方式。最终，这里的目标是为了避免这个：

```php
class ThingThatHasAnExternalDependency {
    public __construct() {
        $this->dependency = new ARequiredDependency;
        // or
        $this->secondDependency = ARquiredDependency::getInstance();
    }
}
```

上面是一个违背了[Hollywood Principle](https://en.wikipedia.org/wiki/Inversion_of_control)的例子，它基本上说的是：“别叫我们，我们会叫你”。

然而，这不是争论的核心。4-5年以前在PHP社区它是的，但现在不是了。争论的核心不是我们应当做它，而是<i>我们怎么开始做它</i>。

这篇文章和DI容器或DI框架的错综复杂的细节或实现细节无关。它也和多种注入依赖到别的对象中的方式无关，或者是何种方式注入更好一些。事实上，这篇文章对于给你的应用使用依赖注入是否好一点也没有任何意见。这篇文章是一个对如何经营任意PHPDI框架来影响一个项目的生命周期，包括代码和开发者，构建它的团队或组织的探索。

## 在PHP中依赖管理的简要历史

知道PHP为什么如此流行很重要，毕竟，DI框架反对在PHP应用框架内部来运营。要理解PHP的流行，历史和进化，看下面的代码：

```php
// 这6行代码实际上展现了5个不同的web中心“语言”
include_once 'includes/config.php'; // 最终有一个mysql_connect()调用在这里的某个地方
include_once 'templates/header.php';
$rows = mysql_query('SELECT * FROM users');// 魔法般地使用mysql_connect()资源
foreach ($rows as $row) {
    echo '<div class="user-row"><a href="/delete-user.php" onclick="someJSFunction();">'.$row['username'].'</a></div>';
}
include_once 'templates/footer.php';
```

从一开始，我们被训练地思考我们的依赖被魔法般地管理起来。如你上面看到的，`mysql_query()`函数，由于它接受一个连接资源，但没有要求它。实际上，如果没有提供，它会使用它在PHP运行时找到的第一次打开的mysql连接。假设上面的`delete-user.php`脚本是一大批PHP脚本（我们叫做“应用”）中的一部分，注意即使这个脚本自身在靠近依赖而不是等它们被注入很重要。对所有的意图和意识，config.php，header.php，和footer.php都是这个脚本的依赖，更像别的相对于delete-user.php的脚本。要总结它，如果有一个新的依赖被这个业务逻辑需要（例如：在header和footer之间的行），它们就被续被引入到这个应用的<i>所有脚本</i>中。这很明显没有附着到[DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself)原则上。

但是，让我们回退一步，从组织的角度看看这段代码。为了达到这个，我们首先得理解在任意组织中的代码生命周期的不同阶段。对于这个例子的意图，我们假定它是从主意到产品，代码会走过下面这些阶段：开发，构建，部署，应用启动（在生产环境）。如果这是一个C/C++或java项目，代码会被编写（开发），编译（构建），被运行（通过一些启动脚本来启动，或是执行一个二进制文件），PHP还有Perl在这时，完成了所有相同的目标，但因为更少的步骤使它成为一个用于高交互的web项目的广泛流行的平台。这个同样的应用用PHP来做的话，可能在一些文本编辑器内写代码（开发），用FTP上传到生产服务器（部署），你会注意到，它既不需要构建／编译，也不需要在服务器上启动，因为Apache已经内置运行PHP了。对于所有的意图，一个简单的方便的FTP工具就是应用的生命周期的构建和部署工具。

因此，PHP是web应用的流行选择。这个流行因为PHP平台足够简单来允许两个非要重要的开发方面来合并：构建应用的想法编程现实，即便是个人新手，并且排除cruft的话，它跟随应用的整个生命周期，在PHP增长的PHP的"fun-ness"系数构建和部署应用。

由于构建应用的这种方式允许需要开发的PHP应用的激增，就有一些负面的事实需要在以后的时间里揭露。由于应用快速激增，维护就变得困难。我们给它起了个名字“意大利面代码”。对象，如果被使用了，通常包含过程方法。所以对象依赖管理对于大多数开发者来说，通常不会考虑。回头看看，也许是最原始的纯粹的想法使得开发者创建应用而不需要知道依赖是什么活着是如何去找到。在任意情况下，因为这些应用不受控制地增长，维护它们就成指数级地丢失了PHP的愉快系数。

## DI框架的简要历史

由于PHP开发者开始确认它们的[Model 1](http://en.wikipedia.org/wiki/Model_1)应用的问题，他们开始在其他编程社区寻找解决方案。这时，Java社区仍旧深深地根植在企业/软件，开发/软件引擎世界中，并且如依赖管理已经有了一些有意思的解决方案。值得一提的是，有个[Spring Framework](http://en.wikipedia.org/wiki/Spring_Framework)，它对于依赖管理的主要设施是一个组件，称作IoC容器（控制反转容器）。这个容器使用回调管理着对象创建的完整生命周期。这意味着，你不再必须使用`new`关键字（在PHP中也是这个关键字）。同时，它在实例化的时候为你接通了依赖项。这意味着，你不需要再关心依赖如何注入，通过构造器，属性或存储方法来完成它。`Spring Framework`是第一个鼓励管理知识需求来接通所有依赖的定义文件的使用者之一。来自Java社区的形式，这些定义文件被创建为XML。

可以看得到，这确实违背了使PHP如此流行的PHP哲学。PHP允许你写最少量的代码来构建应用。在Java/DI世界中，尤其是`Spring Framework`，你有一个更复杂的应用生命周期。不只是你为你的应用编码，你还需要写代码来管理代码。这称作[meta-programming](http://en.wikipedia.org/wiki/Meta-programming)。除了正在继续的这个`meta-programming`之外，你还有这个被Java平台需要的编译阶段，它通常藏在你的构建时任务中。此外，这个应用必须被部署（针对这个已经有工具了），并且（为了更好的评估）由于平台，你的应用需要被启动。无需多说，这个应用生命周期可以再轻一点，对于缺少更好的条款，对于平均水平的PHP开发者。

自那时起，一些框架突然产生了，并且支持一些依赖管理。在这个技术被PHP采用之前，它们都深深地根植于Java和.NET社区中。一个快速的google搜索会返回少量的值得注意的名字，如PicoContainer，Spring.NET，Unity，Butterfly和google-guice。这些框架由于它们试图缓解一些（DI放在开发者上，无论是通过使用反射创建定义，还是甚至添加一个以便于DI定义可以被写在代码里可以管理的注释系统的）负担而获得了流行度。

## DI和PHP

要理解PHP的具有依赖管理框架的可达性。首先应当理解的一个是在Java和.NET中与之对等的事物是如何依赖于它们各自的平台来进行确定的工作的。对于快速指引，查看这篇[博客](http://ralphschindler.com/2010/05/06/phpundamentals-series-a-background-on-statics-part-1-on-statics)上的图片。更重要的层面之一是期望的Java/.NET应用的期望的应用生命周期更为复杂。你被期望要有构建时任务。你被期望有部署任务。并且，通常，你的应用需要理解作为开发，阶段和产品之间的不同——以便于它可以调整如何有根据地运行。此外，平台自身有适当的设施协助开发者同时在开发阶段进行代码生成还有产品阶段。

PHP从不期望或促进任意类型的构建时任务的使用。PHP同时没有任意类型的内置注释支持（一种meta-programming技术），也没有任意种类的应用范围或每应用内存空间。这对于正在创建DI容器的人来说意味着什么？让我们继续探索。

### 开发时

通常来讲，任意时候你编写，修改或仅仅切换代码，你就在开发模式中，你的应用应当运行在一个开发环境中。你的应用的类，函数和文件在文件系统中的结构会在你每次点击保存的时候正确更改。依赖管理系统需要知道你的代码的一些知识以便于能够高效地开展它们的工作。这些知识通常来自于一些定义类型的形式。

这种定义可以由双手创造，由开发者的双手，有一些应用钩子在运行时生成，或通过使用一些特殊工具来生成。如果这是通过双手完成，开发者需要明确地映射多种需要备调用的函数/方法以便于注入一个专门的对象依赖。依赖越多，这种冗繁的定义工作就越多。

一个更好的方式是生成这些文件，毕竟，你写的代码，如果写得正确，就可以自己描述自己的依赖项。对于生成有两个选项，手动和自动。一个手动生成的例子是，开发者给出了一个需要最少信息的命令行工具，它需要能够转换你的代码，指出对于它自己的依赖项映射，生成一些类型的定义来在运行时使用。最少信息可能包含一些种子信息类型，如在哪里可以找到你的类或者在检测类的时候应该使用哪种过滤。有时，这些工具可能使用特殊的接口（也称作接口注入）来理解他们的目的是描述实现了刚才说的接口的类的不同的依赖项。另一种实现是利用在类和类方法上的特殊注释来描述多种需要的和选用的依赖项以及它们如何被注入。

在这个手动视线中的同样的技术也可以被用于自动实现。在自动实现中，想象一下手动实现中的命令行现在是应用自身的一项服务。当在开发模式中时，它会频繁地运行以确定代码更改是否发生了。如果改变发生了，这个服务就会重新生成依赖项定义文件以便于应用的剩余部分可以在运行时给应用利用DI容器内可用的依赖项定义。

考虑依赖管理，有一对专门对于PHP的顾虑。由于PHP是一个非共享（share-nothing）架构，没有应用级别的内存，这中定义需要在每次请求接入的时候载入，转换，并放入内存。需要追踪的依赖项树越大，依赖项定义图片的内存消耗就越大。而且，由于这个定义必须在每次请求接入时载入，如果他在一个非原生格式（意味着任意不是PHP的代码）中，那就会有明确的对于转换格式的消耗，可能是依赖项管理容器需要的机遇内存结构的XML，YAML，JSON，或者INI。更重要的是，PHP平台不追踪文件更改。所以，如果不是一些形式如用户追踪，就很难知道在开发期间有什么文件发生更改了。因此，你的依赖项管理系统，如果它采用了一个自动实现，需要在开发期间，在每次请求接入的时候重新扫描文件系统——它有他自己的重要性。

### 部署时

当代码写完，准备好发布应用到生产环境中，发布这个应用的动作称为部署。这个应用的模式现在被认为是`生产`。在生产环境中，你要确定你的代码结构是稳定的并不会再更改，因此，你的依赖项图表对于更改来说现在就是安全的。因此，没有必要再像开发环境中那样保持更新和重新生成依赖项定义文件了。

即使定义不再更改了，仍旧有在每次请求时载入这个定义是多么昂贵的考虑。理所当然的，定义的最便宜形式应当是个PHP数组或结构，这就可以载入内存。其他的文件类型如XML，YAML，JSON等在它们可用之前首先需要经过一个转换阶段。这个转换活动是昂贵的，并且可以从缓存中受益。以某种形式缓存定义可以确保在应用使用这个依赖管理容器时最小化紧接着的每次请求。

## 其他观测&审定

重要的是意识到依赖管理解决方案和它们本身，以尽可能可用的文字来说，就是完整的框架（full framework）。它们需要你同时理解他们的哲学，还有一个关于它们提供什么设施的最小的理解以便于高效地使用它们。要理解任意框架的真正福利，首先必须要知道这个框架试图解决的痛点是什么。看到了最终的框架结果而不知道它促进了什么可能会导致一个过火或直觉性地忽视它。例如，看下下面的代码（依赖管理系统的典型）：

```php
$userRepository = $dic->get('UserRepository');
```

如果你在没有完全理解正在使用的依赖注入容器的情况下遇到了这段代码，你就不会珍惜它的实用性。你可能会自己实例化`Application\Model\UserRepository`，确定的是，你还需要定位和注入数据库适配器来使用，还有载入用于数据库连接的配置文件，如果你在多个控制器行为中做了这些事情，那就有许多重复的文件范例代码需要`接通`UserRepository对象。在内部，DiC对象载入和查阅一个定义，创建对象，注入这些对象，并返回完整接通并准备就绪的请求对象。

上面的代码同样表明了两个常见的依赖管理框架的批判，同样也是对所有框架的批判。使用这种框架，你就离语言或平台自身的设备越来越远。你请求另外一个对象来创建被请求的对象给你而不是使用`new`关键字创建一个新对象。所做的这些事情将开发者从利用语言自身易于理解的API上切换到了理解框架的API。除此之外，这种类型的代码不易被IDE理解。然而一些特殊的特性可以背加入到IDE中来支持这个框架，他不能从根本上知道调用` $dic->get(..)`返回了什么对象。

## 总结

由于依赖管理框架清掉了drop-in福利，还存在一些未知的或未探索的结果的考虑。例如，如果福利是这样，所以依赖项已经管理好了，所有的开发者要做的只是配置它，那在创建类和类依赖项的时候鼓励了更深入的对象图表吗？如果是这样，冲击这些深入对象图表的性能如何，尤其是在PHP平台上。这些对象绘图的内存含义是什么，它们的速度含义是什么？此外，如果有人需要调试一个由依赖管理框架生成的对象，这容易吗？

到目前为止，要不要使用依赖管理框架是消耗和福利之间的问题。为了能够发起一个无形状的决定，一个开发者需要考虑一些坑发生的情况。首先，需要知道用／不用这个框架代码会是什么样，这会给出一个在代码层面的消耗／福利指示，它是否真正的节省了代码行数，减少了开发者的头疼？其次，需要考虑增加了多少一个开发者／团队需要学习以便理解框架的知识。最后，需要考虑实现这个新框架对应用的承载量在性能上有多大的冲击。




