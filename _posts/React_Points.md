JSX输出带有html标记的内容，不希望自动转义(不推荐)，存在xss攻击可能

```javascript
<ul>
{
	this.state.list.map((item,index) => {
		return (
			<li 
    		key={index} 
			onClick={this.handleItemDelete.bind(this, index)}
			dangerouslySetInnerHTML={{__html:item}}	
			></li>
			)
		})
	}
</ul>
```

jsx点击标签定位到对应的输入框,通过标签和输入用相同id绑定

```
<label htmlFor='insertArea'>输入内容</label>
<input 
	id='insertArea'
	className='input'   
	value={this.state.inputValue}
	onChange={this.handleInputChange.bind(this)}
/>
```

注释

```javascript
{
//这是单行注释
}

{
    /*这是多行注释*/
}
```

jsx中使用css,需要import并且使用className

```
import './sytle.css'
<input 
	id='insertArea'
	className='input'   
	value={this.state.inputValue}
	onChange={this.handleInputChange.bind(this)}
/>
```

性能提升，减少子组件不必要的渲染，通过利用生命周期

```react
shouldComponentUpdate(nextProps, nextState) {
    if(nextProps.content !== this.props.content) {
        return true;
    }else {
        return false;
    }
}
```

关于作用域的函数放在构造函数中

ajax放在componentDidMount中

```react
import axios from 'axios';
componentDidMount() {
    axios.get('/api/todolist')
    	.then(() => {alert('succ')})
    	.catch(() => {alert('error')})
}
```

```javascript
handleAdd=() => {
    this.setState({
        count: this.state.count + 1
    })
}

handleClick(){
    this.setState({
        count: this.state.count + 1
    })
}

render() {
    return 
    	<div>
    		<button onClick={this.handledAdd}>click</button>
    		<button onClick={this.handleClick.bind(this)}>click<.button>
    	</div>
}
```

es6可以不绑定this使用，使用函数式