#### 准备
- 了解gulp自动化构建工具及api
   - 基于流的构建方式，自定义性强，需要开发者自己实现各种功能，相对于grunt基于内存的构建方式读取快
   - 常用api：src(), dest(), series(), parallel(), watch()
#### 构建过程
 - 新建gulpfile.js作为入口文件
 - 样式编译
   - 将sass，less，stylus等css预处理文件编译成css，并进行兼容性处理
   ```js
   const sass = require('gulp-sass')
   const style = done => {
    return src('src/assets/styles/*.scss', { base: 'src' })
      .pipe(sass({ outputStyle: 'expanded' }))
      .pipe(dest('temp'))
   }
   ```
 - 脚本编译
   - 兼容性处理，如把es6，.ts文件转换为js，压缩
 - 页面模板编译
   - 使用模板引擎转换插件，转换页面模板内容
 - 图片和字体文件压缩
 - 开发服务器
   - 启动自动打开浏览器窗口
   - 使用到browser-sync包
     - notify(是否出现提示)
     - port(打开端口号)
     ```js
      bs.init({
        notify: false,
        port: 2000,
        // open: false,
        files: 'dist/**',
        server: {
          baseDir: ['temp', 'dist', 'src', 'public'],
          routes: {
            '/node_modules': 'node_modules'
          }
        }
      })
     ```
 - 文件压缩
   - 压缩转换后的css, html ,js, gulp-htmlmin(默认删除空白符) gulp-uglify gulp-clean-css，去掉空格，换行符等,压缩html用到htmlmin可以配置较多的参数
   ```js
    const useref = () => {
      return src('temp/**/*.html', { base: 'temp' })
        .pipe(plugins.useref({ searchPath: ['temp', '.'] }))
        .pipe(plugins.if(/\.js$/, plugins.uglify()))
        .pipe(plugins.if(/\.css$/, plugins.cleanCss()))
        .pipe(plugins.if(/\.html$/, plugins.htmlmin({
          collapsewhitespace: true,
          minifyCSS: true,
          minifyJS: true
        }))) // 
        .pipe(dest('dist'))
    }
   ```
 - 热更新，监听到src下代码改变自动编译输出到dist，并更新到浏览器
   ```js
    const script = done => {
    return src('src/assets/scripts/*.js', { base: 'src' })
      .pipe(babel({ presets: ['@babel/preset-env'] })).pipe(dest('temp'))
    }
    const style = done => {
      return src('src/assets/styles/*.scss', { base: 'src' })
        .pipe(sass({ outputStyle: 'expanded' }))
        .pipe(dest('temp'))
    }
    const page = () => {
      return src('src/**/*.html', { base: 'src' })
        .pipe(swig({ data: config.data }))
        .pipe(dest('temp'))
    }
    watch('src/assets/styles/*.scss', style),
    watch('src/assets/scripts/*.js', script),
    watch('src/*.html', page),
   ```
 - 提取出gulpfile.js
   - 把公共模块提取出来，封装成npm包的形式，能够在多个项目中使用维护
   - npm publish(发布)
   - npm install 包名 （下载）
