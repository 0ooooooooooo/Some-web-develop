# View Transitions API - startViewTransition

## 概述
`startViewTransition` 是 View Transitions API 的一部分，用于创建页面状态变化时的平滑过渡动画。这个 API 让开发者能够在 DOM 更新时创建动画效果，使用户体验更加流畅。

## 基本用法

```javascript
document.startViewTransition(() => {
    // 在这里更新 DOM
});
```

## 关键特性

### 1. CSS 伪元素
View Transitions API 提供了两个重要的伪元素：
```css
::view-transition-old(root)  // 表示过渡开始前的状态
::view-transition-new(root)  // 表示过渡结束后的状态
```

### 2. 动画控制
可以通过 CSS 来控制这些伪元素的动画：
```css
::view-transition-new(root), 
::view-transition-old(root) {
    animation: none; // 可以禁用默认动画
}
```

## 实际应用示例

以下是一个实现主题切换动画的完整示例：

### HTML 结构
```html
<button id="changeBackGround">click</button>
```

### CSS 设置
```css
:root {
    --bg-color: #fff;
    background-color: var(--bg-color);
}
:root.dark {
    --bg-color: #000;
}
```

### JavaScript 实现
```javascript
const button = document.getElementById('changeBackGround');
button.addEventListener('click', (e) => {
    // 1. 开始视图过渡
    const transition = document.startViewTransition(() => {
        document.documentElement.classList.toggle('dark');
    });

    // 2. 等待过渡准备就绪
    transition.ready.then(() => {
        // 获取点击坐标
        const { clientX, clientY } = e;
        
        // 计算最大半径
        const radius = Math.hypot(
            Math.max(clientX, innerWidth - clientX),
            Math.max(clientY, innerHeight - clientY),
        );

        // 3. 创建圆形扩展动画
        document.documentElement.animate(
            {
                clipPath: [
                    `circle(0% at ${clientX}px ${clientY}px)`,
                    `circle(${radius}px at ${clientX}px ${clientY}px)`,
                ]
            },
            {
                duration: 300,
                pseudoElement: '::view-transition-new(root)'
            }
        );
    });
});
```

## 高级特性

### 1. 过渡状态管理
View Transitions API 在过渡期间会给文档添加特定的类名：
- `view-transition-name`: 指定元素的过渡名称
- `.view-transition-group`: 包含新旧视图的容器
- `.view-transition-image-pair`: 新旧视图的配对容器
- `.view-transition-old`: 旧视图容器
- `.view-transition-new`: 新视图容器

### 2. 错误处理
```javascript
try {
    if (!document.startViewTransition) {
        throw new Error('View Transitions API not supported');
    }
    
    const transition = document.startViewTransition(async () => {
        // DOM 更新操作
    });
    
    await transition.finished; // 等待过渡完成
} catch (e) {
    // 降级处理
    console.log('View transition failed:', e);
}
```

### 3. 性能优化建议
1. **避免大量 DOM 操作**
   - 在过渡回调中尽量减少 DOM 操作
   - 优先使用 CSS 类切换而不是直接修改样式

2. **合理使用异步操作**
   ```javascript
   const transition = document.startViewTransition(async () => {
       const data = await fetchData(); // 异步获取数据
       updateUI(data); // 更新UI
   });
   ```

3. **动画性能优化**
   ```css
   ::view-transition-old(root),
   ::view-transition-new(root) {
       /* 使用 transform 和 opacity 实现动画以获得更好的性能 */
       transform-origin: center;
       will-change: transform;
   }
   ```

## 完整示例：图片画廊切换效果

这个示例展示了如何使用 View Transitions API 创建一个带有平滑过渡效果的图片画廊。

### HTML 结构
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image Gallery with View Transitions</title>
    <style>
        .gallery {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 1rem;
            padding: 1rem;
        }

        .gallery img {
            width: 100%;
            height: 200px;
            object-fit: cover;
            cursor: pointer;
            border-radius: 8px;
            transition: transform 0.2s;
        }

        .gallery img:hover {
            transform: scale(1.05);
        }

        .fullscreen {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: rgba(0, 0, 0, 0.9);
            display: none;
            justify-content: center;
            align-items: center;
        }

        .fullscreen.active {
            display: flex;
        }

        .fullscreen img {
            max-width: 90%;
            max-height: 90vh;
            object-fit: contain;
        }

        /* 过渡动画样式 */
        ::view-transition-old(image-expand),
        ::view-transition-new(image-expand) {
            animation: none;
            mix-blend-mode: normal;
            height: 100%;
            width: 100%;
            object-fit: cover;
        }

        ::view-transition-old(image-expand) {
            object-position: center;
        }

        ::view-transition-new(image-expand) {
            object-position: center;
        }
    </style>
</head>
<body>
    <div class="gallery">
        <img src="image1.jpg" alt="Image 1">
        <img src="image2.jpg" alt="Image 2">
        <img src="image3.jpg" alt="Image 3">
        <!-- 更多图片... -->
    </div>

    <div class="fullscreen">
        <img src="" alt="Fullscreen Image">
    </div>
</body>
</html>
```

### JavaScript 实现
```javascript
document.addEventListener('DOMContentLoaded', () => {
    const gallery = document.querySelector('.gallery');
    const fullscreen = document.querySelector('.fullscreen');
    const fullscreenImg = fullscreen.querySelector('img');
    let isFullscreen = false;

    // 检查浏览器支持
    const supportsViewTransitions = !!document.startViewTransition;

    // 处理图片点击
    gallery.addEventListener('click', async (e) => {
        if (e.target.tagName !== 'IMG') return;

        const clickedImg = e.target;
        
        // 设置过渡名称
        clickedImg.style.viewTransitionName = 'image-expand';
        fullscreenImg.style.viewTransitionName = 'image-expand';

        if (supportsViewTransitions) {
            await document.startViewTransition(async () => {
                // 更新全屏图片
                fullscreenImg.src = clickedImg.src;
                fullscreen.classList.add('active');
                isFullscreen = true;
            }).finished;
        } else {
            // 降级处理
            fullscreenImg.src = clickedImg.src;
            fullscreen.classList.add('active');
            isFullscreen = true;
        }

        // 清除过渡名称
        clickedImg.style.viewTransitionName = '';
        fullscreenImg.style.viewTransitionName = '';
    });

    // 处理全屏图片点击（关闭）
    fullscreen.addEventListener('click', async () => {
        if (!isFullscreen) return;

        const activeThumb = [...gallery.querySelectorAll('img')]
            .find(img => img.src === fullscreenImg.src);

        if (activeThumb && supportsViewTransitions) {
            activeThumb.style.viewTransitionName = 'image-expand';
            fullscreenImg.style.viewTransitionName = 'image-expand';

            await document.startViewTransition(async () => {
                fullscreen.classList.remove('active');
                isFullscreen = false;
            }).finished;

            activeThumb.style.viewTransitionName = '';
            fullscreenImg.style.viewTransitionName = '';
        } else {
            // 降级处理
            fullscreen.classList.remove('active');
            isFullscreen = false;
        }
    });

    // 处理ESC键关闭
    document.addEventListener('keydown', (e) => {
        if (e.key === 'Escape' && isFullscreen) {
            fullscreen.click();
        }
    });
});
```

### 关键实现细节

1. **视图过渡名称**
   - 使用 `view-transition-name` 将缩略图和全屏图片关联起来
   - 在过渡开始前设置，结束后清除

2. **降级处理**
   - 检查浏览器是否支持 View Transitions API
   - 提供基础的显示/隐藏功能作为降级方案

3. **性能优化**
   - 使用 `async/await` 处理过渡完成事件
   - 避免在过渡期间进行不必要的DOM操作

4. **用户体验**
   - 添加鼠标悬停效果
   - 支持ESC键关闭全屏显示
   - 使用 `object-fit` 确保图片正确显示

### 使用说明

1. 准备工作：
   - 准备图片资源
   - 确保图片路径正确
   - 根据需要调整网格布局参数

2. 自定义样式：
   - 可以修改 grid 布局参数适应不同屏幕
   - 调整过渡动画时间和效果
   - 自定义全屏背景样式

3. 扩展功能：
   - 可以添加图片预加载
   - 实现图片轮播功能
   - 添加手势支持
   - 集成图片懒加载

这个例子展示了 View Transitions API 在实际应用中的强大功能，创建了一个流畅的图片画廊体验。通过合理的降级处理，确保了在不支持该API的浏览器中也能正常使用基本功能。

## 调试技巧

1. **Chrome DevTools**
   - 使用 Elements 面板观察过渡类名
   - 使用 Performance 面板分析过渡性能

2. **常见问题排查**
   - 检查CSS选择器优先级
   - 验证DOM更新时机
   - 确认动画持续时间设置

## 最佳实践

1. **保持过渡简单**
   - 避免复杂的动画组合
   - 控制动画时长在300-500ms之间

2. **提供降级方案**
   - 为不支持的浏览器提供基础功能
   - 使用CSS动画作为备选方案

3. **性能考虑**
   - 避免在过渡期间进行重计算
   - 优先使用 CSS transform 属性
   - 合理使用 will-change 属性

## 重要概念解析

1. **transition.ready**
   - 返回一个 Promise
   - 在 DOM 更新完成且过渡准备就绪时解析

2. **Math.hypot()**
   - 计算所有参数平方和的平方根
   - 用于计算动画扩展的最大半径

3. **clipPath**
   - 使用圆形裁剪路径创建扩展效果
   - 从点击位置开始，扩展到覆盖整个视口

## 注意事项

1. 确保浏览器支持 View Transitions API
2. 动画duration建议保持在300ms左右以获得最佳用户体验
3. 可以通过CSS自定义过渡动画效果

## 参考资料
- [MDN - Document.startViewTransition](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/startViewTransition)
