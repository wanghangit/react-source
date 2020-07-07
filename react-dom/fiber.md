# Fiber

这个概念是React16引入进来，让框架可以更细粒度的调节任务调度，数据结构就是下边这样调用ReactDOM.render就会生成一个FiberRoot节点，FiberRoot的current属性指向一个RootFiber节点，每个fiber节点只有一个子节点，如果有同级的节点是通过fiber的sibiling指向的，return指向当前节点的父节点，这样以类似链表的方式将一颗树串联起来，
react中很多数据结构都是使用**链表**实现的
![Fiber树结构](./fiber.webp)