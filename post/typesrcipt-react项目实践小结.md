## 安装
首先全局安装typescript模块。
```
npm install -g typescript
```
安装react和reactDom依赖
```
npm install --save react react-dom @types/react @types/react-dom
```
安装ts-loader
```
npm install --save-dev typescript awesome-typescript-loader source-map-loader
```
webpack配置
```
const webpack = require('webpack');

module.exports = {
  entry: {
    bundle: './src/index.tsx',
    vendor: ['react', 'react-dom', 'axios']
  },
  output: {
    filename: 'bundle.js',
    path: `${__dirname}/dist`,
  },

  devtool: 'source-map',

  resolve: {
    extensions: ['.css', '.ts', '.tsx', '.js', '.json'],
  },

  module: {
    rules: [
      { test: /\.tsx?$/, use: 'awesome-typescript-loader' },
      { enforce: 'pre', test: /\.js$/, use: 'source-map-loader' },
      { test: /\.css$/, use: ['style-loader', 'css-loader'] },
    ],
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({ name: 'vendor', filename: 'vendor.bundle.js' }),
  ],
};
```

配置tsconfig.json
```
{
    "compilerOptions": {
        "outDir": "./dist/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "commonjs",
        "target": "es5",
        "jsx": "react"
    },
    "include": [
        "./src/**/*"
    ]
}
```
这里以cnode的列表作为示例。
目录结构如下所示：
![image.png](http://upload-images.jianshu.io/upload_images/2419083-50efb942409d5317.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
import * as React from 'react';
import { get } from '../../utils/request';
import { Topic } from '../../interface/index';
import TopicItem from '../../components/Topic';

export interface IProps { compiler: string; framework: string}

interface IState { data: Array<Topic> }

class App extends React.Component<IProps, IState> {
  constructor(props: IProps) {
    super(props);
    this.state = {
      data: []
    };
  }
  componentDidMount() {
    get('/topics?limit=15').then((res) => this.setState({ data: res.data.data })).catch(err => alert(err));
  }
  render() {
    const { data } =  this.state;
    console.log(data);
    return (
      <div>
        {data.map((item: Topic) => (
          <TopicItem key={item.id} item={item} />
        ))}
      </div>
    );
  }
}

export default App;
```
这样一个最简单的ts-react项目就搭建成功了。

期间遇到一个比较奇怪的设定
```
interface TabMap {
  ask: string;
  share: string;
  job: string;
  [key: string]: string;
}

const tabMap: TabMap = { ask: '问答', share: '分享', job: '职场' };
```
如果要使用tabMap[tab]就必须在interface里定义[key:string]属性。
