# RNIconfont
一个演示如何在RN中使用我们自定义的ttf文件来展示icon的demo，并提供生成iconfont映射文件的脚本

# ReactNative中如何使用自定义的iconfont
&nbsp;&nbsp;&nbsp;&nbsp;在App包的构成中图片资源是比较占大小的，所以我们可以利用Iconfont来替代原来的png或者jpg单色图标，iconfont的优点是占Size小、支持任意大小不失真、支持任意颜色设置、平台化管理icon等等。

>当App项目的大小达到一定规模时，App包的Size也会随之增大，这个时候我们一般会对App包的Size做分析优化来达到减少包大小的目的，利用iconfont来替代项目中原有的png单色图标是你的一种选择。

&nbsp;&nbsp;&nbsp;&nbsp;这里要分享的是在React Native开发中如何使用自定义的Iconfont图标，并提供一键自动生成对应映射文件的脚本。


**本文主要分为三部分：**

 - react-native-vector-icons库的集成与使用
 - 如何使用任意自定义的iconfont
 - 编写脚本来快捷生成iconfont映射文件

### 1.react-native-vector-icons库的集成与使用
react-native-vector-icons是ReactNative开发中十分好用的一个用来展示iconfont图标的库。

**集成只要两步：**
1.引入依赖
```json
Run: npm install --save react-native-vector-icons
```
2.链接原生库
```json
Run: react-native link react-native-vector-icons
```

如果第二步链接失败或者运行错误，可以自己手动链接，具体可以参考[react-native-vector-icons](https://www.npmjs.com/package/react-native-vector-icons)说的比较清楚了。

集成完毕后，可以看到这个库默认引入了几个ttf文件（android项目在assets/fonts下）：
![默认引入的一些ttf文件](https://user-gold-cdn.xitu.io/2018/4/7/1629d708db04e199?w=538&h=534&f=png&s=57141)

也就是说这些ttf文件中所有icon都可以直接使用，下面就说一下如何使用。

使用非常简单，如下：
```javascript
import Icon from "react-native-vector-icons/Ionicons";

<Icon name='md-pricetag' size={16} color='#cccccc'></Icon>
```

### 2.如何使用任意自定义的iconfont

&nbsp;&nbsp;&nbsp;&nbsp;上面介绍了react-native-vector-icons库的使用，但是目前为止你能使用的iconfont只有上面说的默认引入的那些ttf文件中的icon,因为上面所使用的< Icon>< /Icon>只支持默认引入的那些icon。

&nbsp;&nbsp;&nbsp;&nbsp;这样显然不能满足我们的要求，我们想要使用自己的iconfont,那么我们该如何做呢，这里以从阿里iconfont平台上选择自己想要的icon为例做介绍。

 **一、从阿里iconfont平台上挑选自己想要的icon，打包下载到本地并解压，如下：**
![解压后](https://user-gold-cdn.xitu.io/2018/4/7/1629d7140b55ac4d?w=338&h=388&f=png&s=51946)

**二、将iconfont.ttf文件copy到android项目的assets/fonts目录下**

**三、自定义图标库**

CXIcon.js
```javascript
import createIconSet from 'react-native-vector-icons/lib/create-icon-set';
import glyphMap from './iconfont.json';

const iconSet = createIconSet(glyphMap, 'CXIcon', 'iconfont.ttf');

export default iconSet;

export const Button = iconSet.Button;
export const TabBarItem = iconSet.TabBarItem;
export const TabBarItemIOS = iconSet.TabBarItemIOS;
export const ToolbarAndroid = iconSet.ToolbarAndroid;
export const getImageSource = iconSet.getImageSource;
```
看到import glyphMap from './iconfont.json'了吗，所以这里我们还需要一个iconfont.json，也就是映射map。

iconfont.json:
```json
{
  "biscuit": 58983,
  "pizza": 59024,
  "dangao": 59080
}
```
这是我们在阿里iconfont上下载的三个icon对应的Unicode码。

**ok，到这里我们就可以使用<CXIcon></CXIcon>来展示我们自定义下载的几个icon了，使用如下：**
```javascript
import React, {Component} from 'react';
import {
    Text,
    View
} from 'react-native';
import CXIcon from "./components/cxicon/CXIcon";

type Props = {};
export default class App extends Component<Props> {
    render() {
        return (
            <View style={styles.container}>
                <Text>展示来自自定义的ttf文件的icon</Text>
                <CXIcon name='biscuit' size={50} color='#226688'></CXIcon>
            </View>
        );
    }
}
```

### 3.编写脚本自动生成上面的iconfont.json映射文件
&nbsp;&nbsp;&nbsp;&nbsp;看完第二步其实你就已经可以自由的使用自己选择的icon图标了，但是你会发现一个问题，上面我们需要一个iconfont.json映射文件，这个映射文件是我们手写的，如果只有3个图标那我们可以手写，那如果是300个图标，显然不行。
&nbsp;&nbsp;&nbsp;&nbsp;回到上面，我们从阿里iconfont平台下载到的zip压缩包解压缩后里面有一个iconfont.svg文件，我们就根据这个文件来解析生成我们想要的iconfont.json映射文件，撸起袖子，写个shell脚本。

iconfont_mapper.sh:
```shell
#!/bin/sh

if [ $# != 1 ] ; then

	echo "usage: $0 iconfont.svg(your svg file name)  "
	exit
fi

#OutputFile path,you can customize your path
OutputFileName=`echo iconfont.json`

mapper=` awk '{ if($0 ~ /glyph-name/) print $0;  if($0 ~ /unicode/)  print $0"|split|" }'  $1| tr '[:upper:]' '[:lower:]'| awk '{print $0}'  RS='\='| tr "\n\"&#;" " "| awk  '{ if ($1!="split"&&$1!=""){ printf ("\""$3"\":"); printf ($5","); print "\r " }}' RS="|split|" | sed "s/-/_/g"`

rm $OutputFileName
echo "{" >> $OutputFileName
echo $mapper >> $OutputFileName
echo "}" >> $OutputFileName

#usage:
# ./iconfont_mapper.sh svg_file_path
```

使用：
命令行执行：  ./iconfont_mapper.sh  iconfont.svg          即可。
>注意：如果你的iconfont_mapper.sh脚本和iconfont.svg文件没有放在同一个文件目录下，则iconfont.svg需要拼全路径。

&nbsp;&nbsp;&nbsp;&nbsp;执行完这个脚本你就会发现在脚本所在的目录下自动生成了我们需要的iconfont.json映射文件。将它放到项目中即可。
