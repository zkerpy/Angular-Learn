### 数据绑定

\<h1\>{{}}\</h1\>

使用插值表达式将一根表达式的值显示在模板上

<img [src]="">

使用方括号将HTML标签的一个属性绑定到一个表达式上

<button (click)="toProductDetail()"></button>

使用小括号将组件控制器的一个方法绑定为模板上一个事件的处理器

#### 事件绑定


小括号表示这是 一个事件绑定 当事件发生时执行的表达式

<input (event_name)="Component_function_name($event)">

*等号右侧可以是一个属性赋值*

<button (click)="saved = true"></button>

*被绑定的事件即可以是标准的DOM事件，也可以是自定义事件*

##### 实战

```
bind.component.html

	<button (click)="doOnClick($event)">Onclik me</button>

bind.component.ts

	doOnClick(event:any){
		console.log(event)
	}

app.component.html

	<app-bind></app-bind>

```

#### 属性绑定

*插值表达式和属性绑定是一个东西*

##### 实战

```
bind.component.html

	<button (click)="doOnClick($event)">Onclik me</button>

	<img [src]="imgUrl">
	<!-- 插值表达式写法 -->
	<img src="{{imgUrl}}">

bind.component.ts

	imgUrl: string = ""

	doOnClick(event:any){
		console.log(event)
	}

app.component.html

	<app-bind></app-bind>

```

##### HTML属性和DOM属性的区别

```
bind.component.html

	<input value="Tom" (input)="doOnInput($event)">

bind.component.ts

	doOnInput(event:any){
		// DOM属性 取决于你输入的值
		console.log(event.target.value)

		// HTML属性 Tom
		console.log(event.target.getAttribute('value'))
	}

app.component.html

	<app-bind></app-bind>

```

##### HTML属性和DOM属性的关系

- 少量HTML属性和DOM属性之间有着1:1的映射 如:id
- 有些HTML属性 没有对应的DOM属性  如:colspan
- 有些DOM属性  没有对应的HTML属性 如:textContent
- HTML 属性和DOM 属性 不是同一样东西
- **HTML属性的值指定了初始值;DOM属性的值表示当前值**
- DOM属性可变，HTML属性不可变
- **模板绑定是通过 DOM属性 和 事件 来工作的,而不是HTML属性**


##### HTML属性绑定

###### 基本HTML属性绑定
```
	<td [attr.colspan]="tableColspan">Something</td>
```

###### 实战

```
	<table>
		<tr>
			<td [attr.colspan]="{{size}}"></td>
		</tr>
	</table>

	size:number = 2;
```



###### CSS类绑定

```
	<!-- someExpression的值会替换掉zzz kkk -->
	<div class="zzz kkk" [class]="someExpression">something</div>  

	<!-- 当isSpecial值为true这个div就有class="special" -->
	<div [class.special]="isSpecial">something</div>

	<!-- 同时控制多个CSS类是否显示 zzz,kkk就是CSS类的名字，isA，isB控制这个CSS类是否显示 -->
	<div [ngClass]="{zzz:isA,kkk:isB}"></div>
```

###### 实战

```
bind.component.html

	<div class="a b c" >zkerpy1</div>
	<div [class]="class_style" >zkerpy2</div>

	<div class="a b" [class.c]="isStyle" >zkerpy3</div>

	<div [ngClass]="divStyleClass">zkerpy4</div>

bind.component.ts

		class_style:string ;

		isStyle: boolean;

		divStyleClass:any={
			a:false,
			b:false,
			c:true
		};

		constructor() { }
		ngOnInit() {
			setTimeout(() => {
			  this.class_style="a b";

			  this.isStyle=true;

			  this.divStyleClass={
			  	a:true,
			  	b:true,
			  	c:false
			  }

			},3000)
		}
```

###### 样式绑定

```
	<!-- isSpecial为true，color就为red -->
	<button [style.color] = "isSpecial ? 'red':'green'"></button>
	<div [ngStyle="{'font-style:this.canSave ? 'italic':'normal'}"]></div>
```

#### 双向绑定  

属性绑定 控制器>>>模板

事件绑定 模板>>>控制器

```
	bind.component.html
	<input [value]="name" (input)="doOnInput($event)">
	{{name}}

	bind.component.ts
	export class BindComponent implements OnInit {
		constructor() { }
		ngOnInit() {
		setInterval(
		  () =>{
		    this.name="Tom";
		  },3000
		)
		}
		name:string;
		doOnInput(event){
		this.name=event.target.value;
		}
	}
```

doOnInput > 改变value > 赋值给组件name > [value]="name" && {{name}}

*简化(骚)操作   盒子里装香蕉* 

```
<input [(ngModel)]="name">
```

**Error**

[Can't bind to 'ngModel' since it isn't a known property of 'input'](https://stackoverflow.com/questions/43298011/angular-4-cant-bind-to-ngmodel-since-it-isnt-a-known-property-of-input)


### 响应式编程

![响应式编程](https://github.com/zkerpy/Angular-Learn/blob/master/Notes/Observable.png)


```
bind.component.html
	export class BindComponent implements OnInit {
	  constructor() {
	  	↓↓↓流↓↓↓
	    Observable.from([1, 2, 3, 4])
	      .filter(e => e % 2 == 0)
	      .map(e => e * e)
	      .subscribe(
	        ↓↓↓观察者↓↓↓
	        e => console.log(e),
	        err => console.error(err),
	      )
	  }
	  ngOnInit(): void {
	  }
	}
```
**Error**
TS2339: Property 'from' does not exist on type 'typeof Observable'

>Seems like

>replacing: import {Observable} from 'rxjs/Observable'

>by: import {Observable} from 'rxjs/Rx'

>makes it work.

[TS2339: Property 'from' does not exist on type 'typeof Observable'](https://github.com/ReactiveX/rxjs/issues/1694)

#### 使用响应式编程做事件处理

在浏览器中产生的每一个事件，在JavaScript里面都会被封装成event对象

Angular既可以处理标准的DOM事件，也可以处理自定义的事件

```
bind.component.html
	<input (keyup)="onKey($event)" />

bind.component.ts
	onKey(event) {
	    console.log(event.target.value);
	  }

```

##### 模板本地变量

```
bind.component.html
	<input #myField (keyup)="onKey(myField.value)" />

bind.component.ts
	onKey(value:string) {
	    console.log(value);
	  }
```
*将事件作为永不结束的流来处理*

每个表单元素都有自己的FormControl对象 
一个表单元素的值发生变化的时候，就会触发FormControl，
FormControl会发射一个ValueChange事件，
这些ValueChange事件就会组成一个可订阅的流

##### 实战

1. app.module.ts导入ReactiveFormsModule模块
2. 使用FormControl去监听input变化

```
bind.component.html
	<input [formControl]="searchInput" />

bind.component.ts
	export class BindComponent implements OnInit {
	  searchInput: FormControl = new FormControl();
	  constructor() {
	  	↓↓↓流（被观察者）↓↓↓
	    this.searchInput.valueChanges
			.debounceTime(500)
						↓↓↓观察者↓↓↓
			.subscribe(stockCode => this.getStockInfo(stockCode))
	  }
	  getStockInfo(value:string) {
	    console.log(value);
	  }
	  ngOnInit(): void {
	  }
	}
```
#### 管道

原始值 =>> 显示值

{{birthday| date | uppercase}}

{{birthday| date:'yyyy-MM-dd HH:mm:ss' | uppercase}}

3.1415926

{{pi | number:'2.2-2'}} 3.14

{{pi | number:'2.1-4'}} 3.1416 四舍五入

3

{{pi | number:'2.1-4'}} 3.0

##### 自定义管道

1. 新建管道
```
	ng g pipe pipe/multiple
```
2. 声明在模块的declarations里
```
	@NgModule({
	  declarations: [
	    AppComponent,
	    ProductinfoComponent,
	    ProductdetailComponent,
	    BindComponent,
	    MultiplePipe
	  ],
	  ...
	})
```
3. 编写
```
	import { Pipe, PipeTransform } from '@angular/core';

	@Pipe({
	  name: 'multiple'
	})
	export class MultiplePipe implements PipeTransform {
				例子:  date 	: 'yyyy-MM-dd HH:mm:ss'
				↓原始值↓	↓可选参数↓
	  transform(value: any, args?: any): any {
	  	if(!args){
	  		args=1;
	  	}
	    return value*args;
	  }
	}
```