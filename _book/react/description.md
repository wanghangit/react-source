<<<<<<< HEAD
# 简介
=======
# React 分析

react 目前的主要逻辑已经拆分成几个单独的目录，每个模块负责不同的内容。做为第一个章，先来看 React 这个文件。直接看入口文件

```js
const React = {
  Children: {
    map,
    forEach,
    count,
    toArray,
    only,
  },
  createRef,
  Component,
  PureComponent,

  createContext,
  forwardRef,
  lazy,
  memo,

  useCallback,
  useContext,
  useEffect,
  useImperativeHandle,
  useDebugValue,
  useLayoutEffect,
  useMemo,
  useReducer,
  useRef,
  useState,

  Fragment: REACT_FRAGMENT_TYPE,
  StrictMode: REACT_STRICT_MODE_TYPE,
  Suspense: REACT_SUSPENSE_TYPE,

  createElement: __DEV__ ? createElementWithValidation : createElement,
  cloneElement: __DEV__ ? cloneElementWithValidation : cloneElement,
  createFactory: __DEV__ ? createFactoryWithValidation : createFactory,
  isValidElement: isValidElement,

  version: ReactVersion,

  unstable_ConcurrentMode: REACT_CONCURRENT_MODE_TYPE,
  unstable_Profiler: REACT_PROFILER_TYPE,

  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED: ReactSharedInternals,
};
```

我们在写代码时都要在文件顶部引入`import React`,但好像我们并没有直接使用他，这是因为 jsx 语法在经过 `babel` 编译时，帮我们做了转化，比如说我们返回了这样一个组件

```javascript
// js
<div>
  <span className="demo">demo</span>
</div>;

// 经过编译后会转化成下边这样

React.createElement(
  "div",
  {},
  React.createElement("span", { className: "demo" }, "demo")
);
```

所以即使写一个很简单的组件，不直接用任何方法，我们也要引入react就是这个原因

这么多api常用的就下边几个打算分析下

[createElement](createElement.md)
[Component](Component.md)


>>>>>>> a78ab47191f3e894924d2f50c2b4937cec6503a2
