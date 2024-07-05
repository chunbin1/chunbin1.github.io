---
title: Ts+React实践汇总
date: 2024-07-05 16:23:58
excerpt: 如何写类型完整的React
tags: [TypeScript]
code_block_shrink:  false
---
## React-Typescript如何写Canvas
[参考文档](https://hashnode.blainegarrett.com/html-5-canvas-react-refs-and-typescript-ckf4jju8r00eypos1gyisenyf)  -- 需要翻墙
```typescript
import React, { useRef, useEffect } from 'react';

const SimpleCanvasExample: React.FC<{}> = () => {
  let canvasRef = useRef<HTMLCanvasElement | null>(null);
  let canvasCtxRef = React.useRef<CanvasRenderingContext2D | null>(null);

  useEffect(() => {
    // Initialize
    if (canvasRef.current) {
      canvasCtxRef.current = canvasRef.current.getContext('2d');
      let ctx = canvasCtxRef.current;
      ctx!.beginPath();
      ctx!.arc(95, 50, 40, 0, 2 * Math.PI);
      ctx!.stroke();
    }
  }, []);

  return <canvas ref={canvasRef}></canvas>;
};

export default SimpleCanvasExample;
```

## 类型完整的React.forwardRef写法
需求：一个评论用的窗口，通过forwardRef传递方法给父组件调用
```jsx
export type CommentHandler = {
  openDrawer: () => void
  closeDrawer: () => void
}

const CommentDrawer: React.ForwardRefRenderFunction<CommentHandler, IProps> = (props,ref)=>{

 const openDrawer = ()=>{};
 const closeDrawer = ()=>{};
  useImperativeHandle(ref, () => {
    return {
      openDrawer,
      closeDrawer,
    }
  })
  return <div></div>
}

export default forwardRef(CommentDrawer)
```
使用
```jsx
const ref = useRef<React.ElementRef<typeof CommentDrawer>>(null)
ref.current?.openDrawer()
```