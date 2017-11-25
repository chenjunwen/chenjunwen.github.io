---
title: vue自定义组件发布至npm
date: 2017-11-25 15:05:51
tags: vue
---

## 项目案例 [vue-alert-loading](vue-alert-loading)
vue-alert-loading是一个全局加载组件，以这个自定义组件作为演示；该组件已发布至npm，只需npm install vue-alert-alert --save即可使用，
详情请看[demo](https://chenjunwen.github.io/vue-alert-loading/)


## 项目准备

- nodejs的安装
- npm 安装
- vue-cli安装

## 初始化组件

    # 使用webpack-simple模板初始化，比较适合写自定义插件
    vue init webpack-simple vue-alert-loading
    cd vue-alert-loading
    npm install
    npm run dev
    
## 创建你的组件
    在src新建loaing文件夹
    在loading文件夹创建loading.vue
    在loading文件夹下创建index.js文件
    
> loading.vue

    <template>
      <div v-if="visible" :class="'cpt-loading-mask animated fadeInNoTransform '+direction"
           :style="{background: loadingMaskBg, 'z-index': 1000001, position: 'fixed'}">
        <div class="div-loading" :style="{width:loadingDivWidth+'px', background: loadingBg}">
          <p class="loading-title txt-textOneRow" :style="{color: titleColor}">{{title}}</p>
    
          <div v-if="type!=='pic'" class="loading origin" :style="{ height: originDivHeight+'px', width: originDivWidth +'px'}">
            <div class="div-loadingOrigin" v-for="n in originSize">
              <span :style="{'margin-top': '-10px','margin-left': '-10px', width: originWidth+'px', height: originHeight+'px', background:originBg}"></span>
            </div>
          </div>
    
          <div class="loading pic" :style="{ height: originDivHeight+'px', width: originDivHeight +'px'}" v-else>
              <img src="http://www.daiwei.org/index/images/logo/dw.png"  alt="loading" >
          </div>
          <p class="loading-discription txt-textOneRow" :style="{color: textColor}">{{text}}</p></div>
      </div>
    </template>
    
    <style scoped>
      # 这里用到了外部css需要的话，地址如下  https://github.com/chenjunwen/vue-alert-loading/tree/master/src/loading
      @import "./css/loading.css";
      @import "./css/animate.css";
    </style>
    
    <script>
      export default {
        name:'VueAlertLoading',
        props:{
          // 标题
          title: {
            type: String,
            default: '请稍等!'
          },
          // 标题颜色
          titleColor:{
            type: String,
            default: 'rgba(255, 255, 255, 0.7)'
          },
          // loading的描述
          text: {
            type: String,
            default: ''
          },
          // 描述文本颜色
          textColor:{
            type: String,
            default: 'rgba(255,255,255,0.7)'
          },
          // 方向，col纵向   row 横向
          direction:{
            type: String,
            default: 'col'
          },
          // 是否显示
          visible:Boolean,
          // 中间加载div的宽度
          loadingDivWidth: {
            type: Number,
            default: 260
          },
          // 圆点的个数
          originSize:{
            type: Number,
            default: 6
          },
          // 小圆点的宽度
          originWidth:{
            type: Number,
            default: 6
          },
          // 小圆点的高度
          originHeight:{
            type: Number,
            default: 6
          },
          // 小圆点的背景色
          originBg:{
            type:String,
            default:'rgb(254, 254, 254)'
          },
          // loadingDiv的宽度
          originDivWidth:{
            type: Number,
            default: 40
          },
          // loadingDiv的宽度
          originDivHeight:{
            type: Number,
            default: 40
          },
          //pic：图片   origin：圆点
          type:{
            type: String,
            default: 'origin'
          },
          // 中间的背景色
          loadingBg:{
            type:String,
            default:'rgba(0, 0, 0, 0.6)'
          },
    
          // 背景遮罩层颜色
          loadingMaskBg:{
            type:String,
            default:'rgba(0, 0, 0, 0.2)'
          }
        },
        data() {
          return {
    
          }
        }
      }
    </script>

> index.js # 这里主要是导出loading模块
    
    import Loading from './loading.vue'
    Loading.install = (Vue) => {
      Vue.component(Loading.name, Loading);
    }
    
    export default Loading;


## 修改配置项

> webpack.config.js

这里主要修改filename,你总不想你文件还叫build.js吧
部分代码展示：

    module.exports = {
      entry: './src/main.js',
      output: {
        path: path.resolve(__dirname, './dist'),
        publicPath: '/dist/',
        filename: 'vue-alert-loading.js'
      }
    }
    
> package.json

    {
      "name": "vue-alert-loading",
      "description": "Vue custom load components",
      # 版本号 如果publish的时候项目文件修改了的话这里一定要修改版本号不然会报错
      "version": "3.0.3",
      "author": "chenjunwen <18296456378@163.com>",
      "license": "MIT",
      # 这里需要改成公有，私有的需要收费
      "private": false,
      "scripts": {
        "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
        "build": "cross-env NODE_ENV=production webpack --progress --hide-modules"
      },
      "dependencies": {
      },
      "browserslist": [
        "> 1%",
        "last 2 versions",
        "not ie <= 8"
      ],
      # 项目地址
      "repository": {
        "type": "git",
        "url": "git+https://github.com/chenjunwen/vue-alert-loading.git"
      },
      # npm 搜索关键字
      "keywords": [
        "vue-alert",
        "vue-loading",
        "vue-alert-loading"
        ],
       # 配置main结点，如果不配置，我们在其他项目中就不用import XX from '包名'来引用了，只能以包名作为起点来指定相对的路径
      "main":"src/loading/index.js",
      
      "devDependencies": {
        "babel-core": "^6.26.0",
        "babel-loader": "^7.1.2",
        "babel-preset-env": "^1.6.0",
        "babel-preset-stage-3": "^6.24.1",
        "cross-env": "^5.0.5",
        "css-loader": "^0.28.7",
        "file-loader": "^1.1.4",
        "vue": "^2.4.4",
        "vue-loader": "^13.0.5",
        "vue-template-compiler": "^2.4.4",
        "webpack": "^3.6.0",
        "webpack-dev-server": "^2.9.1"
      },
      # 这里表示的是依赖安装，也就是该项目是基于vue模块下运行的 将dependencies里面的vue模块删除，不然到时候打包的话会将vue也打包进去
      "peerDependencies":{
        "vue": "^2.4.4"
      }
    }


    
    
## .npmignore文件
该文件主要是排除不需上传至npm的文件（可自定义，和.gitignore写法相同）
我的配置如下：

    .idea
    node_modules/
    index.html
    webpack.config.js
    .babelrc
    .editorconfig
    .npmignore
    .gitignore
    *.map
    app.css
    App.vue
    main.js
    
## 打包至本地

    npm run build
    
## 发布至npm
1. 在 [npm官网](https://www.npmjs.com/) 注册一个npm账号 填写邮箱并一定验证（如果未验证是上传不了的）
2. 切换到需要发包的项目根目录下，登录npm账号，输入用户名、密码、邮箱
       
       npm login
       
3. 发布之前先检查下npm有木有相同的包名，如果没有的话就恭喜你了，有的话那你就需要修改`package.json`的name了
    
    npm search vue-alert-loading
    
4. 如果没有的话就执行npm publish

5. 如果需要移除你的包的话就执行npm unpublish --force：移除一个发布包（也可以移除指定版本的包）只有24小时内有效，过了24小时就不能移除，至于为什么，自己百度去吧嘿嘿