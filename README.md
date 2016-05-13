RNMaterialDribbble
---
基于文学编程理念设计的 React Native 入门教程, 循序渐进的提供 Material 风格 Dribbble 第三方客户端开发指引:)

(c) 2016 [borisding@me.com](https://hacker.design/) 一切权利保留.
本作品适用于 [署名-非商业性使用 3.0 中国大陆 (CC BY-NC 3.0 CN)](http://creativecommons.org/licenses/by-nc/3.0/cn/) 协议.

> 重要提示: 本教程目前暂未全部完成,仅做参考之用. 此提示信息将在教程完工后予以撤除.

## 前置知识
本教程作为使用 React Native 的实作指引, 限于篇幅恕不再对于 React 及 React Native 的基础知识再做赘述. 在基于本教程进行实作之前,建议通读 React 及 React Native 官方文档为宜.
* [React 官方文档](https://facebook.github.io/react/docs/getting-started.html)
* [React Native官方文档](https://facebook.github.io/react-native/docs/getting-started.html)

同时本教程亦默认读者具有实际的web开发经验, 而不在针对相关名词或方法论辅以解释说明.

## 如何使用
首先您需要并注册成为 [Dribble开发者](http://developer.dribbble.com/).同时,您需要配置 NodeJS iOS 及 Android的相关开发环境依赖, 其中我们特别建议您使用 [NVM](https://github.com/creationix/nvm) 来管理您的 NodeJS 环境. 而对于 iOS 及 Android 开发环境的配置, 建议您参考 React Native 官方提供的[相关文档](https://facebook.github.io/react-native/docs/getting-started.html).


本教程基于文学编程(Literate programming)理念设计, 您可以直接通过教程的 MarkDown 源文件生成对应的示例代码.

```bash
git clone https://github.com/borisding1994/RNMaterialDribbble
npm install -g literate-programming
cd RNMateriabDribbble
npm install
literate-programming README.md
```

##  入口文件
秉承于 "learn once write anywhere" 的理念, 在 React Native 中不同平台拥有不同的入口文件.形如:
* [index.ios.js](#入口文件 "save:")
* [index.android.js](#入口文件 "save:")

然而在真实项目的开发之中, 我们为了尽可能的复用相似代码会令不同的入口文件均指向到同一APP Component.

导入依赖的库
```js
import {AppRegistry} from 'react-native';
import Application from './src/application';
```
AppRegistry 是运行所有 React Native 应用程序的 JS 入口点。应用程序跟组件需要通过 `AppRegistry.registerComponent` 来注册它们自身，然后本地系统就可以加载应用程序的包，再然后当 `AppRegistry.runApplication` 准备就绪后就可以真正的运行该应用程序了。

AppRegistry 在 require 序列里是 required，确保在其他模块被需求之前 JS 执行环境已经被 required。

```js
AppRegistry.registerComponent('RNMaterialDribbble', () => Application);
```

# Material
我们将在React Native中使用由 [react-native-material-kit](https://github.com/xinthink/react-native-material-kit) 所提供的高质量UI组件.

很大一部分第三方 React Native 组件在安装过程中都需要修改项目中相关的 Native 代码. 在本仓库已经完成了相关 Native 代码的集成, 如您要在其他项目中使用 react-native-material-kit 请注意参阅相关文档.

# AppComponent
[src/application.js](#AppComponent "save:") 在某种意义上或可称之为真正的入口文件.
```js
import React from 'react';
import {View} from 'react-native';
import ScrollableTabView from 'react-native-scrollable-tab-view';
import Header from './components/Header';
import DribbbleList from './components/DribbbleList';

export default class Application extends React.Component {
                 render() {
                   return (
                   <View>
                        <Header />
                        <ScrollableTabView tabBarBackgroundColor="#ED679B" tabBarInactiveTextColor="#eee"
                         tabBarActiveTextColor="#fff" tabBarUnderlineColor="#E73177">
                            <DribbbleList tabLabel="animated"></DribbbleList>
                            <DribbbleList tabLabel="debuts"></DribbbleList>
                            <DribbbleList tabLabel="attachments"></DribbbleList>
                        </ScrollableTabView>
                    </View>
                 );
                 }
               }
```

# Header
[src/components/Header.js](#Header "save:") 是一个 React Native组件, 我们在此组件中定义了APP的头部样式.

```js
import React from 'react';
import {View, Text, Image, StyleSheet} from 'react-native';

export default class Header extends React.Component {
    render() {
        return (
        <View style={style.header}>
            <Text style={style.title}>DDribbble</Text>
        </View>
        )
    }
}
```
React Native基于Flex模型来设计布局样式:
```js
style = StyleSheet.create({
header: {
marginTop: 20,
flex: 1,
height: 40,
backgroundColor: '#ea4c89',
justifyContent: 'center',
alignItems: 'center'
},
title: {
fontSize: 18,
color: '#fff'
}
})
```

# DribbbleList
[src/components/DribbbleList.js](#DribbbleList "save:")

```js
import React from 'react';
import {View, Text, Image, StyleSheet, ListView} from 'react-native';
import {getTheme} from 'react-native-material-kit';
import API from '../helper/api';

const theme = getTheme();


export default class DribbbleList extends React.Component {
 constructor(props) {
        super(props);
        this.state = {
            dataSource: new ListView.DataSource({
                rowHasChanged: (row1, row2) => row1 !== row2,
            }),
            loaded: false
        };
    }

  componentDidMount() {
         this.fetchData();
     }

  fetchData() {
            API.getShotsByType('debuts', 1).then((responseData) => {
                                                    this.setState({
                                                      dataSource: this.state.dataSource.cloneWithRows(responseData),
                                                      loaded: true
                                                    });
                                                  }).done()
      }


    render() {
       if (!this.state.loaded) return this.renderLoadingView();
       return(
         <ListView
                dataSource={this.state.dataSource}
                renderRow={this.renderShot}
              />
       )
    }

    renderShot(shot) {
        return(
            <View style={theme.cardStyle}>
                 <Image source={{uri: shot.images.normal}} style={theme.cardImageStyle}/>
                  <Text style={theme.cardTitleStyle}>{shot.title}</Text>
                  <Text style={theme.cardContentStyle}>
                    {shot.description}
                  </Text>
            </View>
        )
    }

     renderLoadingView() {
            return (
                <View>
                    <Text>
                        正在加载...
                    </Text>
                </View>
            );
            }
}
```

# DribbleAPI
[src/helper/api.js](#DribbleAPI "save:")
参考: https://github.com/catalinmiron/react-native-dribbble-app/blob/master/app/helpers/api.js
```js
"use strict";

var API_URL = "https://api.dribbble.com/v1/",
```

请在此替换你的ACCESS_TOKEN
```js
    ACCESS_TOKEN = "7a22f910dcdff63bd3ebbe48d022f05e8268c67249709b5489d923f97bcf96ec";
```

```js
function fetchData(URL) {
  return fetch(URL, {
    headers: {
      "Authorization": "Bearer " + ACCESS_TOKEN
    }
  }).then((response) => response.json())
}

module.exports = {
  getShotsByType: function(type, pageNumber) {
    var URL = API_URL + "shots/?list=" + type;
    if (pageNumber) {
      URL += "&per_page=10&page=" + pageNumber;
    }

    return fetchData(URL);
  },
  getResources: function(url) {
    return fetchData(url);
  }
};
```