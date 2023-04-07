# react

### 属性方法


#### 属性

a. props [调用模板传数据给代码]只读对象属性[可传对象] 特殊props.children 如果有子元素[数组,当前渲染] 
b. [代码访问内部模板组件] React.createRef(); 
c. state [模板数据变更]单向绑定 初始化:getInitialState 访问:state 修改:setState({}) 会调用render[dom diff]


#### 常用方法

a. componentDidMount() 组件被装配后立即调用[类似dom ready,一般这里初始化操作] 
b. componentWillUnmount() 组件被卸载后立即调用

```
class HelloMessage extends React.Component {
    a=1;//es6 属性
    #a=1;//es6  私有属性
    constructor(props={
        liked:false
      }){
    //构造 最先执行
        super(props);
        this.state={
            liked:props.liked
        };
       //这里不能用setState
       //构造[一次]
      }
      componentWillMount(){
        console.log(2);
        //渲染前[一次]
      }
      componentDidMount(){
        console.log(3);
        //渲染后[一次]
      }
      componentWillReceiveProps(){
        //props 变化会调用这里
        //state 变化不会调用
        console.log(4);
      }
      //shouldComponentUpdate(){
        //是否执行更新,优化性能可尝试重写
        //console.log(5);
        //return true;
      //}
      componentWillUpdate(){
        //更新前执行
        console.log(6);
      }
      componentDidUpdate(){
        //更新完执行
        console.log(7);
      }
      componentWillUnmount(){
        //组件卸载调用
        console.log(9);
      }
    handleClick(ref) {
        ref.focus();
        this.setState({liked: !this.state.liked});
        //this.props.obj is object
    }
    render() {
        //渲染其次执行
        //支持数组直接输出
        //var t=[<div/>,<div/>];
        //var b=<div>{t}<div/>;
        //遍历子元素
         //{
              //  React.Children.map(this.props.children, function (child) {
               //   return <li>{child}</li>;
              //  })
            //  }
            // this.props.children //直接输出子元素
        var text = this.state.liked ? 'like' : 'haven\'t liked';
        let input=React.createRef();
        return <div>
            <span onClick={this.handleClick.bind(this,input)}> Hello{this.props.name}</span>
            <input type="text" ref={input} />
            <p>
                You {text} this
             </p>
        </div>;
    }
}
//组件属性 props
HelloMessage.defaultProps = {
  name: 'demo'//默认属性
};
import PropTypes from 'prop-types';
LoginPage.propTypes = {
    user:PropTypes.object,//声明
};
var name='name';
var obj={o:'o'};
ReactDOM.render(
    <HelloMessage name={name} obj={obj}></HelloMessage>,
    document.getElementById('example')
);
```

> 备注

a. input value 为受控组件 设置后不可修改

```
 <input type="text" value='' />
```

b. fetch 取代AJAX的网络访问

```
 fetch(this.props.source).then(response=>(console.log(response),response.json())).then(data=>console.log(data));
 //response 是 Response 对象 常用方法 json text 属性 headers
```

c. => 函数

```
 let a = text=>text;//最后一个变量为返回值
 let b = ()=>'ddd';//无参数不可省略()
 let c = (a,b)=>console.log(a,b);//多参数不可省略()
```