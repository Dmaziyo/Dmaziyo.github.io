---
title : '创建一个vue组件发布至npm'
date : 2023-10-27T11:33:55+08:00
tags: ["浅入系列"]
---
**本次笔记的原则：**
1. 创建基于成果的任务
2. 写下要做的行动
3. 记录执行过程中遇到的问题和思考

**目标**:发布一个能够展示上传图片的组件至npm上，并且能够在其他项目中使用

**Steps**

1.使用vue-cli初始化项目

2.实现能够上传图片并显示的组件
```
<template>
    <div class="special-upload">
       <h2>File Uploader</h2>   
       <input type="file" @change="loadFile"  accept="image/*" >
       <img ref="output" width="200">
    </div>
</template>
<script>
export default {
   name:"SpecialUpload",
   data() {
    return {
    }
   }, 
   methods: {
    loadFile(event){
        this.$refs.output.src = URL.createObjectURL(event.target.files[0])
    }
   },
}
</script>
<style>
 .special-upload {
    width: 300px;
    height: 400px;
    border: 1px solid black;
    text-align: center;
 }
</style>
```
3.使用vue-cli来实现打包配置

**遇到的问题：**
1. 在其他项目中使用打包后的js文件报错，关闭eslint解决，我一开始还以为是不支持使用umd,一开始无法使用可能是因为打包的时候出现了冗余的代码吧
```
//在package.json配置，使用脚手架提供的内置打包配置
"build-lib":"vue-cli-service build --target lib --name special-upload ./src/index.js",
```
2.配置package.json的入口文件和基本信息
```
{
  "name": "@dctmaziyo/special-upload",
  "version": "0.1.0",
  "private": false,
  "main": "./package/index.umd.js",
  "files": [
  "dist/*",
  "src/*",
  "public/*",
  "*.json",
  "*.js"
],
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lib": "vue-cli-service build --target lib --name index --dest package ./src/index.js",
    "lint": "vue-cli-service lint"
  },
}
```
3.使用npm link 进行本地测试

4.在测试项目中导入组件库
```
import Vue from 'vue'
import App from './App.vue'
import {MaziyoPlugin}   from '@dctmaziyo/special-upload';
Vue.config.productionTip = false
Vue.use(MaziyoPlugin)

new Vue({
  render: h => h(App),
}).$mount('#app')

```
![调用组件](https://github.com/Dmaziyo/blog-img/blob/main/component-lib1.jpg?raw=true)

效果展示
![](https://github.com/Dmaziyo/blog-img/blob/main/component-lib2.jpg?raw=true)
5. 发布npm包
```
npm publish --access public
```
![](https://github.com/Dmaziyo/blog-img/blob/main/component-lib3.jpg?raw=true)
6. 更新npm包的版本(因为有文件忘记添加进白名单了😅),在package.json中修改版本号重新publish就行了

7. 本地测试安装组件库,并且成功运行了
```
npm i @dctmaziyo/special-upload
```
![](https://github.com/Dmaziyo/blog-img/blob/main/component-lib4.jpg?raw=true)
