[toc]

# Css知识

## 一. 修改浏览器默认滚动条

```scss
// 滚动条基本样式, 作用于滚动块和滚动条
::-webkit-scrollbar {
  --bar-width: 5px;
  width: var(--bar-width);
  height: var(--bar-width);
}
// 滚动轨道
::-webkit-scrollbar-track {
  background-color: transparent;
}
// 滚动块
::-webkit-scrollbar-thumb {
  background-color: var(--bar-color);
  border-radius: 20px;
  background-clip: content-box;
  border: 1px solid transparent;
}
```



## 二. 媒体查询

### 1. min-width

- 当屏幕大于等于`600px`时, 下面的类生效, 一个递增的过程

```scss
@media only screen and (min-width: 600px) {
    .tight-container {
        --window-width: 100vw;
        --window-height: var(--full-height);
        --window-content-width: calc(100% - var(--sidebar-width));

        @include container();

        max-width: 100vw;
        max-height: var(--full-height);

        border-radius: 0;
        border: 0;
    }
}
```

### 2. max-width

- 表示元素只有在小于等于`600px`时才会生效

```scss
@media only screen and (max-width: 600px) {
    
}
```



## 三. 选择直接子元素

- 利用`>`符号, 选择`div`元素的直接子元素`a`, 并为它的`:hover`状态编写样式

```css
div > a:hover {
  /* CSS 样式 */
}
```



## 四. 居中显示toast, 并给背景设置颜色和透明度

```scss
.modal-mask {
  z-index: 9999;
  position: fixed;
  top: 0;
  left: 0;
  height: var(--full-height);
  width: 100vw;
  background-color: rgba($color: #000000, $alpha: 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}
```

- 将这个类赋在一个临时添加的div元素上

```js
export function showModal(props: ModalProps) {
  const div = document.createElement("div");
  div.className = "modal-mask";
  document.body.appendChild(div);

  const root = createRoot(div);
  const closeModal = () => {
    props.onClose?.();
    root.unmount();
    div.remove();
  };

  div.onclick = (e) => {
    if (e.target === div) {
      closeModal();
    }
  };

  root.render(<Modal {...props} onClose={closeModal}></Modal>);
}

```

