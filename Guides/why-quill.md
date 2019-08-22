# 为什么是Quill
 内容创建从一开始就是网络的核心。 `<textarea>`为几乎任何Web应用程序提供本机和必要的解决方案。 但在某些时候，您可能需要为文本输入添加格式。 这是富文本编辑器的用武之地。有许多解决方案可供选择，但 Quill 带来了一些现代的想法。

## API驱动设计
富文本编辑器旨在帮助人们编写文本。 然而令人惊讶的是，大多数富文本编辑都不知道用户撰写的文本。 这些编辑通过Web开发人员所做的相同镜头来看待他们的内容：DOM。 这表示阻抗不匹配，因为DOM由在不平衡树中组织的节点组成，而文本由线，单词和字符组成。

没有DOM API，其中字符是度量单位。 有了这个限制，大多数富文本编辑器都无法回答简单的问题，例如“此范围内的文本是什么？”或“光标是否加粗文本？”试图在这些基元之上构建丰富的编辑体验非常困难和令人沮丧。

Quill专为编辑和角色而设计，并在这些以自然文本为中心的单元之上构建其API。 要查明某些内容是否为粗体，Quill不需要遍历DOM查找`<b>`或`<strong>`节点或字体权重样式属性 - 只需调用`getFormat（5,1`。 它的所有核心[API](https://quilljs.com/docs/api/)调用都允许任意索引和长度进行访问或修改。 其[event API](https://quilljs.com/docs/api/#events)还以直观的JSON格式报告更改。 无需解析HTML或差异DOM树。

## 自定义内容和格式
过去评估富文本编辑器就像比较所需格式的清单一样简单。 一个好的富文本编辑器的标记就是它支持的格式。 这仍然是一个重要的衡量标准，但下限接近无穷大。

不再写入要打印的文本。 它被写成在网络上呈现 - 比纸张更丰富的画布。 内容可以是实时的，交互式的，甚至是协作的。 只有一些富文本编辑器甚至可以支持图像和视频等简单媒体; 几乎没有人可以嵌入推文或交互式图表。 然而，这是网络发展的方向：更丰富，更具互动性。 支持内容创建的工具需要考虑这些用例。

Quill公开了自己的文档模型，这是对DOM的强大抽象，允许扩展和自定义。 Quill可以支持的格式和内容的上限是无限的。 用户已经使用它来添加嵌入式幻灯片，交互式清单和3D模型。

## 跨平台
跨平台支持对许多Javascript库很重要，但这意味着什么的标准经常不同。 对于Quill来说，酒吧不仅仅是它运行或运行，它必须以相同的方式运行或工作。 功能不仅是跨平台的考虑因素，而且用户和开发人员的体验也是如此。 如果某些内容在OSX上的Chrome中生成特定标记，则会在IE上生成相同的标记。 如果点击进入在Windows上的Firefox中保留粗体格式状态，它将保留在移动Safari上。

## 使用方便
所有这些优点都来自易于使用的包装。 Quill附带了理智的默认设置，只需几行Javascript即可立即使用：
```javascript
var quill = new Quill('#editor', {
  modules: { toolbar: true },
  theme: 'snow'
});
```

如果您的应用程序从不需要它，您就不必自定义Quill - 只需享受开箱即用的丰富而一致的体验。