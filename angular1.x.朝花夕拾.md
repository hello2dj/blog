### 由于项目原因又捡起了angular1.x
### angular的ocLazyLoad一个问题
最近需要使oclazyload加载的文件带上hashstring,防止某些文件被缓存，但是读了他的文档并没有发现相关的参数，然后搁置了，直到昨天遇到了一个bug，oclazyload有个配置选项serie，如下
```
ngular.module('app')
  .config(['$ocLazyLoadProvider', function ($ocLazyLoadProvider) {
    $ocLazyLoadProvider.config({
      // debug: true,
      events: true,
      modules: [{
        name: 'metrojs',
        files: [
          'assets/plugins/jquery-metrojs/MetroJs.min.js',
          'assets/plugins/jquery-metrojs/MetroJs.css'
        ],
        serie: true
      }
    }
  })
```

我们的项目在定义了多个modules以后，发下只要不加serie为true这个选项，就会出现，文件加载了，但是却没有起作用，于是下了狠心去看了看他的源码，发现其实并没有啥很奇特的地方