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

#### 使用中间人模式传递数据



#### 组件生命周期以及angular的变化发现机制
