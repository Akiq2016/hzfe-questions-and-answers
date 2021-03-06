```bash
# 此页面贡献者：小k
```

# 浏览器渲染过程

浏览器渲染过程，几乎前端面试必考题。一般分为如下几个步骤：
- 解析HTML(HTML Parser)
- 构建DOM树(DOM Tree)
- 渲染树构建(Render Tree)
- 绘制渲染树(Painting)

很显然，浏览器渲染的过程不仅仅只有这些，还有js，css等文件的加载。

## 像素管道
>There are five major areas that you need to know about and be mindful of when you work. They are areas you have the most control over, and key points in the pixels-to-screen pipeline

[pixel pipeline](https://developers.google.com/web/fundamentals/performance/rendering/)，也是经常被提到的 Rendering Pipeline，直译过来就是像素管道，换句话来说就是像素到屏幕上的过程，即浏览器渲染过程。如下图所示：

![rendering pipeline](./images/rendering-pipeline.jpg)

- **JavaScript**，载入与执行JS/CSS。一般来说，我们会使用 JavaScript 来实现一些视觉变化的效果。比如用 jQuery 的 animate 函数做一个动画、对一个数据集进行排序或者往页面里添加一些 DOM 元素等。当然，除了 JavaScript，还有其他一些常用方法也可以实现视觉变化效果，比如：CSS Animations、Transitions 和 Web Animation API。
- **样式计算**，根据js与css计算style。此过程是根据匹配选择器（例如 .headline 或 .nav > .nav__item）计算出哪些元素应用哪些 CSS 规则的过程。从中知道规则之后，将应用规则并计算每个元素的最终样式。
- **布局**，当style作用到元素上时，浏览器会检查是否影响整个页面的排列，有影响的话需要重新布局。在知道对一个元素应用哪些规则之后，浏览器即可开始计算它要占据的空间大小及其在屏幕的位置。网页的布局模式意味着一个元素可能影响其他元素，例如 <body> 元素的宽度一般会影响其子元素的宽度以及树中各处的节点，因此对于浏览器来说，布局过程是经常发生的。
- **绘制**，将元素排列好之后需要对有改动的元素重新绘制。绘制是填充像素的过程。它涉及绘出文本、颜色、图像、边框和阴影，基本上包括元素的每个可视部分。绘制一般是在多个表面（通常称为层）上完成的。
- **合成**，将所有的层渲染到页面上形成最终显示的图像。由于页面的各部分可能被绘制到多层，由此它们需要按正确顺序绘制到屏幕上，以便正确渲染页面。对于与另一元素重叠的元素来说，这点特别重要，因为一个错误可能使一个元素错误地出现在另一个元素的上层。

而实际上最终的Render Tree是由 DOM Tree 与 CSSOM 结合而成。下面分别介绍一下这两个概念。

### DOM Tree
我相信大多数人应该都知道 DOM Tree ，浏览器解析HTML结构生成一个树形的DOM，如下图：
![DOM Tree](./images/DOM-Tree.png)
最终整个流程输出的也就是当前页面的文档对象模型（DOM），之后浏览器的操作基本都基于这个DOM对象。

### CSSOM
与处理 HTML 时一样，我们需要将收到的 CSS 规则转换成某种浏览器能够理解和处理的东西。因此，我们会重复 HTML解析 过程，不过是为 CSS 而不是 HTML：
![CSSOM-Tree](./images/CSSOM-Tree.png)
至于为什么最终也是一个树形的结构呢？其实不难理解，因为在最终计算生效的规则时，总是从上到下来确定最终的样式，也就是所说的样式继承。页面上的任何元素的样式一定是 通用样式（User Agent样式） + 继承样式（可能没有）+ 自定义样式 这样的形式，且优先级逐渐递增。

最终的 Render Tree 就是下图的样子：
![render tree](./images/render-tree-construction.png)

由此我们可以发现文章最初提到的浏览器渲染的4个步骤其实是不准确的，并没有体现样式计算即CSSOM构建这一过程。猜想应该是将这个步骤并入了渲染树的构建。

## 重排，重绘
有了上面的概念，不难发现所谓的页面交互其实就是 Render Tree 的变化，本质则是 DOM Tree 或者 CSSOM 的变化。而变化会触发Rendering Pipeline中特定步骤的重新执行，重新执行布局这一步骤就称为重排，也称为回流，重新触发绘制这一步骤就称为重绘。下面来介绍得更加具体一点：

### 回流 (reflow)
当渲染树（render Tree）中的一部分(或全部)因为元素的规模尺寸，布局，隐藏等改变而需要重新构建。这就称为回流（reflow），也就是重新布局（relayout），这个时候会重新执行一遍上面的Rendering Pipeline。

回流过程：JS / CSS > Style > Layout > Paint > Composite
![rendering pipeline](./images/rendering-pipeline.jpg)

每个页面至少需要一次回流，就是在页面第一次加载的时候。在回流的时候，浏览器会使渲染树中受到影响的部分失效，并重新构造这部分渲染树，完成回流后，浏览器会重新绘制受影响的部分到屏幕中，该过程称为重绘。

### 重绘（repaint）
当render tree中的一些元素需要更新属性，而这些属性只是影响元素的外观，风格，而不会影响布局的，比如 background-color 。则就叫称为重绘。重绘对应则不会触发layout过程。

重绘过程：JS / CSS > Style > Paint > Composite
![repaint](./images/frame-no-layout.jpg)

**值得注意的是，回流必将引起重绘，而重绘不一定会引起回流。** 明显，回流的代价更大，简单而言，当操作元素会使元素修改它的大小或位置，那么就会发生回流。

具体这两个步骤是由哪些改变触发的，可以看第一条引用的参考资料，这里就不详细说明了。

## 层（layer）
相对来说这个概念比较偏硬件，涉及浏览器渲染机制。先上个图：

![render layers](./images/renderLayer.png)

在 **Chrome** 中其实有几种不同的层类型：
- RenderLayers 渲染层，这是负责对应 DOM 子树
- GraphicsLayers 图形层，这是负责对应 RenderLayers子树。

可以看到上面会将DOM tree转换为 RenderObjects。RenderObjects 保持了树结构，一个 RenderObjects 知道如何绘制一个 node 的内容， 他通过向一个绘图上下文（GraphicsContext）发出必要的绘制调用来绘制 nodes。拥有相同坐标空间的RenderObjects，属于同一个渲染层RenderLayers。RenderLayers 来保证页面元素以正确的顺序合成，这时候就会出现层合成（composite），从而正确处理透明元素和重叠元素的显示。

而某些特殊的渲染层会被认为是合成层（Compositing Layers），合成层拥有单独的 GraphicsLayer，而其他不是合成层的渲染层，则和其第一个拥有 GraphicsLayer 父层公用一个。而每个GraphicsLayer（合成层单独拥有的图层） 都有一个 GraphicsContext，GraphicsContext 会负责输出该层的位图，位图是存储在共享内存中，作为[纹理](https://zh.wikipedia.org/zh-cn/%E6%9D%90%E8%B4%A8%E8%B4%B4%E5%9B%BE)上传到 GPU 中，最后由 GPU 将多个位图进行合成，然后显示到屏幕上。

既然合成层拥有单独的GraphicsLayer，所以合成层的有点也就不言而喻了：
1. 合成层的位图，会由GPU处理，比CPU处理要快
2. 当需要 repaint 时，只需要 repaint 本身，不会影响到其他的层

也就是说**一般一个元素变成合成层之后，可以独立于普通文档流中，改动后可以避免整个页面重绘，提升性能**。

那么如何让一个元素变成合成层呢，也就是所谓的 **硬件加速**。开启硬件加速的方式也有很多种，如常见的 3D transform变换，opacity动画，will-change等，除此之外filter, iframe等也可以做到，具体参照 [无线性能优化: composite](http://taobaofed.org/blog/2016/04/25/performance-composite/)这篇文章然当前合成层也并非万能的，使用的过程中也会有一些坑，具体可以参考[浏览器渲染过程&composite](https://segmentfault.com/a/1190000014520786)这篇文档。

## 总结
其实不难发现，渲染这种浏览器行为最终分析的时候始终还是离不开浏览器的。而且最后 层 的介绍中还涉及了 CPU 及 GPU 的一些知识，个人觉得这些只需要了解就好，因为就目前而言，这些阶段并不是前端工程师的可控范围，虽然可以通过某些特定的属性来生成合成层，但其实底层还是不受控制的。但是作为浏览器渲染相关的面试题来说，说到这里应该是可以令面试官满意了。


## 参考资料
- [浏览器渲染原理](https://github.com/chokcoco/cnblogsArticle/issues/10)
- [渲染性能](https://developers.google.com/web/fundamentals/performance/rendering/)
- [浏览器渲染优化](https://blog.arvinh.info/2016/03/26/Front-end%20kata%2060fps%E7%9A%84%E5%BF%AB%E6%84%9F/)
- [无线性能优化: composite](http://taobaofed.org/blog/2016/04/25/performance-composite/)
- [浏览器渲染过程&composite](https://segmentfault.com/a/1190000014520786)