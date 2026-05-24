---
title: 虚拟DOM生成dom
date: 2021-09-14 20:38:44
updated: 2021-09-14 20:38:44
tags: 前端
code_block_shrink: false
excerpt: 使用方法
---
## render
使用方法
```js
React.render(<App/>,document.getElementById('root'))
```
把`<App/>`挂载到`root节点上`
```js
export function render(
  element,  // 虚拟DOM
  container, // 挂载的节点
  callback, // render是有回调的
) {
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback,
  );
}
```
调用了`legacyRenderSubtreeIntoContainer`函数
## legacyRenderSubtreeIntoContainer
```js
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>, // 父节点
  children: ReactNodeList, // 子节点
  container: Container, // 挂载节点
  forceHydrate: boolean, // 是否Hydrate
  callback: ?Function,
) {
  let root: RootType = (container._reactRootContainer: any);
  let fiberRoot;
  // 第一次调用的时候传是dom节点，不存在_reactRootContainer属性
  if (!root) {
    // Initial mount
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate,
    );
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Initial mount should not be batched.
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // Update
    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  return getPublicRootInstance(fiberRoot);
}
```

一阵追踪后从`react-reconciler/src/ReactFiberReconciler`调用了`createContainer`
再一阵追踪调用了从`ReactFiberRoot.new.js`调用了
`createFiberRoot`->
`new FiberRootNode`创建一个根fiber节点 ->
`createHostRootFiber`创建后续节点并放在root的current里->返回了一个`fiber`节点

## updateContainer
然后我们来看看`updateContainer`做了什么,在`ReactFiberReconciler.js`中我们找到了这个方法
```js
  //  创建了update
  const update = createUpdate(expirationTime, suspenseConfig);
  update.payload = {element};

  // 把update推到cureent的更新队列
  enqueueUpdate(current, update);
  // 调度
  scheduleWork(current, expirationTime);
```
Update的结构：
   tag：更新类型，UpdateState、ReplaceState、ForceUpdate、CaptureUpdate
   payload：状态变更函数或新状态本身
   callback：回调，作用于 fiber.effectTag，并将 callback 作为 side-effects 回调
   expirationTime：deadline 时间，未到该时间点，不予更新
   suspenseConfig：suspense 配置
   next：指向下一个 Update
   priority：仅限于 dev 环境

 updateQueue的结构：
   baseState：先前的状态，作为 payload 函数的 prevState 参数
   baseQueue：存储执行中的更新任务 Update 队列，尾节点存储形式
   shared：以 pending 属性存储待执行的更新任务 Update 队列，尾节点存储形式
   effects：side-effects 队列，commit 阶段执行

## scheduleWork
