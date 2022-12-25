[toc]

# React 脚手架和配置

- React脚手架: create-react-app
- Vue脚手架: @vue/cli
- Angular: @angular/cli



- 这些脚手架都是使用Node编写的, 并且都是基于webpack的, 所以想要运行必须先装node环境 

- 创建的项目名称不能包含大写字母



## 一. 了解PWA

- PWA(Progressive Web App) 渐进式WEB应用, 可以通过Web技术编写出一个网页应用(一般用于移动端)
- App Manifest和Service Worker来实现PWA的安装和离线功能
- 作用:
  - 可以在桌面通过打开应用的方式来打开网页
  - 实现离线缓存功能
  - 实现消息推送




## 二. 初始化文件结构

```
01_test
├── README.md
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico		 // 应用顶层的icon图标
│   ├── index.html		 // 被配置成webpack的入口文件
│   ├── logo192.png		 // 在manifest.json中使用
│   ├── logo512.png		 // 在manifest.json中使用
│   ├── manifest.json    // 与PWA有关, 和web app配置相关
│   └── robots.txt		 // 告诉别人我们这个网站哪些能被爬虫,哪些不能被爬
├── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    ├── reportWebVitals.js	 // 发送一些包(待学习)
    └── setupTests.js		 // 测试初始化文件


```

