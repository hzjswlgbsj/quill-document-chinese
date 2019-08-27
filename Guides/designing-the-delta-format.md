# Designing the Delta Format
富文本编辑器缺乏表达自己内容的规范。直到最近，大多数富文本编辑甚至都不知道他们自己的编辑区域是什么。这些编辑器只传递用户HTML，以及解析和解释它的负担。在任何给定时间，此解释将与主要浏览器供应商的解释不同，从而为用户带来不同的编辑体验。

Quill是第一个真正理解自己内容的富文本编辑器。关键是Deltas，这是描述富文本的规范。 Deltas旨在易于理解和使用。我们将介绍Deltas背后的一些想法，以阐明*为什么*事情就像他们一样。

如果您正在寻找关于Deltas的参考，[Delta文档](https://github.com/hzjswlgbsj/quill-document-chinese/blob/master/Documentation/delta.md)是一个更简洁的资源。

## Plain Text
让我们从纯文本的基础开始。 已存在无处不在的格式来存储纯文本：字符串。现在，如果我们想要在此基础上构建并描述格式化文本，例如当范围为粗体时，我们需要添加其他信息。

数组是唯一可用的其他有序数据类型，因此我们使用对象数组。这也使我们能够利用JSON与各种工具兼容。

```js
var content = [
  { text: 'Hello' },
  { text: 'World', bold: true }
];
```

如果我们愿意，我们可以将斜体，下划线和其他格式添加到主对象; 但是将他们与`text`分开是更清晰的，因此我们在一个字段下组织格式，我们将其命名为`attributes`。

```js
var content = [
  { text: 'Hello' },
  { text: 'World', attributes: { bold: true } }
];
```

## Compact（紧凑的）
由于Delta格式太过于简单，上面的“Hello World”示例可以有不同的方式表示（就像下面的代码表现的一样），因此无法预测将生成哪个：

```js
var content = [
  { text: 'Hel' },
  { text: 'lo' },
  { text: 'World', attributes: { bold: true } }
];
```

为了解决这个问题，我们添加了Deltas必须紧凑的约束。有了这个约束，上面的表示不是有效的Delta，因为它可以通过前面的例子更紧凑地表示，其中“Hel”和“lo”不是分开的。同样，我们不能有`{bold：false，italic：true，underline：null}`，因为`{italic：true}`更紧凑。

## Canonical（标准）
我们没有为`bold`指定任何含义，只是它描述了一些文本格式。我们很可能使用了不同的名称，例如`weighted`或`strong`，或使用了不同范围的可能值，例如数字或描述性权重范围。可以在CSS中找到一个例子，其中大多数歧义都在发挥作用。如果我们在页面上看到粗体文本，我们无法预测其规则集是否为`font-weight：bold`或`font-weight：700`。这使得解析CSS的任务能够辨别其含义，这要复杂得多。

我们没有定义可能属性的集合，也没有定义它们的含义，但我们确实添加了一个额外的约束，Deltas必须是规范的。如果两个Deltas相等，则它们表示的内容必须相等，并且不能有两个表示相同内容的不等Deltas。以编程方式，这允许您简单地深入比较两个Deltas以确定它们所代表的内容是否相等。

因此，如果我们有以下内容，我们可以得出的唯一结论是`a`与`b`不同，但不是不知道`a`或`b`的意思。

```js
var content = [{
  text: "Mystery",
  attributes: {
    a: true,
    b: true
  }
}];
```

实施者可以选择合适的名称：

```js
var content = [{
  text: "Mystery",
  attributes: {
    italic: true,
    bold: true
  }
}];
```

此规范化适用于键和值，文本和属性。 例如，Quill默认情况下：

- 使用六个字符的十六进制值来表示颜色而不是RGB

- 只有一种方法可以表示换行符`\n`，而不是`\r`或`\r\n`

- `text：“Hello World”`明确表示“Hello”和“World”之间恰好有两个空格

其中一些选择可以由用户自定义，但Deltas中的规范约束规定选择必须是唯一的。

这种明确的可预测性使得Deltas更容易使用，因为您需要处理的案例更少，并且因为相应的Delta看起来没有意外。 从长远来看，这使得使用Deltas的应用程序更易于理解和维护。

## Line Formatting
