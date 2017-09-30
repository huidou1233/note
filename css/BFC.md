block formatting context(块格式化上下文)是Web页面中盒模型布局的CSS渲染模式。

- 一个块格式化上下文由以下之一创建：
- 浮动元素(元素的float不是none)
- 绝对定位元素(元素具有position为absolute或fixed)
- display值为table-cell，`table-caption`, `inline-block`, `flex`, 或者 `inline-flex`中的其中一个
- 具有overflow且值不是visible的块元素
- 在一个块格式化上下中，盒在 竖直方向一个接一个地放置，从包含块的顶部开始。两个兄弟盒之间的竖直距离由'margin'属性决定。同一个块格式化上下中的相邻块级盒之间的竖直margin会合并

在一个块格式化上下中，每个盒的left外边，挨着包含块的left边(对于从右向左的格式化，right边挨着)。即使存在浮动(尽管一个盒的*行盒*可能会因为浮动收缩),这也成立。除非该盒建立了一个新的块级格式化上下文(这种情况下，该盒自身可能会因为浮动变窄)。



