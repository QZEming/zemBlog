---
title: vue loader配置及对vue文件（vue单文件组件）的处理
abbrlink: 561f0192
date: 2019-08-03 20:49:51
tags: vue
categories: 框架
---
通常情况下，我们会使用vue cli直接创建一个项目的脚手架，如果我们要自己用webpack来配置vue项目的话，就要用到vue loader
<!-- more -->
# vue loader的配置
---
首先要下载相应的依赖包
```
npm i -D vue-loader vue-template-compiler
```
每个vue包的新版本发布时都会有一个新的vue-template-compiler随之发布，所以这里也要下载对应的 vue-template-compiler以使vue-loader生成兼容的代码
vue-loader除了和其他的loader一样，需要在webpack.config.js中的rules对所有的vue文件配置使用，还需要在配置中添加vue loader的插件
```javascript
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
    module: {
        rules: [{
            // 对所有的vue文件使用vue-loader处理
            test: /\.vue$/,
            loader: "vue-loader"
        }]
    },
    plugins: [
        // 引入插件
        new VueLoaderPlugin()
    ]
}
```
# 处理资源路径
---
我们知道但单元件组件一般由三个部分组成，\<template>，\<script>，\<style>和自定义标签组成，而对于template中的资源URL，vue-loader会将其转换为webpack模块请求
如下面的代码
```html
<img src="../image.png">
```
变成下面的代码
```javascript
createElement('img', {
  attrs: {
    src: require('../image.png') // 现在这是一个模块的请求了
  }
})
```
默认情况下，下面的标签对应的属性会被转换为webpack模块请求
```javascript
{
  video: ['src', 'poster'],
  source: 'src',
  img: 'src',
  image: ['xlink:href', 'href'],
  use: ['xlink:href', 'href']
}
```
此外，如果为\<style>配置了css-loader，那么也会对其中的资源URL进行转换
## 转换规则
* 绝对路径：保留原样
	```"/images/foo.png"```：请求绝对路径images文件夹下的foo.png图片
* 以.开头的路径：会以相对路径的形式，请求对应的资源
	```"./images/foo.png"```：请求当前配置文件下的images文件夹下的foo.png图片
*  以~开头的路径：被看做模块依赖，会请求node_modules中的模块对应的资源
	```"~/npm-package/foo.png"```：请求node_modules中某个npm包下的foo.png图片
*  以@开头的路径：也被看做模块依赖，可以在webpack中给@配置别名，将资源指向配置的地方，在vue-cli搭建的脚手架下默认指向/src
	```"@/images/foo.png```：@对应的别名下的images文件夹下的foo.png图片
## 相关的loader
虽然将图片转换为模块来请求，但是图片始终不是JavaScript模块，所以要通过file-loader或者url-loader去处理他们。vue cli搭建的项目自动处理了这部分内容。
### file-loader
通过npm下载该依赖包后，在webpack中配置相应的内容
```javascript
module.exports = {
  module: {
    rules: [
      {
      // 对.png,.jpg,.jpeg,.gif的图片文件进行处理
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {},
          },
        ],
      },
    ],
  },
};
```
#### options选项
##### name
指定目标文件的自定义文件模板
例如
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: 'dirname/[hash].[ext]',
            },
          },
        ],
      },
    ],
  },
};
```
上面的配置将图片处理到dirname文件夹下，如```dirname/ 0dcbbaa701328ae351f.png```
除了使用字符串的写法，也可以使用方法来配置
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name(file) {
                if (process.env.NODE_ENV === 'development') {
                  return '[path][name].[ext]';
                }

                return '[hash].[ext]';
              },
            },
          },
        ],
      },
    ],
  },
};
```
##### outputPath
用于指定将放置目标文件的文件系统路径
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              outputPath: 'images',
            },
          },
        ],
      },
    ],
  },
};
```
通过这个配置将目标文件放置到images文件夹下
使用方法配置：
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              outputPath: (url, resourcePath, context) => {
                // `resourcePath`是绝对路径
                // `context`是存储静态资源 (`rootContext`) 或者 `context`的目录
                // 要使用相对路径的话，可以使用
                // const relativePath = path.relative(context, resourcePath);

                if (/my-custom-image\.png/.test(resourcePath)) {
                  return `other_output_path/${url}`;
                }

                if (/images/.test(context)) {
                  return `image_output_path/${url}`;
                }

                return `output_path/${url}`;
              },
            },
          },
        ],
      },
    ],
  },
};
```
##### publicPath
指定目标文件的自定义公共路径
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              publicPath: 'assets',
            },
          },
        ],
      },
    ],
  },
};
```
方法配置
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              publicPath: (url, resourcePath, context) => {
                // `resourcePath` is original absolute path to asset
                // `context` is directory where stored asset (`rootContext`) or `context` option

                // To get relative path you can use
                // const relativePath = path.relative(context, resourcePath);

                if (/my-custom-image\.png/.test(resourcePath)) {
                  return `other_public_path/${url}`;
                }

                if (/images/.test(context)) {
                  return `image_output_path/${url}`;
                }

                return `public_path/${url}`;
              },
            },
          },
        ],
      },
    ],
  },
};
```
##### context
指定自定义文件上下文，默认情况下使用当前目录
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              context: 'project',
            },
          },
        ],
      },
    ],
  },
};
```
##### emitFile
布尔类型，为true时发出文件（将文件写入系统），为false时加载程序将返回公共url，但不会发出该文件，默认为true
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              emitFile: false,
            },
          },
        ],
      },
    ],
  },
};
```
##### regExp
使用正则表达式匹配目标文件路径的部分，可以在name属性中使用[N]占位符来使用
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              regExp: /\/([a-z0-9]+)\/[a-z0-9]+\.png$/,
              name: '[1]-[name].[ext]',
            },
          },
        ],
      },
    ],
  },
};
```
如果使用了[0]，将代替整个被匹配的字符串，而[1]是被匹配的第一个字符串，[2]是第二个，以此类推
### url-loader
url-loader是webpack的加载器，用于将文件转换为base64 URI。
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
          },
        ],
      },
    ],
  },
};
```
#### options选项
##### fallback
当指定目标文件的大小超过limit属性设置的大小时，使用备用的loader，默认使用'file-loader'
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              fallback: 'responsive-loader',
            },
          },
        ],
      },
    ],
  },
};
```
后备的加载程序使用和url-loader相同的options
##### limit
可以为布尔值和数值，默认为undefined
为数值时，数值大小为限制文件的大小，当超过该大小时，会将图片编译成base64格式
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 200*1024,
            },
          },
        ],
      },
    ],
  },
};
```
这里的200*1024指的是200k大小
为布尔值时，选择启用或禁用转换
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: false,
            },
          },
        ],
      },
    ],
  },
};
```
##### mimetype
设置转换文件的MIME类型，如果没有指定，则使用文件扩展名来查找MIME类型
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              mimetype: 'image/png',
            },
          },
        ],
      },
    ],
  },
};
```

---
参考文档：[file-loader文档](https://github.com/webpack-contrib/file-loader)
				  [url-loader文档](https://github.com/webpack-contrib/url-loader)
				  [vue-loader文档](https://vue-loader.vuejs.org/)