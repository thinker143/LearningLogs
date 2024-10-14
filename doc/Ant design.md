# ant design

开发工具：VSCode

使用create-react-app创建了一个TypeScript项目，并且引入了antd

##### 1、安装和初始化

命令：npx create-react-app antd-demo --template typescript

##### 2、进入项目并启动

命令：cd antd-demo

​           npm run start

##### 3、安装并引入antd

命令：npm install antd --save



##### 练习：

引入antd的按钮组件

```
命令：import React from 'react';
import { Button } from 'antd';

const App: React.FC = () => (
  <div className="App">
    <Button type="primary">Button</Button>
  </div>
);

export default App;
```
