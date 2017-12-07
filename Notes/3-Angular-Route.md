### Angular路由

#### 新建路由项目

```
ng new <router-project-name> --routing
```
#### 新建组件

```
ng g component <component-name>
```

#### Routes

- 在app-routing.module.ts中配置路由

```
const routes: Routes = [
  {path:'', component: HomeComponent},
  {path:'product', component: ProductComponent}
];
```
*path不使用/开头，是为了在多个试图间导航时可以自由的使用绝对路径和相对路径*

- app.component.html

```
<a [routerLink]="['/product']">Product</a>
<a [routerLink]="['/']">Home</a>
<input type="button" value="Detail Prodcut" (click)="toProductDeails()">
```

**path的顺序有很大的差别** 
>如按上面的配置，那么在app.component.html中，跳转不到product链接，必须把他们位置对调


对调后如下
```
const routes: Routes = [
  {path:'product', component: ProductComponent}
  {path:'', component: HomeComponent},
];
```

>先匹配者优先的原则，要把具体的路由放在前面

- app.component.ts

```
export class AppComponent {
  title = 'app';
  constructor(public  router : Router){}
  鼠标点击事件
  toProductDeails(){
    this.router.navigate(['/product'])
  }
}
```

#### 在路由时传递数据

##### 在查询参数中传递数据

```
		　　　　获取
/product?id=2&name=2 <==== ActivatedRoute.queryParams[id]
```
在商品详情组件接受这个参数

使用ActivatedRoute(当前激活的路由对象，保存着当前路由的信息，如路由地址，路由参数等)

1. 在app.component.html中配置要传入的参数

```
<a [routerLink]="['/product']" [queryParams]="{id: 1}">Product</a>
```
2. 在product.component.ts使用ActiveRoute

```
export class ProductComponent implements OnInit {
  // 接受这个参数
  public productId: number;
  constructor(public routeInfo: ActivatedRoute){}
  ngOnInit() {
    this.productId = this.routeInfo.snapshot.queryParams["id"];
  }
}
```
3. 显示这个ID

```
{{productId}}
```
##### 在路由路径中传递数据

```
				   获取
{path:/product/:id} => /product/1 <=== ActivedRoute.params[id]
```
1. 修改app-routing.module.ts的path属性使其可以携带参数

```
{path:'product/:id', component: ProductComponent},
```
2. 修改app.component.html路由链接的参数
```
<a [routerLink]="['/product',1]">Product</a>
```
3. 改变商品详细信息的组件取值方式 查询参数中取数据 <==> URL中取数据

```
export class ProductComponent implements OnInit {
  // 接受这个参数
  public productId: number;
  constructor(public routeInfo: ActivatedRoute){}
  ngOnInit() {
    this.productId = this.routeInfo.snapshot.params["id"];
  }
}
```
*queryParams->params*

>activatedRouter 构造之后在ngOnInit （在组件第一次呈现的时候只被调用一次）生命周期中调用的**snapshot是参数快照**；
还有一种得参数订阅，如果在组件路由到自身的情况下，必须使用**参数订阅（subscribe）**，才能取到动态的参数；

订阅方式

```
this.routeInfo.params.subscribe((params: Params) => this.productId =params["id"]);
```
##### 在路由配置中传统数据

```
{path:/product,component:ProductComponent,data:[{isProd:true}]} => ActivatedRoute.data[0][isProd]
```

##### 重定向路由

在用户访问一个特定的地址时，将其重定向到另一个指定的地址

原来的地址访问不了，定向到新地址

www\.codefork.me => love.codefork.me

```
{path:'', redirectTo: '/home',pathMatch: "full"},
```

##### 子路由


```
app.component.html

  <router-outlet></router-outlet>

product.component.html

  <a [routerLink]="['./']">产品信息</a>
  <a [routerLink]="['./saleinfo', 22]">销售员信息</a>
  <router-outlet></router-outlet>

app-routing.module.ts

  {path:'product', component: ProductComponent,
    children: [
      {path: "saleinfo/:sId",component: SalerInfoComponent},
      {path: "",component: DetailproductComponent},
    ]
  },
  
saler-info.component.ts

  export class SalerInfoComponent implements OnInit {
    public Id: number;
    constructor(public routeInfo: ActivatedRoute) { }
    ngOnInit() {
      this.Id=this.routeInfo.snapshot.params['sId']
    }
  }

saler-info.component.html
  
  {{Id}}

```


##### 辅助路由 (兄弟关系)

1. 在app组件的模板上再定义一个插座来显示聊天面板
2. 单独开发一个聊天组件(chat),只显示在新定义的插座上
3. 通过路由参数控制新插座是否显示聊天面板

```
app.component.html

  <router-outlet></router-outlet>
  <router-outlet name="auxname"></router-outlet>

app-routing.module.ts

  {path:'chat', component: ChatComponent,outlet: 'auxname'},

app.component.html

  <a [routerLink]="[{primary: 'home' outlets: {aux: 'chat'}}]">开始聊天</a>
  <a [routerLink]="[{outlets: {aux: null}}]">结束聊天</a>
```

##### 路由守卫

使用场景:

- 只有当用户已经登录并拥有某些权限时才能进入某些路由
- 一个由多个表单组件组成的向导,例如注册流程,用户只有在当前路由的组件中填写了满足要求的信息才可以导航到下一个路由
- 当用户未执行保存操作而试图离开当前导航时提醒用户

为了解决这些问题,Angular提供一些钩子:

- CanActive: 处理导航到某路由的情况
- CanDeactivate: 处理从当前路由离开的情况
- Resolve:　在路由激活之前获取路由数据 


guard/

```
login.guard.ts

	export class  LoginGuard implements CanActivate{
		canActivate() {
			let loggedIn : boolean = Math.random() < 0.5;
			if (!loggedIn){
				console.log('login  ---');
			}
			return loggedIn;
		}
	}

unsaveGuard.ts
						要保护组件的类型↓↓↓↓↓↓↓
	export class UnsavedGuard implements CanDeactivate<ProductComponent> {
	  canDeactivate(component: ProductComponent,) {
	    return window.confirm('确定离开吗');
	  }
	}

app-routing.module.ts

	{path: 'chat', component: ChatComponent,outlet: 'aux',
		canActivate : [LoginGuard],
		canDeactivate : [UnsavedGuard],
	},

	providers:[LoginGuard, UnsavedGuard , ProductResolve]
						
```

##### Resolve

在导航到某一路由下，预先加载好数据

```

productResolve.guard.ts

	@Injectable()
	export class ProductResolve implements Resolve<Product> {
	  constructor(private router: Router) {

	  }

	  resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Product> | Promise<Product> | Product {
	    let productId: number = route.params['id'];
	    //如果获取的id是1的时候返回
	    if (productId == 1) {
	      return new Product(1, "Mi");
	    } else {
	      this.router.navigate(['/home']);
	      return undefined;
	    }
	  }
	}


app-routing.module.ts

	{path: 'product/:id', component: ProductComponent,
	    children: [
	      {path: "info/:sId", component: SelierInfoComponent},
	      {path: "", component: DetailproductComponent},
	    ], resolve : {
	      product : ProductResolve
	    }
	  },

	providers:[LoginGuard, UnsavedGuard , ProductResolve]

app.component.html

	<a [routerLink]="['/home']">Home</a>
	<a [routerLink]="['/product',1]">Product</a>
	<router-outlet></router-outlet>

product.component.ts

	export class ProductComponent implements OnInit {

	  // 接受这个参数
	  public productId: number;
	  public productName: string;

	  constructor(public routeInfo: ActivatedRoute) {}

	  ngOnInit() {
	       app-routing.module.ts里面resolve的key↓↓↓
	    this.routeInfo.data.subscribe((data: {product: Product}) => {
	      this.productId = data.product.id;
	      this.productName = data.product.title;
	    });
	  }

	}
	
	export  class Product{
	  constructor(
	    public id: number,
	    public title: string,
	  ) {}
	}
	
product.component.html
	<p>
	  这里是商品信息组件
	</p>

	<p>
	  商品id是 {{productId}}
	</p>
	<p>
	  商品id是 {{productName}}
	</p>

	<a [routerLink]="['./']">产品信息</a>
	<a [routerLink]="['./info', 22]">销售员信息</a>

	<router-outlet></router-outlet>

``` 


*约定*

路由名和组件名一致

**参考**

[Angular 传递参数](http://www.jianshu.com/p/183010077253)





