---
title: 将 Python 2 代码迁移到 Python 3
m_slug: python2 to python3
author: 布雷特·坎农
categories:
  - python
type: post
date: 2019-12-02T20:39:21.455Z
description: >-
  Python 3 是 Python 的未来，但 Python 2 仍处于活跃使用阶段，最好让您的项目在两个主要版本的Python
  上都可用.本指南旨在帮助您了解如何最好地同时支持 Python 2 和 3.


  如果您希望迁移扩展模块而不是纯 Python 代码，请参阅 将扩展模块移植到 Python 3.


  如果你想了解核心 Python 开发者对于 Python 3 的出现有何看法，你可以阅读 Nick Coghlan 的 Python 3 Q & A 或
  Brett Cannon 的 为什么要有 Python 3.


  有关迁移的帮助，您可以通过电子邮件向 python-porting 邮件列表发送问题。
featured: /images/uploads/cloud-large.jpg
featuredpath: date
---
简要说明
为了使您的项目与Python 2/3单一源兼容，基本步骤是：

只担心支持Python 2.7

确保有良好的测试覆盖率（可以用 coverage.py 来完成此任务；用 pip install coverage 安装）

了解Python 2 和 3之间的区别

使用 Futurize (或 Modernize) 升级你的代码 (例如. pip install future)

使用Pylint帮助确保您不依赖Python 3支持（ pip install pylint ）

使用caniusepython3找出哪个依赖项阻止了对Python 3的使用（ pip install caniusepython3 ）

一旦您的依赖关系不再受阻，请使用持续集成以确保与Python 2和3保持兼容（ tox可以帮助测试多个版本的Python； pip install tox ）

考虑使用可选的静态类型检查，以确保您的类型用法在Python 2和3中都可以使用（例如，使用mypy在Python 2和Python 3中都检查您的键入）.

详情
关于支持Python的2和3的一个关键点是同时你今天可以开始了！ 即使你的依赖是不支持Python 3里然而，这并不意味着你不能现在现代化您的代码来支持Python 3的大部分更改，以支持Python 3里导致使用较新的做法，即使在Python 2代码更干净的代码需要.

另一个关键点是，现代化Python 2代码以同时支持Python 3对您来说是自动化的. 尽管您可能需要借助Python 3来澄清文本数据和二进制数据，从而做出一些API决策，但现在大部分工作已由您完成，因此至少可以立即受益于自动更改.

在继续阅读有关移植代码以同时支持Python 2和3的详细信息时，请记住这些关键点.

删除对Python 2.6及更早版本的支持
虽然可以使Python 2.5与Python 3一起使用，但如果只需要与Python 2.7一起使用， 则要容易得多 . 如果不能选择放弃Python 2.5，那么这六个项目可以帮助您同时支持Python 2.5和3（ pip install six ）. 但是请务必意识到，该HOWTO中列出的几乎所有项目都将对您不可用.

如果您可以跳过Python 2.5和更早的版本，则对代码进行的所需更改应继续看起来和惯用的Python代码一样. 在最坏的情况下，在某些情况下，您将不得不使用函数而不是方法，或者必须导入函数而不是使用内置函数，但是否则，整个转换对您而言不会感到陌生.

但是您应该只支持Python 2.7. 不再免费支持Python 2.6，因此不再收到错误修复. 这意味着您将必须解决Python 2.6遇到的所有问题. 在本HOWTO中还提到了一些不支持Python 2.6的工具（例如Pylint ），随着时间的流逝，它将变得越来越普遍. 如果您仅支持必须支持的Python版本，那么对您来说将更加容易.

Make sure you specify the proper version support in your setup.py file
在setup.py文件中，您应该具有适当的trove分类器，用于指定您支持的Python版本. 由于您的项目不支持Python 3，但您至少应具有Programming Language :: Python :: 2 :: Only指定. 理想情况下，您还应该指定您支持的每个主要/次要版本的Python，例如， Programming Language :: Python :: 2.7 .

良好的测试覆盖率
一旦您的代码支持了您想要的最旧版本的Python 2，就将要确保您的测试套件具有良好的覆盖率. 一个好的经验法则是，如果您对测试套件有足够的信心，那么在使用工具重写代码后出现的任何故障都是工具中的实际错误，而不是代码中的实际错误. 如果您想要一个数字作为目标，请尝试获得80％以上的覆盖率（如果发现很难获得超过90％的覆盖率，也不要感到难过）. 如果您还没有衡量测试覆盖率的工具，那么建议使用coverage.py .

了解Python 2 和 3之间的区别
一旦对代码进行了充分的测试，就可以开始将代码移植到Python 3了！ 但是要完全理解您的代码将如何更改以及在编写代码时要注意什么，您将需要了解Python 3就Python 2所做的更改.通常，两种最佳的阅读方法是阅读每个Python 3版本和" 移植到Python 3"书（在线免费）中的"新增功能"文档. Python-Future项目还有一个方便的备忘单 .

更新代码
一旦您知道Python 3与Python 2有什么不同，就该更新代码了！ 您可以选择两种工具来自动移植代码： Futurize和Modernize . 选择哪种工具将取决于您希望代码成为Python 3的程度. Futurize尽最大努力使Python 3中的习惯用法和实践存在于Python 2中，例如，从Python 3中反向移植bytes类型，以便在主要版本的Python之间具有语义奇偶校验. 另一方面， Modernize比较保守，其目标是直接依赖于六个 Python 2/3的Python子集来提供兼容性. 随着Python 3的未来发展，最好考虑使用Futurize来开始适应Python 3引入的您还不习惯的任何新实践.

无论您选择哪种工具，他们都会更新您的代码以在Python 3下运行，同时与您最初使用的Python 2版本保持兼容. 根据您想要保持的保守程度，您可能希望首先在测试套件上运行该工具，然后目视检查差异以确保转换正确. 转换完测试套件并确认所有测试仍按预期通过后，您可以转换应用程序代码，知道任何失败的测试都是翻译失败.

不幸的是，这些工具不能自动完成所有工作以使您的代码在Python 3下工作，因此您需要手动进行一些更新才能获得完整的Python 3支持（这些步骤中的哪些步骤因工具而异）. 阅读您选择使用的工具的文档，以查看其默认情况下已修复的功能以及可以选择执行的操作，以了解将为您（不）解决的问题以及您可能需要自己解决的问题（例如，使用io.open()通过内置open()函数是默认关闭的现代化）. 幸运的是，只有几件事需要注意，可以将其视为大问题，如果不加以注意可能很难调试.

除法
在Python 3中， 5 / 2 == 2.5而不是2 ； int值之间的所有除法都会导致float . 从2002年发布Python 2.2开始，实际上已经在计划这一更改.从那时起，鼓励用户from __future__ import division添加到使用/和//运算符或使用-Q运行解释器的所有文件中旗. 如果尚未执行此操作，则需要遍历代码并执行以下两项操作：

添加from __future__ import division给您的文件

根据需要更新任何除法运算符，以使用//使用地板除法或继续使用/并期望有浮点数

/不能简单地自动转换为//的原因是，如果一个对象定义了__truediv__方法而不是__floordiv__则您的代码将开始失败（例如，用户定义的类使用/表示某些操作，但不//出于同一件事或根本没有）.

文本与二进制数据
In Python 2 you could use the str type for both text and binary data. Unfortunately this confluence of two different concepts could lead to brittle code which sometimes worked for either kind of data, sometimes not. It also could lead to confusing APIs if people didn't explicitly state that something that accepted str accepted either text or binary data instead of one specific type. This complicated the situation especially for anyone supporting multiple languages as APIs wouldn't bother explicitly supporting unicode when they claimed text data support.

为了使文本和二进制数据之间的区别更清晰，更明显，Python 3做了互联网时代大多数语言创建的语言，并使文本和二进制数据的不同类型不能盲目地混在一起（Python早于对互联网）. 对于仅处理文本或仅处理二进制数据的任何代码，这种分隔都不会造成问题. 但是对于必须同时处理这两者的代码，这确实意味着您可能现在必须关心与二进制数据相比何时使用文本，这就是为什么不能完全自动化的原因.

To start, you will need to decide which APIs take text and which take binary (it is highly recommended you don't design APIs that can take both due to the difficulty of keeping the code working; as stated earlier it is difficult to do well). In Python 2 this means making sure the APIs that take text can work with unicode and those that work with binary data work with the bytes type from Python 3 (which is a subset of str in Python 2 and acts as an alias for bytes type in Python 2). Usually the biggest issue is realizing which methods exist on which types in Python 2 & 3 simultaneously (for text that's unicode in Python 2 and str in Python 3, for binary that's str/bytes in Python 2 and bytes in Python 3). The following table lists the unique methods of each data type across Python 2 & 3 (e.g., the decode() method is usable on the equivalent binary data type in either Python 2 or 3, but it can't be used by the textual data type consistently between Python 2 and 3 because str in Python 3 doesn't have the method). Do note that as of Python 3.5 the __mod__ method was added to the bytes type.

文本数据

二进制数据

decode

encode

format

isdecimal

isnumeric

通过在代码边缘的二进制数据和文本之间进行编码和解码，可以使区分更易于处理. 这意味着当您接收二进制数据中的文本时，应立即对其进行解码. 而且，如果您的代码需要将文本作为二进制数据发送，则应尽可能晚地对其进行编码. 这样一来，您的代码就只能在内部使用文本，从而无需跟踪正在处理的数据类型.

下一个问题是确保您知道代码中的字符串文字是表示文本还是二进制数据. 您应该在任何表示二进制数据的文字上添加b前缀. 对于文本，应在文本文字上添加u前缀. （有一个__future__导入强制所有未指定的文字为Unicode，但是用法表明，它不如向所有文字中明确添加b或u前缀一样有效）

作为这种二分法的一部分，您还需要注意打开文件的注意事项. 除非您在Windows上工作过，否则打开二进制文件（例如，用rb读取二进制文件）时，您不一定总是会费心添加b模式. 在Python 3中，二进制文件和文本文件明显不同且互不兼容. 有关详细信息，请参见io模块. 因此，您必须决定是将文件用于二进制访问（允许读取和/或写入二进制数据）还是文本访问（允许读取和/或写入文本数据）. 您还应该使用io.open()来打开文件，而不要使用内置的open()函数，因为io模块在Python 2到3中是一致的，而内置的open()函数不是（在Python 3中实际上是io.open() ）. 不要打扰使用codecs.open()的过时做法，因为只有这样才能保持与Python 2.5的兼容性.

对于Python 2和3之间的相同参数， str和bytes的构造函数具有不同的语义.在Python 2中将整数传递给bytes将为您提供整数的字符串表示形式： bytes(3) == '3' . 但是在Python 3中， bytes的整数参数将为您提供一个字节对象，只要指定的整数（用空字节填充bytes(3) == b'\x00\x00\x00' ： bytes(3) == b'\x00\x00\x00' . 将字节对象传递给str时，也有类似的担心. 在Python 2中，您只需返回bytes对象： str(b'3') == b'3' . 但是在Python 3中，您将获得bytes对象的字符串表示形式： str(b'3') == "b'3'" .

最后，二进制数据的索引，需要小心处理（切片不需要任何特殊处理）. 在Python 2中， b'123'[1] == b'2'而在Python 3中， b'123'[1] == 50 . 因为二进制数据只是二进制数的集合，所以Python 3返回您索引的字节的整数值. 但在Python 2中，因为bytes == str ，索引将返回一个单项字节切片. 这六个项目有一个名为six.indexbytes()的函数，该函数将返回一个整数，类似于Python 3： six.indexbytes(b'123', 1) .

总结一下：

确定您的哪个API使用文本，哪些使用二进制数据

确保与文本一起使用的代码也可以与unicode并且二进制数据的代码在Python 2中可以与bytes一起使用（有关每种类型不能使用的方法，请参见上表）

用b前缀标记所有二进制文字，用u前缀标记文字文字

尽快将二进制数据解码为文本，尽快将文本编码为二进制数据

使用io.open()打开文件，并确保在适当时指定b模式

索引到二进制数据时要小心

Use feature detection instead of version detection
不可避免地，您将拥有必须根据正在运行的Python版本选择要执行的操作的代码. 最好的方法是通过功能检测您正在运行的Python版本是否支持所需的功能. 如果由于某种原因不起作用，则应针对Python 2而不是Python 3进行版本检查.为帮助解释这一点，让我们看一个示例.

假设您需要访问importlib的功能，该功能自Python 3.3起在Python的标准库中可用，并且可通过PyPI上的importlib2在Python 2中使用. 您可能想通过执行以下操作来编写代码以访问例如importlib.abc模块：

import sys

if sys.version_info[0] == 3:
    from importlib import abc
else:
    from importlib2 import abc
这段代码的问题是当Python 4发布时会发生什么？ 最好将Python 2视为特殊情况，而不是Python 3，并假定将来的Python版本将比Python 2与Python 3更兼容：

import sys

if sys.version_info[0] > 2:
    from importlib import abc
else:
    from importlib2 import abc
但是，最好的解决方案是完全不进行版本检测，而要依靠功能检测. 这样可以避免因版本检测错误而引起的任何潜在问题，并有助于保持将来的兼容性：

try:
    from importlib import abc
except ImportError:
    from importlib2 import abc
Prevent compatibility regressions
完全翻译完代码以使其与Python 3兼容后，您将要确保代码不会退化并停止在Python 3下运行.如果您的依赖项阻止您实际在以下环境下运行，则尤其如此.目前使用Python 3.

为了保持兼容性，您创建的任何新模块的顶部至少应包含以下代码块：

from __future__ import absolute_import
from __future__ import division
from __future__ import print_function
您还可以运行带有-3标志的Python 2，以警告代码在执行过程中触发的各种兼容性问题. 如果使用-Werror将警告转换为错误，则可以确保您不会意外遗漏警告.

您也可以使用pylint的项目及其--py3k标志掉毛你的代码收到警告，当你的代码开始在Python 3兼容性偏离. 这也避免了您必须定期对代码运行Modernize或Futurize来捕获兼容性回归. 这确实要求您仅支持Python 2.7和Python 3.4或更高版本，因为这是Pylint的最低Python版本支持.

Check which dependencies block your transition
你做你的代码与Python 3兼容后，你应该开始关心是否你的依赖也被移植. 该caniusepython3项目的设立是为了帮助你确定哪些项目-直接或间接地-从支持Python 3的阻挡你有既是一个命令行工具，以及在Web界面https://caniusepython3.com .

该项目还提供了可以集成到测试套件中的代码，这样当您不再具有依赖项而无法使用Python 3时，您将无法通过测试.这使您避免手动检查依赖项并迅速得到通知.当您可以开始在Python 3上运行时.

Update your setup.py file to denote Python 3 compatibility
一旦您的代码在Python 3下工作，您就应该更新setup.py的分类器，使其包含Programming Language :: Python :: 3并且不要指定唯一的Python 2支持. 这将告诉使用您的代码的任何人都支持Python 2 和 3.理想情况下，您还希望为现在支持的每个主要/次要版本的Python添加分类器.

Use continuous integration to stay compatible
Once you are able to fully run under Python 3 you will want to make sure your code always works under both Python 2 & 3. Probably the best tool for running your tests under multiple Python interpreters is tox. You can then integrate tox with your continuous integration system so that you never accidentally break Python 2 or 3 support.

当您比较字节和字符串或字节和int时，您可能还想在Python 3解释器中使用-bb标志来触发异常（后者从Python 3.5开始可用）. 默认情况下，类型不同的比较仅返回False ，但是如果您在文本/二进制数据处理或对字节的索引分离中犯了一个错误，则不会轻易发现错误. 发生此类比较时，此标志将引发异常，从而使错误更易于查找.

大部分就是这样！ 此时，您的代码库同时与Python 2和3兼容. 还将对您的测试进行设置，以确保您在开发过程中通常不会在哪个版本下运行测试，都不会意外破坏Python 2或3的兼容性.

考虑使用可选的静态类型检查
帮助移植代码的另一种方法是在代码上使用诸如mypy或pytype之类的静态类型检查器. 这些工具可用于分析您的代码，就像它在Python 2下运行一样.然后，您可以再次运行该工具，就像您的代码在Python 3下运行一样.通过像这样运行两次静态类型检查器，您可以发现是否例如，与另一个版本相比，您在一个版本的Python中误用了二进制数据类型. 如果您在代码中添加了可选的类型提示，则还可以显式声明您的API使用的是文本数据还是二进制数据，从而有助于确保所有功能在这两个Python版本中都能正常运行.
