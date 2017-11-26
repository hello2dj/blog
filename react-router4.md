`React Router question” is probably a “React question`

[理解rr4](https://css-tricks.com/react-router-4/)

[api](https://reacttraining.com/react-router/web/guides)

[关于querystring的issue](https://github.com/ReactTraining/react-router/issues/4410)

[block updating](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/guides/blocked-updates.md)


[react-router-redux@next]这个是兼容rr4的

[history]

![](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-03.svg)


### react router 4
  * ### Route组件的使用1, component的传递方式不同渲染也会不同
    * #### <Route component={A}>这种使用方法会把location的props传递给A,A会随着location的改变而响应
    * #### <Route ...><A /> </Route>,这种事不会把loaction等传递给A组件的，意味着，A不会随着location的变化而变化，Route只会控制是否渲染A, A的render却不会随着locaiton的变化而被调用，这时需要把A包裹在withRouter中就好了
    * ### [详见](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/modules/Route.js#L121)
