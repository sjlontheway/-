# webpack 核心

## entry 概念

Entry是webpack 文件打包的入口文件,webpack 会根据入口文件中依赖的文件，生成依赖树，经过层层处理，最终输出静态资源。

单入口打包的entry属性是一个字符串 ``` entry:/path/entry.js ``` ,多入口entry配置是一个对象 ``` entry:{ name1:/path/entry1.js,name2:/path/entry2 } ```

## output 概念

output 参数是webpack将打包后产物输出到磁盘何处以及输出产物的名称
一般会配置为

``` js
//单入口
output: {
    filename:'path.js',
    path: /path/to/dist
}

//多入口
output: {
    filename:'[name].js',
    path: /path/to/dist

}
```

## Loader 概念

webpack 默认只识别js以及json等格式代码，loader支持webpack识别更多不同文件格式的内容，将其加入到项目依赖中去

用法如下：

``` js
   module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // [style-loader](/loaders/style-loader)
          { loader: 'style-loader' },
          // [css-loader](/loaders/css-loader)
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          // [sass-loader](/loaders/sass-loader)
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```

### plugins 概念

用来增强webpack功能,用于bundle优化，资源管理以及环境变量注入等，作用域整个构建过程

### Mode

用于指定webpack构建环境

- none 默认不做任何优化

- 将环境变量注入js，并且默认加载一些开发环境调试插件

- 将环境变量标识加入js，默认添加一些打包优化配置插件;
  
### 文件监听

通过监听文件系统指定文件的变化，来进行重新构建

```js
watch:true,
watchOptions:{

    //忽略文件变化监听目录
    ignored:'/node_modules',

    //文件变化后，延迟执行时长，默认300ms
    aggregateTimeout: 300,

    //轮询文件发生变化的事件
    poll:1000, //ms
}
```

### HMR

webpack热更新，提供功能监听文件变化后，自动刷新网页内容
主要配置为：

``` js
// yarn add webpack-dev-server
  devServer: {
    contentBase: './dist',
    hot: true
  },
```

![HMD原理]('../src/assert/HMD.png')

### 文件指纹(hash)

文件hash一般用来标识文件是否变化,当打包缓存更新时以此来标识。我们打包后的文件产物会有一段hash字符串，不同的hash类型区别是作用范围差别

- Hash： 与整个项目相关, 当项目文件有修改时，整个项目构建的hash值就会发生变化

- ChunkHash： 与打包的chunk有关系，我们在写入口文件时，有几个入口就会生成对应的chunk，每个chunk会有对应的chunkhash值, 当与chunk关联的文件变化时chunk的文件会发生变化

- ContentHash： 根据文件内容来定义hash，文件内容不变，hash值不变, csshash

### 代码压缩

- js 压缩：内置uglifyjs-webpack-plugin 内置压缩

- css压缩：optimize-css-assets-webpack-plugin, 同时使用cssnano 处理器

- html压缩: html-webpack-plugin
  
### 构建自动清理

当文件发生改变rebuild时，由于hash值变化，文件会累积越来越多，所以我们需要自动清理。
我们可以通过clean-webpack-plugin来清理output指定的目录

### 自动添加Css前缀

当在处理多个浏览器兼容性时，需要指定css3特定前缀，不同的浏览器需要添加指定的前缀来处理，如IE Trident(-ms), firefox GeKo(-moz), safri webkit(-webkit), Opera(-o),Chromium (blink)

postcss-loader,autoprefixer 自动补齐css3前缀，css module等功能

```js
modules:{
    rules:[
        {
            test:/'\.css$'/g,
            use: [
                'style-loader',
                'css-loader',
                'less-loader',
                {
                    loader：'postcss-loader',
                    options:{
                        plugins:()=>{
                            require('autoprefixer')({
                                browers:[
                                    'last 2 version',
                                    '>1%',
                                    'ios 7'
                                ]
                            })
                        }
                    }
                }
            ]
        }
    ]
}
```

### 移动端css px自动转rem

不同分辨率设备兼容性的处理
以前是通过媒体查询来处理不同分辨率设备，需要多套样式代码适配屏幕
rem :font-size of the root element

``` js
modules:{
    rules:[
        {
            test:/'\.css$'/g,
            use: [
                'style-loader',
                'css-loader',
                'less-loader',
                {
                    loader：'postcss-loader',
                    options:{
                        plugins:()=>{
                            require('autoprefixer')({
                                browers:[
                                    'last 2 version',
                                    '>1%',
                                    'ios 7'
                                ]
                            })
                        }
                    }
                },{
                    loader:'px2rem-loader',
                    options:{
                        remUnit: 75,  //1rem == 75px
                        remPrecesion: 8 // px 转 rem 小数点位数
                    }
                }
            ]
        }
    ]
}
```

### 静态资源内联

css、js、字体、图片内联至html页面
代码层面：页面框架初始化事情，上报事件，css内联避免页面闪动
请求层面：小图片内联

raw-loader (注意版本) 内联html

``` html
<script>${require('raw-loader!babel-loader!./meta.html')} </script>

<script>${require('raw-loader!babel-loader!../node_modules/lib-flexible')}</script>
```

css 内联：html-inline-css-webpack-plugin 