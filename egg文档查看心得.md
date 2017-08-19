### 目标
* #### stronloop（有个好的设计蓝图）, sails, hapi等
* #### 孕育出更多上层框架（不同领域出现不同的框架（类似于dcoker或者说是git commit, 积累沉底出领域优秀的东西））
* #### 帮主开发人员降低开发成本和维护成本（通过各种约定以及基础设施）

### 原则
* #### 专注于提供 Web 开发的核心功能和一套灵活可扩展的插件机制
* #### 约定优于配置

### 特点
* ### 提供基于 Egg 定制上层框架的能力
* #### 高度可扩展的插件机制
* #### 内置多进程管理
* #### 基于 Koa 开发，性能优异
* #### 框架稳定，测试覆盖率高
* #### 渐进式开发(值得一提)， 前端渐进增强

### egg 和 koa
* ####  扩展
    在基于 Egg 的框架或者应用中，我们可以通过定义 app/extend/{application,context,request,response}.js 来扩展 Koa 中对应的四个对象的原型，通过这个功能，我们可以快速的增加更多的辅助方法，例如我们在 app/extend/context.js 中写入下列代码：
    ```
    // app/extend/context.js
    module.exports = {
      get isIOS() {
        const iosReg = /iphone|ipad|ipod/i;
        return iosReg.test(this.get('user-agent'));
      },
    };
    ```
    在controller里就可以通过context调用
    ```
    // app/controller/home.js
    exports.handler = function* (ctx) {
      ctx.body = this.isIOS
        ? 'Your operating system is iOS.'
        : 'Your operating system is not iOS.';
    };
    ```
    
* ###  插件
    一个插件可以包含
    extend：扩展基础对象的上下文，提供各种工具类、属性。
    middleware：增加一个或多个中间件，提供请求的前置、后置处理逻辑。
    config：配置各个环境下插件自身的默认配置项。
    一个独立领域下的插件实现，可以在代码维护性非常高的情况下实现非常完善的功能，而插件也支持配置各个环境下的默认（最佳）配置，让我们使用插件的时候几乎可以不需要修改配置项。

* egg-mock启动和正常启动有差异，一切以egg-mock没问题为准