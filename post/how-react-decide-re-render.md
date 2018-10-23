React以其性能著称。因为它有虚拟DOM，并且只在需要的时候才去更新真正的DOM，所以它始终可以比直接更新DOM快得多。然而，React也就这么智能了（此刻！），所以我们要了解它的模式和限制，这样才不会在某些原因下破坏了它的性能。

我们需要注意的一个方面是React如何决定何时重新渲染组件。不是在“更新DOM渲染”中，而只是调用render方法来更改虚拟DOM。我们应该告诉它什么时候应该渲染。我们再来看看这两个...

### 1.组件的状态改变

重新渲染只能在组件状态发生变化时触发。组件状态可以从改变props改变，或者从一个直接的setState变化。组件获取更新状态，React决定是否应该重新渲染组件。不幸的是，默认情况下，React是非常简单的，基本上重新渲染所有的东西。

组件已更改？重新渲染。父元素改变了？重新渲染。实际上不影响视图的部分改变了？重新渲染。
```
class Todo extends React.Component {

    componentDidMount() {
        setInterval(() => {
            this.setState(() => {
                console.log('setting state');
                return { unseen: "does not display" }
            });
        }, 1000);
    }

    render() {
        console.log('render called');
        return (<div>...</div>);
    }
}
```
在这个例子中Todo，即使该render方法根本不使用，它也会每秒重新渲染unseen。其实，unseen甚至没有改变它的价值！

但是一直重新渲染是没有任何帮助的...

我的意思是，我很欣赏React非常仔细。如果状态发生变化，而组件进行重新渲染，情况会更糟糕。我怎么知道我的朋友寄给我的新消息？我会错过的，而且她可能会认为这是故意的，那么她就不再和我说话，整个友谊就会毁了。这样做的风险很高。重新渲染绝对是安全的选择。

但重新渲染似乎影响性能。

是的，重新渲染不必要的生命周期，通常不是一个好主意。然而，React不知道何时可以忽略部分状态。因此，只要状态发生变化，重要或不重要，它就全部重新呈现。

我们如何告诉React跳过重新渲染？

那好吧，我们来讲第二点...

### 2. shouldComponentUpdate方法
默认情况下，shouldComponentUpdate返回true。这就是我们上面看到的“一直更新所有东西”的原因。但是如果你需要提升性能，shouldComponentUpdate可以更智能。与其让react重新渲染所有组件，你可以告诉react，你不希望触发re-render。

当React渲染组件时，它会执行shouldComponentUpdate，看看它是否返回true（组件应该更新，也就是重新渲染）或false（React可以跳过这次重新渲染）。所以你需要重写shouldComponentUpdate返回true还是false来告诉React什么时候重新渲染以及什么时候跳过。

当你使用shouldComponentUpdate的时候，你需要决定哪些数据实际上对于重新渲染很重要。让我们回到我们的例子：
```
class Todo extends React.Component {

    componentDidMount() {
        setInterval(() => {
            this.setState(() => {
                console.log('setting state');
                return { unseen: "does not display" }
            });
        }, 1000);
    }

    shouldComponentUpdate(nextProps) {
        const differentTitle = this.props.title !== nextProps.title;
        const differentDone = this.props.done !== nextProps.done
        return differentTitle || differentDone;
    }

    render() {
        console.log('render called');
        return (<div>...</div>);
    }
}
```

正如你所看到的，我们只需要在title或done属性发生了变化才重新渲染Todo如果。我们不在乎unseen是否改变了，所以我们不把它写在shouldComponentUpdate里。

当React渲染一个Todo组件时（如由setState触发），它将首先检查状态是否已经改变（通过props或state）。假设状态不同（这是因为我们做了一个明确的setState调用），React将检查shouldComponentUpdateTodo组件。React会评估shouldComponentUpdate是否是真的，并决定在那里渲染。

更新后的代码中setState仍然会每一秒都执行一次，但render只能在初始加载（或当title或done属性发生改变时）。

这个例子特别详细，因为有两个我们关心的属性（title和done），只有一个属性我们需要忽略（unseen）。根据你的数据，只检查一个或两个属性可能会更有意义。

> 返回false不会阻止子组件在其状态更改时重新渲染。

这适用于子组件的state但不是他们的props。所以如果一个子组件在内部管理其状态的某个方面（用setState它自己的状态），那么它仍然会被更新。但是，如果父组件返回false，shouldComponentUpdate则不会将更新传递props给子组件，因此，即使子组件props已经更新，子组件也不会重新渲染。

### 简单的性能测试
编写和运行计算shouldComponentUpdate可能是昂贵的，所以你应该确保他们值得的时间。在编写任何shouldComponentUpdates 之前，您可以检查React在默认情况下的浪费周期数。通过这些信息来指导你，你可以知道哪些组件渲染太频繁而导致性能瓶颈。

使用React的性能工具来查找浪费的周期：

```
Perf.start()
// Do the render
Perf.stop()
Perf.printWasted()
```

哪些组件浪费了很多渲染周期？你怎么能让他们更聪明的shouldComponentUpdate？尝试一些方法，并确保使用性能工具相互检查它们！

### PS
最后讲一讲functional stateless components和stateless class-based components在ShouldComponentUpdate的差异。
```
const FuncComponent = ({ some, props, here}) => <fancymarkup/>
```

```
class ClassicalComponent extends Component {
  render () {
    return <fancymarkup/>
  }
}
```
这两个组件表现上一致。实际上，ClassicalComponent实现了ShouldComponentUpdate方法并且可以排除任何不必要的re-render。所以，如果你有一个使用了大量stateless functional components的大型APP，将这些functional components重写成class-based components，你会明显的看到性能提升。