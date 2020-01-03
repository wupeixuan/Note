```
import React, {useState} from 'react';
import logo from './logo.svg';
import './App.css';
import 'antd/dist/antd.css';
import {Radio} from 'antd';
import {Select} from 'antd';

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
    //定义
    const {Option} = Select;
    const [name, setName] = useState('');
    const [cards, setCards] = useState<NameCardProps[]>([]);
    const [gender, setGender] = useState('');
    const [jobTitle, setJobTitle] = useState('');

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
                        //赋值
                        setName(e.target.value);
                    }}/>
                    <br/>
                    <hr/>
                    {/*单选框*/}
                    <Radio.Group onChange={e => {
                        console.log('radio checked', e.target.value);
                        setGender(e.target.value);
                    }}>
                        <Radio value={"男"}>男</Radio>
                        <Radio value={"女"}>女</Radio>
                    </Radio.Group>
                    <br/>
                    <hr/>
                    {/*下拉框*/}
                    <Select value={jobTitle} defaultValue="Java工程师" style={{width: 120}} onChange={e => {
                        //单项绑定
                        setJobTitle(e);
                    }}>
                        <Option value="Java工程师">Java工程师</Option>
                        <Option value="前端工程师">前端工程师</Option>
                    </Select>
                    <br/>
                    <hr/>
                    <button onClick={e => {
                        console.log("被点击" + name);
                        const card: NameCardProps = {
                            name: name,
                            gender: gender,
                            jobTitle: jobTitle
                        };
                        // cards.push(card);
                        setCards(cards => {
                            //防止异步
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