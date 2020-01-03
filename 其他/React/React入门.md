安装node.js

通过npm使用React

使用淘宝的cnpm

```
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
$ npm config set registry https://registry.npm.taobao.org
```

安装 JS 包管理工具yarn
```
$ npm -g i yarn
$ npx create-react-app chat-room-client --typescript
$ yarn start
```

阅读
```
React 文档中文翻译 https://doc.react-china.org/docs/hello-world.html
高级指南 https://doc.react-china.org/docs/jsx-in-depth.html
API 参考 https://doc.react-china.org/docs/react-api.html
```

```
https://ant.design
https://codesandbox.io
```

定义组件

```
function NameCard() {
  return(
    <div>
      WuPeiXuan
    </div>
  );
}
```

嵌入组件
```
const App: React.FC = () => {
  return (
    <div className="App">
      <header className="App-header">
        <NameCard/>
      </header>
    </div>
  );
}
```

定义function，并显示

```
function NameCard(props: NameCardProps) {
    return (
        <div id={"MyNameCard"}>
            姓名：{props.name}<br/>
            <img id="logo" className="App-logo" src={props.icon}/><br/>
            性别：{props.gender}<br/>
            职位：{props.jobTitle}<br/>
        </div>
    );
}
```

```
const App: React.FC = () => {
    return (
        <div className="App">
            <header className="App-header">
                <NameCard name={"武培轩"} gender="男" jobTitle={"Java"}
                          icon={"https://pic.cnblogs.com/avatar/1356806/20180320195124.png"}/>
                <img src={logo} className="App-logo" alt="logo"/>
            </header>

        </div>
    );
}
```
表单传递
```
      const arr = [{name: "武培轩", gender: "男", jobTitle: "Java"}, {name: "王鹏鹏", gender: "男", jobTitle: "Java"}];
      const result = arr.map(i => {
          return <NameCard name={i.name} gender={i.gender} jobTitle={i.jobTitle}
                           icon={"https://pic.cnblogs.com/avatar/1356806/20180320195124.png"}/>;
      });
```


提交表单，并把表单的内容显示
```
import React, {useState} from 'react';
import logo from './logo.svg';
import './App.css';
import 'antd/dist/antd.css';

/**
 * 组件接口
 */
interface NameCardProps {
    name: string;
    gender: string;
    jobTitle: string;
}

/**
 * 组件
 * @param props
 * @constructor
 */
function NameCard(props: NameCardProps) {
    return (
        <div id={"MyNameCard"}>
            姓名：{props.name}<br/>
            性别：{props.gender}<br/>
            职业：{props.jobTitle}<br/>
        </div>
    );
}

const App: React.FC = () => {
    const [name, setName] = useState('');
    const [cards, setCards] = useState<NameCardProps[]>([]);
    const result = cards.map(card => {
        return <NameCard name={card.name} gender={card.gender} jobTitle={card.jobTitle}
        />;
    })

    return (
        <div className="App">
            <header className="App-header">
                <img src={logo} className="App-logo" alt="logo"/>
                <div className={"form"}>
                    <input id={"myInput"} type="text" name="name" value={name} onChange={e => {
                        // console.log(e.target.value);
                        setName(e.target.value);
                    }}/>
                    <button onClick={e => {
                        console.log("被点击" + name);
                        const card: NameCardProps = {
                            name: name,
                            gender: "男",
                            jobTitle: "Java"
                        };
                        // cards.push(card);
                        setCards(cards => {
                            return [...cards, card];
                        });
                        setName('');
                    }}>提交
                    </button>
                </div>

                {result}
            </header>
        </div>
    );
}

export default App;


```