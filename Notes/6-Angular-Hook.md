### 组件间通讯

#### 组件的输入输出属性

##### @Input

```
app.component.html

	<div class="">我是父组件</div>
	<div class="">
	  <input type="text" placeholder="请输入股票代码" [(ngModel)]="stock">
	  <app-zorder [stockCode]="stock" [amount]="100"></app-zorder>
	</div>

app.component.ts

	export class AppComponent {
	  stock = '';
	}
zorder.component.html

	<p>
	  子组件
	</p>
	<div>买{{amount}} 手 {{stockCode}}股票</div>

zorder.component.ts
	
	export class ZorderComponent implements OnInit {
	  @Input()
	  stockCode: string;
	  @Input()
	  amount: number;
	  constructor() {
	    setInterval(
	      () => {
	        this.stockCode = "Apple";
	      },3000
	    )
	  }
	  ngOnInit() {
	  }
	}
```

这个例子中属性绑定是单向的，是从父组件到子组件
子组件的属性变化不会影响父组件的变化

##### 路由

参考Angular-Route那一章

##### @Output

场景：模拟股票交易市场实时显示

```
price-quote.component.html
	
	<div>股票价格{{stockCode}} 股票金额{{Price | number:'2.2-2'}}</div>

price-quote.component.ts

	export class PriceQuoteComponent implements OnInit {

	  stockCode:string = "Apple";
	  Price:number;

	  constructor() {
	    setInterval(() =>{
	      let priceQuote:PriceQuote= new PriceQuote(this.stockCode,100*Math.random());
	      this.Price=priceQuote.lastPrice;
	    },1000)
	  }

	  ngOnInit() {
	  }

	}
	export class PriceQuote{
	  constructor(
	    public stockCode:string,
	    public lastPrice:number,
	  ){}
	}

app.component.html

	<app-price-quote></app-price-quote>
```

信息输出到组件外部，信息通过price-quote组件传递到app父组件

```
price-quote.component.html
	
	<div>股票价格{{stockCode}} 股票金额{{Price | number:'2.2-2'}}</div>

price-quote.component.ts

	export class PriceQuoteComponent implements OnInit {

	  stockCode:string = "Apple";
	  Price:number;

	  // EventEmitter既可以是发射事件，又可以订阅事件
	  // 提供给其他组件访问
	  //   改名字↓↓↓↓↓↓↓↓↓↓↓↓↓↓
	  // @Output('priceChange')
	  @Output()
	  outputPrice:EventEmitter<PriceQuote>=new EventEmitter();

	  constructor() {
	    setInterval(() =>{
	      let priceQuote:PriceQuote= new PriceQuote(this.stockCode,100*Math.random());
	      this.Price=priceQuote.lastPrice;

	      // 这个泛型所指定这个类型的变量输出
	      this.outputPrice.emit(priceQuote);
	    },1000)
	  }

	  ngOnInit() {
	  }

	}
	export class PriceQuote{
	  constructor(
	    public stockCode:string,
	    public lastPrice:number,
	  ){}
	}

app.component.html

	<!--和输出属性的名字一样↓↓↓↓↓@Output-->
	<app-price-quote (outputPrice)="priceQuoteHandler($event)"></app-price-quote>
	<div class="">
	  这是报价组件外部
	  股票代码是{{priceQuote.stockCode}}
	  股票代码是{{priceQuote.lastPrice}}
	</div>

app.component.ts

	export class AppComponent {

	  priceQuote:PriceQuote=new PriceQuote("", 0)
	  priceQuoteHandler(event: PriceQuote){
	    this.priceQuote=event
	  }
	}


```

stockCode&&Price >> @Output (EventEmitter.emit) >>  app-price-quote (@Output_name) >> PriceQuote 


#### 使用中间人模式传递数据

场景：当交易金额到达某一个值的时候要买这个股票


当价格到达一定的值的时候，点击price-quote.component.html的按钮 price-quote.component.ts（@Output）发送信息给中间人

```
price-quote.component.html

	<input type="button" value="click" (click)="buyStock($event)">

price-quote.component.ts

	@Output()
	buy:EventEmitter<PriceQuote>=new EventEmitter();
	
	前台点击
	buyStock(event){
		this.buy.emit(new PriceQuote(this.stockCode,this.Price));
	}

```

中间人接受到信息

```
app.component.html

	<app-price-quote (buy)="buyHandler($event)"></app-price-quote>
	<app-order [priceQuote]="priceQuote"></app-order>

app.component.ts

	priceQuote:PriceQuote=new PriceQuote("", 0)
	buyHandler(event: PriceQuote){
		this.priceQuote=event
	}

```

orderz组件（@Input）

```
order.component.ts
	
	@Input
	priceQuote:PriceQuote;

order.component.html

	<div>
	  {{priceQuote.stockCode}}股票
	  价格是{{priceQuote.lastPrice}}
	</div>
```

如果两个组件没有共同的父组件，那么使用服务来解耦合


#### 组件生命周期以及angular的变化发现机制

![组件生命周期钩子](https://github.com/zkerpy/Angular-Learn/blob/master/Notes/Hook.png)

绿色的方法只会被调用一次

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



用户只是改变了可变对象user的属性name，user对象的引用自身是没有改变的。ngOnChanges没有被调用
ngOnChanges只有是输入属性变化时才被调用，所以message变化并没有打印log



