---
title: React源码解析（二）-- 虚拟DOM的生成
date: 2021-04-25 14:30:39
updated: 2021-04-25 14:30:39
tags: 前端
code_block_shrink: false
excerpt: 以以下片段为例子
---
## createElement
以以下片段为例子
```js
<div style={{
  width:120
}}>
  第一个children
  <span>第二个children</span>
</div>
```
通过babel转为
```js
React.createElement("div", {
  style: {
    width: 120
  }
}, "第一个children", 
React.createElement("span", null, "第二个children"));
```
虚拟DOM的生成主要依赖于createElement函数,dev的判断已经去除
```js
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    // 从config提取ref和key
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 除了key,ref,__self,__source以外的config转变为props
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }

  // 挂载defaultProps
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current, // 应该是有fiber中注入的 暂时不用理
    props,
  );
}

```

## ReactElement
```js
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    // 用来判断是否为React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  return element;
};
```

## 小知识点
### defaultProps
设置默认props值，使用方法
```js
// 函数组件
function Text(props){
  return <div>
    {
      props.text
    }
  </div>
}

Text.defaultProps = {
  text: '默认文案'
}

// class组件
class CfnText extends React.Component{
  static defaultProps = {
    text:'默认文案2'
  }

  render(){
    return <div>
      {this.props.text}
    </div>
  }
}
```

## 小结
`babel`把`jsx`转换为`React.createElement`，创建了一颗虚拟DOM树