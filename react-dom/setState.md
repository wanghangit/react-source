# setState

在使用class组件时，使用最多的应该就是这个api，我们用这个来更新组件的state，在看Component的时候看到过这个方法，其实就是调用`this.updater.enqueueSetState`来触发更新。这个update具体怎么注入进来的需要在renderFiber阶段看，下边直接贴出实际调用的方法,replaceState,forceUpdate类似就不看了，可以看到这个方法和render流程的最后几部很类似，都是生成一个update对象加入到当前Fiber的更新队列中，等到更新阶段取出来更新。

```js
// class组件的Update方法
const classComponentUpdater = {
  isMounted,
  enqueueSetState(inst, payload, callback) {
    const fiber = getInstance(inst); // 获取当前fiber实例
    const currentTime = requestCurrentTime();
    const expirationTime = computeExpirationForFiber(currentTime, fiber);

    const update = createUpdate(expirationTime); // 创建更新对象
    update.payload = payload; // 这个就是设置的对象或者函数
    if (callback !== undefined && callback !== null) {
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'setState');
      }
      update.callback = callback;
    }

    flushPassiveEffects();
    enqueueUpdate(fiber, update); // 加入队列
    scheduleWork(fiber, expirationTime); // 开启调度任务
  },
};
```