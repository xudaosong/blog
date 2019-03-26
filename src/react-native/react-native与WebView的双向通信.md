基于RN的WebView与H5双向通信
=========
在RN中，WebView有onMessage事件和postMessage方法。
* onMessage用于接收H5的window.postMessage发送的消息
* postMessage用于向H5发送消息，H5通过监听message事件接收消息

## RN代码
```js
import React, { Component } from 'react';
import {
  WebView
} from 'react-native';

export default class App extends Component {
  constructor(props){
    super(props)
    this.handleMessage = this.handleMessage.bind(this)
  }
  handleMessage(e) {
    // 获取H5的消息
    let event = e.nativeEvent;
    let data = JSON.parse(event.data);
    // 向H5发送消息
    this.refs.webviewRef.postMessage(JSON.stringify({
      result: this.data.userId + ' success'
    }))
  }
  render() {
    return (
      <WebView
        ref={'webviewRef'}
        source={{ uri: 'http://10.8.25.41:8000/login' }}
        onMessage={this.handleMessage}
      />
    );
  }
}

```

## H5代码
```js
// 接收RN的消息
document.addEventListener('message',function(e){
    alert(e.data)
})
// 向RN发送消息
window.postMessage(JSON.stringify({
    type:'login',
    payload:{
        userId: '111',
        password: '222'
    }
}))
```

## 备注
> 如果生产环境中使用的话，还可以对调用形式进一步封装，比如H5中方法调用RN中的分享，并且要求知道分享结果是否成功，可以使用 webviewBridgecallRN('share',{title:'微信分享'，content:"分享内容"},callback)这种类似的方式，同时RN中可根据事件的名字进行分模块来实现

> 查阅了react-native的Webview源码变更记录 onMessage/postMessage是从 0.37 版才支持的。android与ios的支持版本同步。

## 参考
http://kunkun12.com/2017/03/14/react-native-webview-communicate/