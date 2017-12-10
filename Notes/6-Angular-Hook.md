### Hook(钩子)

#### 组件生命周期以及angular的变化发现机制

#### OnChanges

![组件生命周期钩子](https://github.com/zkerpy/Angular-Learn/blob/master/Img/Hook.png)

红色的方法只会被调用一次

跟Android的生命周期有点像


var greeting="Hello"
greeting="Hello world"

内存中有两个字符串 两个字符串都是不可变的，
对greeting变量来说是改变的（指向字符串的内存地址发生改变“Hello”>>"Hello world"）


var user={name:"Tom"}
user.name="Jerry"

对于user对象来说，变得是user.name指向的内存地址，而user对象内存地址不变 


##### 实战

```
zchild.component.html

	<div class="">
	  <h2>child</h2>
	  <div class="">{{greeting}}</div>
	  <div class="">{{user.name}}</div>
	  <div class="">messgage <input [(ngModel)]="message"></div>
	</div>


zchild.component.ts

	export class ZchildComponent implements OnInit , OnChanges {

		@Input()
		greeting:string;

		@Input()
		user: {name:string};

		message:string;

		constructor() { }

		ngOnInit() {
		}
		ngOnChanges(changes: SimpleChanges): void {
			console.log(JSON.stringify(changes, null, 2));
		}

	}


app.component.ts

	export class AppComponent {

		greeting: string = "Hello";
		user: {name:string} = {name:"Tom"};

	}

app.component.html

	<div class="">
	  <h2>fu</h2>
	  <div class=""><input type="text" [(ngModel)]="greeting"></div>
	  <div class=""><input type="text" [(ngModel)]="user.name"></div>
	</div>
	<hr/>

	<app-zchild [greeting]="greeting" [user]="user"></app-zchild>
```

用户只是改变了可变对象user的属性name，user对象的引用自身是没有改变的，所以ngOnChanges没有被调用。

ngOnChanges只有是输入属性变化时才被调用，所以message变化并没有打印log


#### 变更检测

zone.js 保证组件属性的变化和页面的变化同步的

![变更检测](https://github.com/zkerpy/Angular-Learn/blob/master/Img/变更检测.png)

#### DoCheck

```
	oldUsername: string;

	changeDetected: boolean = false;

	noChangeCount: number;

	ngDoCheck(): void {
		if (this.user.name !== this.oldUsername) {
			this.changeDetected = true;
			console.log('change');
			this.oldUsername = this.user.name;
		}
		if (this.changeDetected) {
			<this class="n">noChangeCount = 0;</this>
		}else {
			// user.name没有变化，doDocheck也会被调用
			this.noChangeCount = this.noChangeCount + 1;
			console.log("被调用"+this.noChangeCount);
		}
		this.changeDetected = false;
	}
```


#### 父组件调用子组件的方法

在子组件定义一个方法

```
	greetingfun(name: string) {
	    console.log("hello" + name);
	}
```

##### 父组件的代码中调用

```

	<app-zchild #child1></app-zchild>

	@ViewChild("child1")
	child: ZchildComponent;

	ngOnInit(): void {
		this.child.greetingfun("zkerpy");
	}
```



##### 父组件的模板中调用

```

	<app-zchild #child2></app-zchild>
	<button (click)="child2.greetingfun('Tom')">Hello Tom</button>
```

#### AfterViewInit AfterViewChecked

组件视图都组装完毕才调用，如果这个时候企图通过这个两个钩子改变视图会报错(解决方法是跳出当前生命周期（setTimeout()）等)

子组件初始化完毕>子组件变更检测完毕>父组件初始化完毕>父组件变更检测完毕

子组件变更检测完毕>父组件变更检测完毕

#### 投影(页头和页脚)

```
app.component.html

	<div class="wrapper">
	  <h2>父组件</h2>
	  <div class="">这个div定义在父组件中</div>
	  <app-zchild>
	    <div class="header">这个div是父组件投影到子组件的{{title}}</div>
	    <div class="footer">这个div是父组件投影到子组件的</div>
	  </app-zchild>
	</div>

child.component.html
	<div class="wrapper">
	  <h2>子组件</h2>
	  <div class="">这个div定义在子组件中</div>
	  <ng-content select=".header"></ng-content>
	  定义一个投影点
	  <ng-content select=".footer"></ng-content>
	</div>
```

>Tip

```
	divContent="<div>hello world</div>"
	<div [innerHTML]="divContent"></div>
```

#### AfterContentInit AfterContentChecked


在被投影进入的内容组装完成后调用,当然这个时候企图改变视图不报错,因为Angular的调用顺序是
投影进来的视图内容>>子组件视图内容>>父组件视图内容

运行时动态改变组件内容
