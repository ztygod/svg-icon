在我们的项目中经常需要从外部引入一些icon，就目前来说相比于unicode（不支持多色图标，在不同的设备浏览器字体的渲染会略有差别）与font-class，我们更常使用svg-icon，svg也正在成为主流的方式。

#### svg的优点也很明显

- 支持多色图标了，不再受单色限制。
- 支持像字体那样通过font-size,color来调整样式。
- 支持 ie9+
- 可利用CSS实现动画。
- 减少HTTP请求。
- 矢量，缩放不失真
- 可以很精细的控制SVG图标的每一部分

## 组件实现

现在我们可以去网上下载svg图标，此外要考虑的点就是如何优雅的在组件中实现svg，我们可以封装一个svg-icon，以作复用，下面是具体实现：
组件：

```vue
<template>
    <svg :class="svgClass" aria-hidden="true">
        <use :xlink:href="iconName"></use>
    </svg>
</template>

<script>
export default {
    name: 'SvgIcon',
    props: {
        iconClass: {
            type: String,
            require: true,
        },
        className: {
            type: String,
            default: ''
        }
    },
    computed: {
        iconName() {
            return `#icon-${iconClass}`
        },
        svgClass() {
            if (this.className) {
                return 'svg-icon' + this.className
            } else {
                return 'svg-icon'
            }
        }
    }
}
</script>

<style scoped>
.svg-icon {
    width: 1em;
    height: 1em;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
}
</style>
```

这样我们就得到了最基础的svg组件，同时为了使组件能够在全局中使用，同时做到自动导入，我们做以下操作。

## 使用svg-sprite

它是一个 webpack loader ，可以将多个 svg 打包成 `svg-sprite` 。
但是`vue-cli`默认情况下会使用 `url-loader` 对svg进行处理，会将它放在`/img` 目录下，所以这时候我们引入`svg-sprite-loader` 会引发一些冲突。
安全合理的做法是使用 webpack 的 [exclude](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Fconfiguration%2Fmodule%2F%23rule-exclude) 和 [include](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.js.org%2Fconfiguration%2Fmodule%2F%23rule-include) ，让`svg-sprite-loader`只处理你指定文件夹下面的 svg，`url-loaer`只处理除此文件夹之外的所以 svg，这样就完美解决了之前冲突的问题。 代码如下。

```javascript
 // set svg-sprite-loader
    config.module
      .rule('svg')
      .exclude.add(resolve('src/icons'))
      .end()
    config.module
      .rule('icons')
      .test(/\.svg$/)
      .include.add(resolve('src/icons'))
      .end()
      .use('svg-sprite-loader')
      .loader('svg-sprite-loader')
      .options({
        symbolId: 'icon-[name]'
      })
      .end()
```

## 自动导入

放置图标 icon 的文件夹如：`@/src/icons`，将所有 icon 放在这个文件夹下。 之后我们就要使用到 webpack 的 require.context。
自动引入 `@/src/icons` 下面所有的图标了：
```javascript
const requireAll = requireContext => requireContext.keys().map(requireContext)
const req = require.context('./svg', false, /\.svg$/)
requireAll(req)
```
