# Shadow DOM Word Count Component

这是一个使用 Web Components 和 Shadow DOM 实现的单词计数组件。通过扩展原生段落元素实现实时字数统计功能。

## 基础 HTML 结构

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Shadow DOM Word Count Component</title>
  </head>
  <body>
    <!-- 可编辑的文章区域 -->
    <article contenteditable="">
      <h2>Sample heading</h2>
      <!-- 示例段落 -->
      <p>Your content here...</p>
      
      <!-- 使用自定义组件 - 注意这里使用 is 属性来扩展原生 p 标签 -->
      <p is="word-count"></p>
    </article>
  </body>
</html>
```

## JavaScript 实现

```javascript
// 创建自定义元素类，继承自 HTMLParagraphElement
class WordCount extends HTMLParagraphElement {
    constructor() {
        // 必须首先调用 super()
        super();

        // 获取父元素（用于计数）
        const wcParent = this.parentNode;
        
        /**
         * 计算文本中的单词数量
         * @param {Node} node - DOM 节点
         * @returns {number} 单词数量
         */
        function countWords(node) {
            const text = node.innerText || node.textContent;
            return text.trim().split(/\s+/g).filter(a => a.trim().length > 0).length;
        }

        // 格式化计数文本
        const count = `Words: ${countWords(wcParent)}`;

        // 创建 Shadow Root
        const shadow = this.attachShadow({mode: 'open'});

        // 创建文本节点并添加计数
        const text = document.createElement('span');
        text.textContent = count;

        // 将文本节点添加到 Shadow Root
        shadow.appendChild(text);

        // 监听内容变化并更新计数
        this.parentNode.addEventListener('input', () => {
            text.textContent = `Words: ${countWords(wcParent)}`;
        });
    }

    /**
     * 生命周期回调：元素被添加到文档时
     */
    connectedCallback() {
        console.log('WordCount element added to the page.');
        
        // 创建新的 Shadow Root
        const shadow = this.attachShadow({ mode: "open" });

        // 创建组件结构
        const wrapper = document.createElement("span");
        wrapper.setAttribute("class", "wrapper");

        const icon = document.createElement("span");
        icon.setAttribute("class", "icon");
        icon.setAttribute("tabindex", 0);

        const info = document.createElement("span");
        info.setAttribute("class", "info");

        // 处理自定义文本属性
        const text = this.getAttribute("data-text");
        info.textContent = text;

        // 处理图标属性
        let imgUrl = this.hasAttribute("img") 
            ? this.getAttribute("img") 
            : "img/default.png";

        const img = document.createElement("img");
        img.src = imgUrl;
        icon.appendChild(img);

        // 添加外部样式
        const linkElem = document.createElement("link");
        linkElem.setAttribute("rel", "stylesheet");
        linkElem.setAttribute("href", "style.css");

        // 组装 Shadow DOM 结构
        shadow.appendChild(linkElem);
        shadow.appendChild(wrapper);
        wrapper.appendChild(icon);
        wrapper.appendChild(info);
    }

    /**
     * 生命周期回调：元素从文档中移除时
     */
    disconnectedCallback() {
        console.log('WordCount element removed from the page.');
    }

    /**
     * 生命周期回调：元素被移动到新文档时
     */
    adoptedCallback() {
        console.log('WordCount element moved to a new page.');
    }

    /**
     * 生命周期回调：元素属性变化时
     * @param {string} name - 变化的属性名
     * @param {string} oldValue - 原值
     * @param {string} newValue - 新值
     */
    attributeChangedCallback(name, oldValue, newValue) {
        console.log(`WordCount element attribute changed: ${name} from ${oldValue} to ${newValue}`);
    }
}

// 注册自定义元素
customElements.define('word-count', WordCount, { extends: 'p' });
```

## CSS 样式

```css
.wrapper {
    position: relative;
    display: inline-block;
}

.icon {
    display: inline-block;
    width: 20px;
    height: 20px;
    cursor: pointer;
}

.icon img {
    width: 100%;
    height: 100%;
}

.info {
    position: absolute;
    display: none;
    padding: 10px;
    background: #fff;
    border: 1px solid #ccc;
    border-radius: 4px;
}

.icon:hover + .info,
.icon:focus + .info {
    display: block;
}
```

## 核心技术详解

### 1. Shadow DOM 封装
#### 实现原理
```javascript
const shadow = this.attachShadow({mode: 'open'});
```
- **mode: 'open'** 
  - 允许外部 JavaScript 访问 Shadow DOM
  - 便于调试和外部控制
  - 可选 'closed' 实现完全隔离

#### 优势
- **样式隔离**
  - Shadow DOM 创建独立的作用域
  - 内部样式不会泄露到外部
  - 外部样式不会影响组件
  
#### 结构设计
```javascript
shadow.appendChild(linkElem);     // 外部样式
shadow.appendChild(wrapper);      // 容器元素
wrapper.appendChild(icon);        // 图标元素
wrapper.appendChild(info);        // 信息元素
```
- 分层结构确保组件的可维护性
- 便于样式管理和事件委托

### 2. 字数统计功能
#### 核心算法
```javascript
function countWords(node) {
    const text = node.innerText || node.textContent;
    return text.trim()           // 去除首尾空白
             .split(/\s+/g)      // 按空白字符分割
             .filter(a => a.trim().length > 0)  // 过滤空字符串
             .length;
}
```

#### 实时监听机制
```javascript
this.parentNode.addEventListener('input', () => {
    text.textContent = `Words: ${countWords(wcParent)}`;
});
```
- **事件选择**：使用 `input` 事件而不是 `change`
  - 实时响应用户输入
  - 提供即时反馈
  - 性能优化考虑

#### 性能优化
- 使用 `textContent` 替代 `innerHTML`
  - 更快的解析速度
  - 避免 XSS 风险
- 考虑使用防抖（Debounce）
  ```javascript
  const debounce = (fn, delay) => {
      let timer;
      return (...args) => {
          clearTimeout(timer);
          timer = setTimeout(() => fn(...args), delay);
      };
  };
  ```

### 3. 自定义属性支持
#### 属性配置
```javascript
// 提示文本配置
const text = this.getAttribute("data-text");
info.textContent = text;

// 图标配置
let imgUrl = this.hasAttribute("img") 
    ? this.getAttribute("img") 
    : "img/default.png";
```

#### 属性监听
```javascript
static get observedAttributes() {
    return ['data-text', 'img'];
}

attributeChangedCallback(name, oldValue, newValue) {
    if (name === 'data-text') {
        this.updateTooltip(newValue);
    } else if (name === 'img') {
        this.updateIcon(newValue);
    }
}
```

#### 扩展性设计
- 支持动态属性更新
- 提供默认值机制
- 错误处理和回退方案

### 4. 生命周期管理
#### constructor
```javascript
constructor() {
    super();  // 必须首先调用
    this.state = {
        wordCount: 0,
        isVisible: false
    };
    this.initShadowDOM();
}
```
- 初始化组件状态
- 创建 Shadow DOM
- 设置初始属性

#### connectedCallback
```javascript
connectedCallback() {
    this.setupEventListeners();
    this.updateWordCount();
    console.log('组件已挂载');
}
```
- DOM 挂载完成时调用
- 设置事件监听
- 初始化UI状态

#### disconnectedCallback
```javascript
disconnectedCallback() {
    this.cleanupEventListeners();
    console.log('组件已卸载');
}
```
- 组件移除时调用
- 清理事件监听器
- 释放资源

#### attributeChangedCallback
```javascript
attributeChangedCallback(name, oldValue, newValue) {
    // 属性变化时的处理逻辑
    switch(name) {
        case 'data-text':
            this.updateTooltip(newValue);
            break;
        case 'img':
            this.updateIcon(newValue);
            break;
    }
}
```
- 响应属性变化
- 更新组件状态
- 触发UI更新

## 使用示例

### 基本用法
```html
<p is="word-count"></p>
```

### 带自定义属性
```html
<p is="word-count" 
   data-text="点击查看详情" 
   img="custom-icon.png">
</p>
```

## 注意事项

1. 浏览器兼容性
   - Chrome 67+
   - Firefox 63+
   - Safari 12+
   - Edge 79+

2. 使用限制
   - 需要现代浏览器支持
   - 需要服务器环境运行
   - 注意图片路径配置

3. 最佳实践
   - 添加错误处理机制
   - 考虑降级方案
   - 优化性能

## 开发建议

1. 扩展功能
   - 添加更多自定义选项
   - 支持不同的计数规则
   - 添加动画效果

2. 性能优化
   - 使用节流/防抖
   - 优化 DOM 操作
   - 减少不必要的重渲染

3. 可访问性
   - 添加 ARIA 属性
   - 支持键盘导航
   - 提供屏幕阅读器支持
