FinEx前端React重构方案
=====================
# 前言
> 注： 文章中所指的dva都是指dva 2.1.0版本

整体的重构方案是基于[dva](https://github.com/dvajs/dva)的框架，我们在[dva](https://github.com/dvajs/dva)框架的基础上进行定制化完善。
首先我们使用dva-cli创建新应用，具体使用请参考[这里](https://ant.design/docs/react/practical-projects-cn)，然后我们通过定制化的手段进行如下完善。
> 注：由于通过dva-cli创建的应用采用的react、eslint等库都是较旧的版本，如果需要最新的版本请使用如下命令进行升级。更新到最新后，请注意依赖是否兼容最新版本，即能正常运行。
```
npm install react@latest react-dom@latest dva@latest babel-runtime@latest --save
npm install babel-eslint@latest babel-plugin-dva-hmr@latest babel-plugin-transform-runtime@latest eslint@latest eslint-config-airbnb@latest eslint-plugin-import@latest eslint-plugin-jsx-a11y@latest eslint-plugin-react@latest expect@latest husky@latest redbox-react@latest roadhog@latest --save-dev
```

## 1. 减少重复开发
* 采用组件化方案，提供基础组件和业务组件。
    * 基础组件基于Antd，echarts
    * 业务组件提供：图表组件、登录、找回密码、加密、校验。实现逻辑和表现分离，以支持不同UE的效果变更。
* mobile和platform共用一套业务逻辑

## 2. 提高开发效率
**使用CSS Modules处理样式，减少样式名称长度和可读性**
dva提供了css modules处理，默认采用`[local]___[hash:base64:5]`命名格式，如果需要修改命名格式则需要打开`node_modules\roadhog\lib\utils\getCSSLoaders.js`文件，把`localIdentName`属性对应的值改为`[name]__[local]___[hash:base64:5]`，或者其它你期望的命名格式。

如果需要把整个文件设置为全局样式，打开.roadhogrc文件添加cssModulesExclude配置项就可以了，如下所示：
```
"cssModulesExclude": [
  './src/a.css',
  './src/b.less',
]
```

如果只需要局部样式使用全局样式，则可以使用`:global`关键词，如：
```css
:global .class {
}
```

如果项目关联使用了antd组件库，则推荐使用less来管理样式，并在项目中使用同一套变量。

推荐把重复的样式通过mixin抽象，同时减少嵌套，因为css modules生的样式名必定是全局唯一的，因此不需要嵌套。

## 3. 与后端并行开发
采用mock模拟接口，后端只需提供接口协议，等后端接口开发完成后再进行联调。
目前dva框架已经提供了mock的支持，我们在此基础上引入mock.js库直接使用就好。

**实现方法：**
1. 安装mock.js `npm install --save-dev mockjs`
2. 在`.roadhogrc.mock.js`文件中，编写如下代码：
```js
// 加载并执行mock目录下的所有文件，并合并到mock对象中，最后把合并后的mock进行export
// 这么做的原因是为了能够按模块Mock数据，如果是大型系统，可以使用多级目录划分模块
const mock = {}
require('fs').readdirSync(require('path').join(__dirname + '/mock')).forEach(function (file) {
  Object.assign(mock, require('./mock/' + file))
})
module.exports = mock
```

3. 在mock目录中创建需要Mock数据的模块，引入mock.js进行数据模拟，详见 [roadhog的mock使用说明](https://github.com/sorrycc/roadhog#mock) 和 [Mock.js文档](https://github.com/nuysoft/Mock/wiki)，示例如下：
```js
// 简单的mock数据模拟
module.exports = {
  // 支持值为 Object 和 Array
  'GET /api/users': { users: [1,2] },

  // GET POST 可省略
  '/api/users/1': { id: 1 },

  // 支持自定义函数，API 参考 express@4
  'POST /api/users/create': (req, res) => { res.end('OK'); },

  // Forward 到另一个服务器
  'GET /assets/*': 'https://assets.online/',

  // Forward 到另一个服务器，并指定子路径
  // 请求 /someDir/0.0.50/index.css 会被代理到 https://g.alicdn.com/tb-page/taobao-home, 实际返回 https://g.alicdn.com/tb-page/taobao-home/0.0.50/index.css
  'GET /someDir/(.*)': 'https://g.alicdn.com/tb-page/taobao-home',
}
```
```js
// 采用mock.js模拟
'use strict'

const mockjs = require('mockjs') // 导入mock.js的模块
const Random = mockjs.Random // 导入mock.js的随机数

// 数据持久化，保存在global的全局变量中
let holdingPlans = {}

if (!global.holdingPlans) {
  // 生成mock数据
  const data = mockjs.mock({
    content: {
      'holdingPlans|8': [{
        'planId|+1': 4008000,
        'planName': `${Random.name()} - ${Random.cname()}`,
        amount: Random.float(100000, 1000000000, 0, 3),
        principal: Random.natural(100000, 1000000000),
        profit: Random.float(100000, 1000000000, 0, 3),
        totalDeclared: Random.float(100000, 1000000000, 0, 3)
      }],
      totalAmount: Random.float(1000000, 1000000000, 0, 3),
      totalRecords: 8
    }
  })
  holdingPlans = data
  global.holdingPlans = holdingPlans
} else {
  holdingPlans = global.holdingPlans
}

module.exports = {
  'GET /api/user/list': { users: [1, 2, 3, 4, 5, 6, 7] },
  // post请求  /api/users/ 是拦截的地址   方法内部接受 request response对象
  'GET /api/v2/users/assets/holding-plans': (req, res) => {
    res.json({ // 将请求json格式返回
      ...holdingPlans,
      errors: [],
      result: 'success'
    })
  }
}

```

4. 后端接口开发完成后，修改.roadhogrc文件的proxy参数，详见 [webpack-dev-server#proxy](https://webpack.github.io/docs/webpack-dev-server.html#proxy)，示例如下：
```js
"proxy": {
  "/api": {
    "target": "http://jsonplaceholder.typicode.com/",
    "changeOrigin": true,
    "pathRewrite": { "^/api" : "" }
  }
}
```
然后访问 /api/users 就能访问到 http://jsonplaceholder.typicode.com/users 的数据。

> 另一种方案是使用代理指向到json-server，具体请参考 https://github.com/typicode/json-server
> 这里采用mock.js是因为它有更好的定制能力且功能强大，但json-server更为简单易用，可以根据实际情况选择。

**参考资料：**
[roadhog的使用说明](https://github.com/sorrycc/roadhog#mock)
[Mock.js的Wiki](https://github.com/nuysoft/Mock/wiki)
[只需要三步，即可使用mock进行模拟数据](http://doc.okbase.net/tjc1996/archive/262169.html)

## 4. 统一的风格
* 使用静态代码检查保障
dva本身提供了eslint的代码检查，采用了[eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)规范进行静态代码检查，但该规范为javascript通用规范，对于React来说，我们选择更为严谨和标准的[eslint-config-standard-react](https://github.com/standard/eslint-config-standard-react)规范。
因此我们进行了以下的改造：
1. 移除无用的eslint包
`npm uninstall --save-dev eslint-config-airbnb eslint-plugin-jsx-a11y`
2. 安装eslint-config-standard-react及依赖包，也可以参考[这里](https://github.com/standard/eslint-config-standard-react)
`npm install --save-dev eslint-config-standard eslint-config-standard-react eslint-plugin-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node eslint-plugin-react`
使用Eslint进行代码静态检查，采用
3. 替换应用目录下的.eslintrc文件内容
```javascript
{
  "parser": "babel-eslint",
  "parserOptions": {
    "ecmaVersion": 8,
    "ecmaFeatures": {
      "experimentalObjectRestSpread": true,
      "jsx": true
    },
    "sourceType": "module"
  },
  "plugins": [
    "react"
  ],
  "extends": [
    "standard",
    "standard-react"
  ],
  "env": {
    "browser": true,
    "amd": true,
    "es6": true,
    "node": true,
    "mocha": true
  },
  "rules": {
    "space-before-function-paren": [2, { "anonymous": "always", "named": "never" }]
  }
}
```
进行了以上三步改造后，就可以通过npm run lint进行代码检查了。
> 注：
> 1. 安装eslint-config-standard-react后，会提示eslint太旧（如果之前没有升级），需要按提示升级到最新版本。
> 2. 由于dva-cli自动生成的代码不符合eslint-config-standard-react的规范，因此会报检查错误，这是正常的，实际开发中需要更改过来。
> 3. 如果一些静态检查的错误不知道如何处理，可以使用错误信息通过google搜索即可找到答案。

**Vistual Studio编辑器配置Eslint**
1. 按快捷键`Shift + Alt + P`打开控制台，输入`Install Extensions`打开扩展安装
2. 在应用商店中搜索`eslint`扩展，点击“安装”，安装成功后，点击“重新加载”即可完成VsCode编辑器的Eslint检查
3. 因为eslint-config-standard-react规范默认采用2个空格的Tab，而VsCode默认的Tab是4个空格，因此通过如下方法修改配置：按快捷键`Shift + Alt + P`打开控制台，输入`Indent Using Spaces`，选择`2`即可。
4. 点击文件->首选项->设置，在用户设置中添加下面代码，即可配置新建js文件时，默认采用2个空格的tab

```
"[javascript]": {
  "editor.insertSpaces": true,
  "editor.tabSize": 2,
}
```

## 5. 极致的性能和体验
1. 使用并发异步请求
在使用redux里，我们会采用Generator语法来进行异步调用。但有部份同学会采用如下写法：
```js
* query({ payload }, { call, put }) {
  const aData = yield call(queryA, payload)
  const bData = yield call(queryB, payload)
  const cData = yield call(queryC, payload)
  if (aData.success || bData.success || cData.success) {
    yield put({
      type: 'save',
      payload: {
        aData: aData.content,
        bData: bData.content,
        cData: cData.content
      }
    })
  }
}
```
上面代码的queryA、queryB和queryC请求是串行请求，即queryA执行完成并返回结果后向服务器请求queryB，等待queryB执行完成并返回结果后向服务器请求queryC，以此类推。假设queryA请求的响应是200ms，queryB请求的响应是300ms，queryC请求的响应是200ms，那么，理论上总响应时间是700ms。只有当下一个请求依赖上一个请求的响应结果时，才能使用上面的串行请求方式。但是该事例各请求之间不相互依赖返回结果，因此应当采用并发的请求方式，以缩短整体的响应时间为300ms。代码如下：
```js
// 并发请求
const aPromise = queryA(payload)
const bPromise = queryB(payload)
const cPromise = queryC(payload)
// 等待结果
const aData = yield aPromise
const bData = yield bPromise
const aData = yield cPromise
// 处理结果
if (aData.success || bData.success || aData.success) {
    yield put({
      type: 'save',
      payload: {
        aData: aData.content,
        bData: bData.content,
        cData: cData.content
      }
    })
}
```
上面的代码会同时发送queryA、queryB和queryC请求，等待所有请求都返回结果后，再进行处理。但是该实现方式的前提是假定所有的请求都能正确执行，如果发生错误需要写链式的catch或者使用try...catch来捕获错误。为了解决此问题，可以使用下面的写法：
```js
// 并发请求
const [aData, bData, cData] = yield Promise.all([queryA(payload), queryB(payload), queryC(payload)])
// 处理结果
if (aData.success || bData.success || aData.success) {
    yield put({
      type: 'save',
      payload: {
        aData: aData.content,
        bData: bData.content,
        cData: cData.content
      }
    })
}
```
上面的代码通过Promise.all对所有要进行并发请求的接口进行包装，代码更为简洁直观，推荐使用。

提供按需加载，懒加载方案
统一的错误处理
统一的加载提示
版本号

## 6. 优秀的代码组织
按模块划分目录结构

**参考资料**
https://github.com/dvajs/dva/issues/21

## 7. 高质量的代码
使用单元测试保障
引入监控统计（后期）
    
## 8. 门户
新手引导
知识沉淀

## 9. 债务
1. 目前鉴权是通过调用最常用的保护接口（如：profile接口），然后根据返回值结果判断是否为登录状态。能否不调用接口就能判断登录状态？过期时间采用监听所有请求的响应结果判断是否登录过期。
2. LoanDetail 获取完LoanInfo的数据后，再弹出对话框。目前采用回调方式传递方法到model，由model进行弹窗。因此需要封装对话框展现装饰器，如LoanDetailModal
> 可以按Antd Modal的示例使用，直接把对应的触发按钮封装到业务Modal组件，在组件内控制展现，这样外部调用时，只需要引入组件并通过props传递参数即可。
3. <del>目前Modal多请求为同步发送，能否改为并发</del>
4. React-Intl国际化，在切换语言时不刷新页面
> 把Root根元素封装成组件并对外公开，然后在需要切换语言的功能（如：header）里引入Root组件进行语言切换，如下代码所示：
```js
// 这是index.js文件，即应用启动入口
// 把ReactDOM.render放入Root方法里，并把Root方法公开
function Root({ lang = null }) {
  ReactDOM.render(
    <IntlProvider lang={lang} defaultLang='en-US'>
      <App />
    </IntlProvider>,
    document.getElementById('root')
  )
}

Root({})

export default Root

// 这是Header.js文件
// 引入根元素组件，并根据语言重新渲染组件
import Root from '../../index'

const changeLocale = function (locale) {
  Cookies.set('locale', locale)
  // 重新渲染根组件，以达到语言切换的目标
  Root({ lang: locale })
}

```
5. 如何把http1.1升级到http2.0
6. 路由跳转后，不会自动滚动到顶部
> 解决该问题分二步走，
> 第一步: 在Route里封装一层父元素，用于判断是否返回顶部。
> 因为所有的路由切换都要经过Route进行渲染，所以在Route的入口位置添加返回顶部的特性，代码如下：
```js
// 这是PrivateRoute.js文件
function PrivateRoute({ component: Component, ...rest }) {
  return (
    <Route {...rest} render={props => {
      // state的数据来自于<Link>组件的to属性，看第二步
      const { location: { state = {} } } = rest
      if (state.returnTop !== false) {
        setTimeout(() => window.scrollTo(0, 0), 10)
      }
      return <Component {...props} />
    }} />
  )
}

// 在route.js文件里使用PrivateRoute
<Switch>
  <PrivateRoute exact path={path} component={component} />
</Switch>
```

> 第二步：在不需要返回顶部的Link链接里，使用如下代码
```js
// Link组件的to属性使用对象，并在对象的state里设置returnTop为false
<Link to={{ pathname: pathname, state: { returnTop: false } }}>Link Name</Link>
```
参考链接：https://reacttraining.com/react-router/web/api/Link

> 注意：
> 1. 同事建议：可能问题没有真正解决，setTimeout只是做了个异步延迟，如果这个时候dom没有转成真实的dom是问题的，所以应该在dom的onload事件上处理，react应该在router change的时候处理， 现在没问题应该是react处理虚拟dom的速度很快，这个问题如果以后复现你可以从这里排查一下。但从目前来看，即使去掉setTimeout并且页面内容很长的情况下也一样能正常返回顶部。这个想法先记录下来，如果有遇到问题，再回过来排查。
> 2. 本特性只支持返回顶部，不支持锚点，锚点还需另外处理。