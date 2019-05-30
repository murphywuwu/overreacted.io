---
title: 函数式组件和类组件有何不同
date: '2019-03-03'
spoiler: 它们是完全不同的
---

React的函数式组件和React的类组件有何不同?

有一段时间，规范的答案是类组件可以提供更多功能(比如state)。有了[Hooks](https://reactjs.org/docs/hooks-intro.html), 这不再是真的了。

也许你听说过，其中一个比另一个性能更好。哪一个更好呢？许多此类[基准](https://medium.com/@dan_abramov/this-benchmark-is-indeed-flawed-c3d6b5b6f97f?source=your_stories_page---------------------------)都存在缺陷，因此我会小心地从中得出[结论](https://github.com/ryardley/hooks-perf-issues/pull/2) from them. Performance primarily depends on what the code is doing rather than whether you chose a function or a class. In our observation, the performance differences are negligible, though optimization strategies are a bit [different](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)。性能主要取决于代码做了什么，而不是你选择的是函数组件还是类组件。在我们的看法中，虽然优化策略有点[不同](https://reactjs.org/docs/hooks-faq.html#are-hooks-slow-because-of-creating-functions-in-render)，但性能差异可以忽略不计。

在任何一种情况下，除非你有其他原因并且不介意成为早期用户，否则我们[不建议](https://reactjs.org/docs/hooks-faq.html#should-i-use-hooks-classes-or-a-mix-of-both) rewriting your existing components unless you have other reasons and don’t mind being an early adopter. Hooks are still new (like React was in 2014)重写现有组件。Hooks仍然很新(就像2014年的React一样)，并且一些“最佳实践”尚未进入教程。

那么我们离开了哪里呢？React函数组件和类组件之间是否有任何根本区别? 当然有， - 他们有着不一样的心智模型。**这篇文章中, 你将看到他们之间最大的区别**。自2015年p[介绍](https://reactjs.org/blog/2015/09/10/react-v0.14-rc1.html#stateless-function-components)函数式组件以来，它一直存在，但它经常被忽视。

>**函数式组件捕获渲染值**

让我们接着往下探索，这到底是什么意思。

---

**注意: 这篇文章不是对类组件和函数式组件的价值判断。我只是描述了React中这两种编程模型之间的区别。 有关更广泛采用函数式组件的问题，请参阅[Hooks FAQ](https://reactjs.org/docs/hooks-faq.html#adoption-strategy)。**

---

考虑这个组件:

```jsx
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

它展示了一个按钮，使用`setTimeOut`模型网络请求，然后显示一个确认alert弹窗。例如，如果`props.user`是`Dan`，它将在3秒后显示`Followed Dan`。很简单。

*(请注意，在上面的示例中我是否使用箭头或函数声明并不重要. `function handleClick()` 会以完全相同的方式工作。)*


我们如何把它写成一个类组件？天真的翻译可能如下所示：

```jsx
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };

  handleClick = () => {
    setTimeout(this.showMessage, 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

通常认为这两个代码片段是等价的。人们通常在这些模式之间自由地重构，而不会注意到他们的含义：

![Spot the difference between two versions](./wtf.gif)

**但是, 这两个代码片段略有不同。** 好好看看他们。你看到了差异吗？ 就个人而言, 我花了一段时间才看到。

**前面有剧透，所以如果你想自己搞清楚，这里是一个[现场演示](https://codesandbox.io/s/pjqnl16lm7)**。本文的其余部分解释了差异的重要性及其重要性。

---

在我们继续之前，我想强调一点，我所描述的差异与React Hooks本身无关。以上示例甚至不使用Hooks！

这就是React中函数式组件和类组件之间的区别。如果您计划在React应用程序中更频繁地使用函数式组件，则可能需要了解它。

---

**我们将通过React应用程序中常见的bug说明其问题。**

打开这个**[example sandbox](https://codesandbox.io/s/pjqnl16lm7)** ，

Open this **[example sandbox](https://codesandbox.io/s/pjqnl16lm7)，可以看到当前的profile选择器和两个`ProfilePage`的代码实现，每个都渲染了一个Follow按钮。

使用两个按钮尝试以下操作顺序：

1. **点击**其中一个Follow按钮
2. **改变**在3秒之前更改所选的profile
3. **阅读**弹出文字

你会注意到一个特殊的区别

* 使用上面的 `ProfilePage` **函数式**组件, 点击 Follow on Dan’s profile 然后导航到Sophie’s，仍然会弹出 `'Followed Dan'`.

* 使用上面的`ProfilePage` **类**组件, 则会弹出`'Followed Sophie'`:

![Demonstration of the steps](./bug.gif)

---
