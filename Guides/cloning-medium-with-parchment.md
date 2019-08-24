# Cloning Medium with Parchment
> 注意：本节包含大量代码，原文都是 codepen 的代码预览视图，我为了阅读方便也将代码贴过来了，但是 HTML 和 css 代码我只会贴一遍，后面遇到 codepen 的地方我只贴 js 的代码。

要提供一致的编辑体验，您需要一致的数据和可预测的行为。 不幸的是，DOM缺乏这两者。现代编辑器的解决方案是维护自己的文档模型来表示其内容。[Parchment](https://github.com/quilljs/parchment)就是Quill的解决方案。它在自己的代码库中组织，具有自己的API层。通过Parchment，您可以自定义Quill识别的内容和格式，或添加全新的内容和格式。

在本指南中，我们将使用Parchment和Quill提供的构建块在Medium上复制​​编辑器。 我们将从Quill的骨干开始，没有任何主题，无关的模块或格式。 在这个基本级别，Quill只能理解纯文本。 但是，在本指南结束时，我们将了解链接，视频甚至推文。

## Groundwork（准备工作）
让我们在没有使用Quill的情况下开始，只需要一个textarea和按钮，连接到一个虚拟事件监听器。 我们将在本指南中使用jQuery以方便使用，但Quill和Parchment都不依赖于此。在[Google Fonts](https://fonts.google.com/)和[Font Awesome](http://www.fontawesome.com.cn/faicons/)的帮助下，我们还将添加一些基本样式。 这些与Quill或Parchment没有任何关系，所以我们会快速进行。

```html
<div id="tooltip-controls">
  <button id="bold-button"><i class="fa fa-bold"></i></button>
  <button id="italic-button"><i class="fa fa-italic"></i></button>
  <button id="link-button"><i class="fa fa-link"></i></button>
  <button id="blockquote-button"><i class="fa fa-quote-right"></i></button>
  <button id="header-1-button"><i class="fa fa-header"><sub>1</sub></i></button>
  <button id="header-2-button"><i class="fa fa-header"><sub>2</sub></i></button>
</div>
<div id="sidebar-controls">
  <button id="image-button"><i class="fa fa-camera"></i></button>
  <button id="video-button"><i class="fa fa-play"></i></button>
  <button id="tweet-button"><i class="fa fa-twitter"></i></button>
  <button id="divider-button"><i class="fa fa-minus"></i></button>
</div>
<!-- <textarea id="editor-container">Tell your story...</textarea> -->
<div id="editor-container">Tell your story...</div>
```

```css
* {
  box-sizing: border-box;
}

html, body {
  height: 100%;
  margin: 0;
  width: 100%;
}
body {
  padding: 25px;
}

#editor-container {
  display: block;
  font-family: 'Open Sans', Helvetica, sans-serif;
  font-size: 1.2em;
  height: 200px;
  margin: 0 auto;
  width: 450px;
}

#tooltip-controls, #sidebar-controls {
  text-align: center;
}

button {
  background: transparent;
  border: none;
  cursor: pointer;
  display: inline-block;
  font-size: 18px;
  padding: 0;
  height: 32px;
  width: 32px;
  text-align: center;
}
button:active, button:focus {
  outline: none;
}
```

```js
$('button').click(function() {
  alert('Click!');
});
```
注：原文这里是一个 codepen 的视图，我们可以直接通过[这个链接](https://codepen.io/quill/pen/oLVAKZ)过去。

## Adding Quill Core
接下来，我们将用Quill核心替换textarea，缺少主题，格式和无关模块。 在键入编辑器时打开开发人员控制台以检查演示。 您可以在工作中看到Parchment文档的基本构建块。

```js
$('button').click(function() {
  alert('Click!');
});

let quill = new Quill('#editor-container');
```

注：原文这里是一个 codepen 的视图，我们可以直接通过[这个链接](https://codepen.io/quill/pen/QEoZQb)过去。

与DOM一样，Parchment文档也是一棵树。 它的节点称为Blots，是DOM节点的抽象。我们已经为我们定义了一些blots：Scroll, Block, Inline, Text 和 Break。键入时，Text blot与相应的DOM Text节点同步; 输入通过创建新的blot块来处理。在Parchment中，可以拥有孩子的Blots必须至少有一个孩子，所以空的块填充了一个Break blot。这使得处理叶子简单且可预测。 所有这些都是在根Scroll blot下组织的。

您只能在此处键入内容来观察Inline blot，因为它不会为文档提供有意义的结构或格式。合理的Quill文档必须是规范和紧凑的。这里只有一个可以表示给定文档的有效DOM树，并且该DOM树包含最少数量的节点。

由于`<p><span>文本</span> </p>`和`<p>文本</p>`表示相同的内容，因此前者无效，并且它是Quill打开`<span>`的优化过程的一部分。同样，一旦我们添加格式，`<p><em>Te</em><em>st</em></p>`和`<p><em><em>Test</em></em> </p>`也无效，因为它们不是最紧凑的表示。

由于这些约束，**Quill不能支持任意DOM树和HTML的更改**。但正如我们将看到的，这种结构提供的一致性和可预测性使我们能够轻松地构建丰富的编辑体验。

## Basic Formatting
我们之前提到Inline不提供格式化。 这是基础Inline类的例外，而不是规则。 基本块级blot对块级元素的工作方式相同。

要实现粗体和斜体，我们只需要从Inline继承，设置`blotName`和`tagName`，并将其注册到Quill。有关继承和静态方法和变量的签名的完整参考，请查看[Parchment](https://github.com/quilljs/parchment/)。

```js
let Inline = Quill.import('blots/inline');

class BoldBlot extends Inline { }
BoldBlot.blotName = 'bold';
BoldBlot.tagName = 'strong';

class ItalicBlot extends Inline { }
ItalicBlot.blotName = 'italic';
ItalicBlot.tagName = 'em';

Quill.register(BoldBlot);
Quill.register(ItalicBlot);
```

我们在这里使用`strong`和`em`标签遵循Medium的例子，但你也可以使用`b`和`i`标签。 blot的名称将被Quill用作格式的名称。通过注册我们的blots，我们现在可以在我们的新格式上使用Quill的完整API：

```js
Quill.register(BoldBlot);
Quill.register(ItalicBlot);

var quill = new Quill('#editor');

quill.insertText(0, 'Test', { bold: true });
quill.formatText(0, 4, 'italic', true);
// If we named our italic blot "myitalic", we would call
// quill.formatText(0, 4, 'myitalic', true);
```

让我们摆脱我们的虚拟按钮处理程序，并将粗体和斜体按钮连接到Quill的[format()](https://github.com/hzjswlgbsj/quill-document-chinese/blob/master/Documentation/API/formatting.md)。为了简单起见，我们将硬编码true以始终添加格式。在您的应用程序中，您可以使用[getFormat()](https://github.com/hzjswlgbsj/quill-document-chinese/blob/master/Documentation/API/formatting.md)来检索任意范围内的当前格式，以决定是添加还是删除格式。[Toolbar](https://github.com/hzjswlgbsj/quill-document-chinese/blob/master/Documentation/modules/toolbar.md)模块为Quill实现了这一点，我们不在这里重新实现它。

打开您的开发人员控制台，以新的粗体和斜体格式试用Quill的[API](https://github.com/hzjswlgbsj/quill-document-chinese/blob/master/Documentation/API/API.md)！ 确保将上下文设置为正确的CodePen iframe，以便能够访问演示中的quill变量。

```js
let Inline = Quill.import('blots/inline');

class BoldBlot extends Inline { }
BoldBlot.blotName = 'bold';
BoldBlot.tagName = 'strong';

class ItalicBlot extends Inline { }
ItalicBlot.blotName = 'italic';
ItalicBlot.tagName = 'em';

Quill.register(BoldBlot);
Quill.register(ItalicBlot);

var quill = new Quill('#editor-container');

$('#bold-button').click(function() {
  quill.format('bold', true);
});
$('#italic-button').click(function() {
  quill.format('italic', true);
});
```

注：原文这里是一个 codepen 的视图，我们可以直接通过[这个链接](https://codepen.io/quill/pen/VjRovy)过去。

请注意，如果您对某些文本同时应用粗体和斜体，则无论您执行何种顺序，Quill都会以一致的顺序将`<strong>`标记包装在`<em>`标记之外。

## Links
链接稍微复杂一些，因为我们需要不止一个布尔值来存储链接URL。这会以两种方式影响我们的链接blot：创建和格式检索。我们将url表示为字符串值，但我们可以通过其他方式轻松实现，例如带有url键的对象，允许设置其他键/值对并定义链接。我们稍后将用下文的`image`演示。

```js
class LinkBlot extends Inline {
  static create(value) {
    let node = super.create();
    // 如果需要，清除url值
    node.setAttribute('href', value);
    // 可以设置其他非格式相关的属性
    // 这些对于Parchment是不可见的，所以必须是静态的
    node.setAttribute('target', '_blank');
    return node;
  }

  static formats(node) {
    // 我们将只在已经有节点的情况下被调用
    // 确定是Link blot，所以我们这样做
    // 不需要检查自己
    return node.getAttribute('href');
  }
}
LinkBlot.blotName = 'link';
LinkBlot.tagName = 'a';

Quill.register(LinkBlot);
```

现在我们可以将链接按钮挂钩到一个花哨的`prompt`，再次保持简单，然后传递给Quill的`format()`。

```js
let Inline = Quill.import('blots/inline');

class BoldBlot extends Inline { }
BoldBlot.blotName = 'bold';
BoldBlot.tagName = 'strong';

class ItalicBlot extends Inline { }
ItalicBlot.blotName = 'italic';
ItalicBlot.tagName = 'em';

class LinkBlot extends Inline {
  static create(url) {
    let node = super.create();
    node.setAttribute('href', url);
    node.setAttribute('target', '_blank');
    return node;
  }
  
  static formats(node) {
    return node.getAttribute('href');
  }
}
LinkBlot.blotName = 'link';
LinkBlot.tagName = 'a';

Quill.register(BoldBlot);
Quill.register(ItalicBlot);
Quill.register(LinkBlot);

let quill = new Quill('#editor-container');

$('#bold-button').click(function() {
  quill.format('bold', true);
});
$('#italic-button').click(function() {
  quill.format('italic', true);
});

$('#link-button').click(function() {
  let value = prompt('Enter link URL');
  quill.format('link', value);
});
```

注：原文这里是一个 codepen 的视图，我们可以直接通过[这个链接](https://codepen.io/quill/pen/VjRovy)过去。
