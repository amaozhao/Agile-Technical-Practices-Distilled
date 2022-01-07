# 测试遗留代码
> 没有测试的代码是糟糕的代码。不管它写得有多好；不管它有多漂亮、多面向对象或多好的封装。通过测试，我们可以快速且可验证地更改代码的行为。没有它们，我们真的不知道我们的代码是变得更好还是更糟。对我来说，遗留代码只是没有测试的代码。
>
> – Michael C. Feathers，有效处理遗留代码

曾几何时，一位精通本书主题的朋友去一家大公司的咨询公司面试。他们给了他一个关于重构一些遗留代码的练习。他创造了一个接缝，并用它以一种他们以前从未见过的方式解决了这个练习。

起初，他们想拒绝申请。直到有人指出这是一种合法的重构技术时，他们才改变主意并开始咨询工作。在那里工作了六个月后，他们非常欣赏他的工作，以至于他们试图永久雇用他（顺便说一句，没有成功）。

## 使用接缝打破依赖关系

有时由于过度耦合而无法编写测试。如果你的代码处理外部依赖，我们可以打破依赖，让被测对象独立运行。

如果依赖是通过构造函数或 setter 注入的，用测试替身替换依赖应该就足够了。不幸的是，情况并非总是如此。有时你必须处理静态依赖项或本地生成的依赖项。在这些情况下，你将不得不分离依赖项，在你的代码中引入一个接缝（如 Michael C. Feathers 在 Working Effectively with Legacy Code 中所指定的）。

在服装中，接缝将各部分连接在一起形成一件衣服。在代码中，我们可以使用这个概念来找到可以分离耦合部分的软点。如果你的语言支持将函数作为参数传递，你可以使用 Llewellyn Falco 的 Peel-and-Slice 技术。

在打破依赖关系时我们应该非常小心，尤其是当我们的测试覆盖率很低时。这是一个非常微妙的操作，因此我们希望尽量减少测试未涵盖的更改。理想情况下，我们希望对测试代码而不是生产代码进行大部分更改。

### 使用继承解耦生产代码

```
public class Game
{
public void Play()
{
var diceResult = new Random().Next(1, 6);
...
}
}
```

在这种情况下，Game 类与随机数生成器库相结合。我们需要控制骰子滚动以测试 Game 类。

在 Game 类中添加一个受保护的虚方法来封装存在耦合问题的行为。我们使用一个受保护的方法来避免扩展 Game 类的公共接口：

```
public class Game
{
public void Play()
{
var diceResult = roll();
...
}
protected virtual int Roll()
{
return new Random().Next(1, 6);
}
}
```

在你的测试代码中，从 Game 类继承并将受保护的虚拟方法 Roll 的行为更改为你可以控制的内容：

```
public class TestableGame : Game
{
private int roll;
public TestableGame(int desiredRoll)
{
roll = desiredRoll;
}
protected override int Roll()
{
return roll;
}
}
```

使用 TestableGame 类编写测试：

```
public class GameShould
{
[Test]
public void Do_Something_When_A_Six_Is_Rolled()
{
var game = new TestableGame(6);
game.Play();
//ASSERT
}
}
```

当生产代码调用静态方法时，也可以应用此技术。使用这种方法的优点是它最大限度地减少了对生产代码的更改，并且可以使用自动重构来完成，从而最大限度地减少对生产代码引入破坏性更改的风险。例如，一旦你拥有良好的测试覆盖率，你就可以重构 Game 类并通过在 Game 类中注入依赖项来消除对接缝的需要。

## 特性测试

Michael C. Feathers 创造了术语特性化测试，他将其定义为描述（特性化）一段代码实际行为的测试。这些测试通过自动化测试保护遗留代码的现有行为免受意外更改。

换句话说，他们不像规范测试那样检查代码应该做什么，而是检查代码实际和当前做了什么。拥有一组特性测试有助于开发人员处理遗留代码，因为他们可以在修改代码后运行这些测试，确保他们的修改不会导致其他地方的功能发生任何意外或不需要的更改。

1. 在测试工具中使用一段代码。
2. 写一个你知道会失败的断言。
3. 运行测试，让失败告诉你实际的行为是什么。
4. 更改测试以使其预期代码实际产生的行为。
5. 重复，直到你合理地确定所有自由度都已识别和测试。
6. 根据你所描述的商业行为命名测试。

## Kata

### Emily Bache 对镀金玫瑰卡塔的特性测试

向 GildedRose 类添加特性测试。尝试从业务角度命名测试，以描述代码正在做什么的方式。与我们的第一种测试方法一样，记住自由度；在进入下一个练习之前，保持一种行为，直到你完全表征它。

你可以在 Emily 的公共存储库中找到你选择的语言的存储库：https://github.com/emilybache/GildedRose-Refactoring-Kata。

## 金大师
当在系统级别很容易获得清晰的输入和输出时，Golden Master 技术非常有用。金主功法，在某些情况下，是很难施展的，也有根本施展不了的。对于所有给定的情况，我们需要考虑使用金大师生成的系统测试是否足够，或者是否需要添加其他类型的测试，例如单元测试、组件测试、集成测试等。

在以下小节中，我们提供了为系统创建黄金大师的指南。

### 可行性

1. 系统是否有明确的输入和输出？示例可能包括控制台、文件系统、网络等。
2. 系统是否为相同的输入生成相同的输出？如果没有，我们可以使用测试替身来实现吗？隔离副作用。
3. 我们能否在不改变系统行为的情况下捕捉系统对我们测试的影响？选项是将输出重定向到文件或用内存数据层替换数据层。
4. 我们可以在不改变系统行为的情况下在测试中注入系统的输入吗？与第 3 点相同的选项。

### 生成输入/输出（黄金大师）

1. 逐渐为系统创建一个假输入并将其保存在文件中。提示：注意输入和输出中的模式。
2. 创建一个测试，加载假输入，将其注入系统，并捕获并保留输出。
3. 测量测试覆盖率。重复直到覆盖率接近或达到 100%。

### 断言

1. 扩展前面的测试以断言给定输入（Golden Master 输入），我们得到上一步保存的预期输出（Golden Master）。
2. 提交。不要忘记包含 Golden Master 输入和输出文件。
3. 对生产代码进行小改动进行试验，以检查测试是否失败。完成后还原。如果你对结果不满意，请返回上一节并生成更好的输入/输出组合，或者使用新的输入/输出组合编写新测试。

如果我们按照前面部分中的步骤进行操作，我们现在应该对我们的系统进行 Golden Master 测试。如果我们对生产代码进行任何破坏系统行为的更改，我们的 Golden Master 测试应该会失败。

## Kata

艾米丽·巴奇 (Emily Bache) 的镀金玫瑰卡塔 (Golded Rose Kata) 上的金大师

为镀金玫瑰卡塔创造一个黄金大师。你可以在此处找到存储库：https://github.com/emilybache/GildedRose-Refactoring-Kata。

> 提示：
>
> 从文件中读取输入。

将输出重定向到文件。在这种情况下，系统的输出是控制台。为了捕获系统输出，我们可以将控制台重定向到内存中的流，然后将流保存到文件中。

C#

```c#
var streamwriter = new StreamWriter(new FileStream(“/location/out.txt”, FileMode.Create));
streamwriter.AutoFlush = true;
Console.SetOut(streamwriter);
```

Java

```java
System.setOut(new PrintStream(new BufferedOutputStream(new FileOutputStream(“/location/out.txt”)), true));
```

Python

```python
import sys
sys.stdout = open('/location/out.txt', 'w')
```

Ruby

```ruby
$stdout = File.new('/location/out.txt', 'w')
$stdout.sync = true
```

## Llewellyn Falco 的批准测试

Approval Tests 是由 Llewellyn Falco 基于黄金大师的想法构思的测试框架。它有一个流畅的 API，允许定义场景和验证输出。通常，它将序列化人类将标记为已接受的执行的输出。如果代码中的任何内容更改了输出，则测试将失败。

Llewellyn Falco：http://approvaltests.com/。

批准测试的一个鲜为人知的用法是锁定用于重构的代码。通过代码锁定，我们指的是围绕一段代码添加测试以使重构成为可能的活动。将代码覆盖率工具与此技术结合使用可提供有关我们的新测试实际执行了多少代码的反馈。当我们达到令人满意的覆盖水平时，我们就可以进入重构阶段。一旦我们的代码处于更好的状态并且组件被提取和隔离，它们应该更容易理解。然后我们可以为它们中的每一个添加测试，以便我们可以删除锁定测试。

### 组合测试

组合测试是批准测试的一个功能，它允许以组合方式将参数传递给被测系统。给定一组已知的输入，它们将以所有可能的方式组合起来，以尽可能多地执行执行路径。

小心组合爆炸！能力越大责任越大，因此在向输入集添加更多值时要小心。由此产生的参数集合可能会增加得如此之快，以至于你的测试执行将无法继续，并且捕获的黄金大师将占用你磁盘上的大量空间。

## 重温重构指南
### 重构时保持绿色

在第 6 课重构中，我们说过，“不要更改测试未涵盖的生产代码。如果我们需要重构一些没有测试的代码，那么从添加行为测试开始。”现在我们知道如何测试遗留代码，我们可以扩展我们的指导方针以包括：

- 基于批准的测试
- 特性测试
- 黄金大师测试
- 基于模型的测试
- 基于属性的测试

## Kata

### 镀金玫瑰重构 Kata by Emily Bache

如果你已经执行了前面的练习，现在应该有一套完整的测试来证明代码在任何自由度下的行为。

相信你的测试并保持绿色，你终于可以继续重构 Emily Bache 的 Gilded Rose kata。

## 结论
精通为遗留代码编写测试需要时间，我们明白有时这是一项艰巨的任务。在本课中，我们提炼了我们认为遗留代码测试中最重要的概念。花一些时间练习我们概述的技术是一项非常有用的练习，我们希望它可以给你一些想法，帮助你使这项工作不那么令人生畏。

### 指导问题

详细了解遗留代码重要吗？

如果答案是肯定的，特性测试可能是最好的解决方案。这是一个缓慢的过程，但为我们提供了有关遗留代码如何工作的最佳见解。

我们可以通过对遗留代码的肤浅理解来解决问题吗？

如果答案是肯定的，并且被测系统有明确的输入和输出，我们可能可以使用黄金大师技术或批准测试。

我们添加了一些网络资源和书籍，可以更深入地探讨这个主题。

## 好习惯

在本课中，我们引入了一个新习惯。在以下列表中查看。

### 编写新测试时的注意事项

- 测试应该只测试一件事。
- 创建更具体的测试以推动更通用的解决方案（三角测量）。
- 为你的测试提供反映你的业务领域的有意义的名称（面向行为/面向目标的名称）。
- 查看测试是否因正确的原因而失败。
- 确保你从失败的测试中获得有意义的反馈。
- 将你的测试和生产代码分开。
- 组织你的单元测试以反映你的生产代码（类似的项目结构）。
- 在安排、行动和断言块中组织你的测试。
    - 安排（也称为给定）所有必要的先决条件。
    - 针对被测对象采取行动（也称为何时）。
    - 断言（也称为 Then）预期结果已经发生。
- 先写断言，然后逆向工作。
    - 首先编写断言来编写测试；甚至不用担心正确命名测试。
    - 编写测试的行为部分。
    - 如果需要，编写排列块。
    - 最后，命名测试。
- 编写快速、隔离、可重复和自我验证的测试。
- 考虑使用对象健美操来推动设计决策。
- 考虑向遗留代码添加测试（新习惯）。
    - 批准测试、特性测试、黄金大师测试。

### 使失败的测试通过时的注意事项

- 编写最简单的代码来通过测试。
- 编写任何可以让你更快地进入重构阶段的代码。
- 使用转换优先的前提。
- 考虑使用对象健美操来推动设计决策。

### 测试通过后的注意事项

- 使用三法则来解决重复问题。
- 不断重构设计。
- 应用对象健美操来改进你的设计。
- 重构时保持绿色。
- 使用 IDE 快速安全地重构。
- 首先重构代码以提高可读性/可理解性。
- 注意代码异味并相应地重构代码。

### 我应该什么时候进入下一课？

- 当你能够打破依赖关系以允许测试时。
- 当你能够向以前未测试的代码添加测试时。
- 当你能够重构生产遗留代码时。
- 当你不能再容忍坏名字时。

## 资源
### 网络

- Llewellyn Falco 的批准测试：http://approvaltests.com/。
- Emily Bache 的公共 GitHub 存储库：https://github.com/emilybache。
- 剥皮和切片技术，Llewellyn Falco：https://www.youtube.com/playlist?list=PLb4ON7iRsxZNNqZuA2dlQOW3MOQwQ6AjM。
- Llewellyn Falco 在 .NET 中使用 ApprovalTests：https://www.youtube.com/playlist?list=PL0C32F89E8BBB5368。

### 图书

- Michael C. Feathers 的《有效处理遗留代码》：https://www.goodreads.com/book/show/44919.Working_Effectively_with_Legacy_Code。