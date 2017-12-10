### Angular 依赖注入

什么是依赖注入模式及使用依赖注入的好处

介绍Angular的依赖注入实现：注入器和提供器

注入器的层级关系


依赖注入：Dependency Injection 简称DI  手段

控制反转：Inversion of Control 简称IOC 目的

演练
```
    @NgModule({
        提供器
        providers: [ProductService] 那些对象需要依赖注入 <==> providers:[{provide:ProductService,useClass:ProductService}]
        # providers：[{provide:ProductService,useClass:AnotherProductService}]
        # providers：[{provide:ProductService,useFactory:() => {...}]
    })

    export class ProductComponent{
        product: Product;
        注入器
        构造函数声明需要一个ProductService的token
        constructor(productService: ProductService){
            this.product=productService.getProduct()	
        }
    }
```
- 创建组件

	ng g component productinfo

- 创建服务

	ng g service shared/product


1. 先写服务，通过这个服务获取商品信息 

```
product.service.ts

    Injectable()
	export  class  ProductService{
	  constructor(){}
	  getProduct():Product {
	    return new Product(0, "Mi", 1999,"very good")
	  }
	}
	export class  Product{
	  constructor(
	    public id: number,
	    public title: string,
	    public price: number,
	    public desc: string,
	  ){}
	}

```
2.  模块的声明 app.module.ts

```
	@NgModule({
	  imports: [RouterModule.forRoot(routes)],
	  exports: [RouterModule]

	  providers: [ProductService],

	})	

```
3. 获取这个服务

```
	export class ProductinfoComponent implements OnInit{
	  product: Product;
	  constructor(private productService:ProductService){}
	  ngOnInit(){
	    this.product=this.productService.getProduct();
	  }
	}
```
4.  productinfo.component.html显示

```
    {{product.title}}
    {{product.price}}
    ...
```

---

1. 先写AnotherProductService服务，实现ProductService

```
product.service.ts

    Injectable()
	export  class AnotherProductService implements ProductService{
	  constructor(){}
	  getProduct():Product {
	    return new Product(1, "Apple", 5000,New IPad")
	  }
	}

```

2. 获取这个服务

```
    @Component({
        ...
        providers:[{
            provide:ProductService,useClass:AnotherProductService
        }]
    })

	export class Productinfo2Component implements OnInit{
	  product: Product;
	  constructor(private productService:ProductService){}
	  ngOnInit(){
	    this.product=this.productService.getProduct();
	  }
	}
```
3.  productinfo.component.html显示

```
    {{product.title}}
    {{product.price}}
    ...
```

4. 修改app.component.html
```
    <app-productinfo></app-productinfo>
    <app-productinfo2></app-productinfo2>
```

#### 使用工厂和值声明提供器

app.module.ts
```
    providers: [{
      provide: ProductService,
      useFactory: () => {
        let logger = new LoggerService();
        let dev = Math.random() > 0.5;
        if (dev) {
          return new ProductService(logger);
        }else {
          return new AnotherProductService(logger);
        }
      }
    }
    , LoggerService],
```

**工厂方法创建的对象是一个单例对象**

##### 升级

使用LoggerService提供器 

```
  providers: [{
    provide: ProductService,
    useFactory: (logger: LoggerService) => {
      let dev = Math.random() > 0.5;
      if (dev) {
        return new ProductService(logger);
      }else {
        return new AnotherProductService(logger);
      }
    },
    deps: [LoggerService]
  }
  , LoggerService],
```

使用单一变量(属性)作为提供器

```
  providers: [{
    provide: ProductService,
    useFactory: (logger: LoggerService,   isDev  ) => {
      if (  isDev  ) {
        return new ProductService(logger);
      }else {
        return new AnotherProductService(logger);
      }
    },
    deps: [LoggerService,"IS_DEV_ENV"]
  }
  , LoggerService, {
        provide: "IS_DEV_ENV", useValue: false
    }],
```

使用值对象作为提供器

```
  providers: [{
    provide: ProductService,
    useFactory: (logger: LoggerService,   appConfig  ) => {
      if (  appConfig.isDev  ) {
        return new ProductService(logger);
      }else {
        return new AnotherProductService(logger);
      }
    },
    deps: [LoggerService,"APP_CONFIG"]
  }
  , LoggerService, {
        provide: "APP_CONFIG", useValue: {isDev: false}
    }],
```

#### 注入器

模块声明需要的提供器都注册到应用级注入器里面

appModule的provide && import ==>AppComponent

应用级注入器 =>> 主组件注入器 =>> 子组件注入器


**只有一个注入方法：构造函数注入**

*手动注入*

```
export class ProductdetailComponent implements OnInit {
  producted: Product;
  // constructor(private productService: ProductService){}
  // 手动注入
  public productService: ProductService;
  constructor(private injector: Injector) {
    this.productService = injector.get(ProductService);
  }

  ngOnInit() {
    this.producted = this.productService.getProduct();
  }
}
```