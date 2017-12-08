### 模板表单

##### 模板式表单 （FormsModule）

*定义*

表单的数据模型是通过组件模板中的相关指令来定义的，因为使用这种方式定义表单的数据模型时，我们会受限于HTML语法，所以，模板驱动方式只适合用于一些简单的场景。


NgForm ======================> FromGroup

ngForm会拦截标准HTML表单事件，阻止表单的自动提交，使用ngSubmit来提交

如果表单不是定义在本组件中，而是引用子组件的表单<app-from></app-from>，那么这个组件会接管子组件的表单，点击提交的时候，form的action就不起作用

可以使用ngNoForm取消接管

	<form action="" method="" ngNoForm>

	</form>
 

NgModel =====================> FormControl

代表表单中的一个字段，会隐式的创建一个FormControl实例来代表字段的数据模型，
使用FormControl对象来存储字段的值

与NgForm指令类似，NgModel这个指令创建的对象也可以通过一个模板变量来引用，并通过这个模板变量的value属性来访问字段的值

  <input type="password" #password="ngModel" ngModel  name="password"/>

  {{password.value}}

在有标记ngForm的表单中，使用ngModel不需要使用方括号来标记。



NgModelGroup ======================> FromGroup

代表的是表单的一部分，它允许你将一些表单字段组织在一起，形成更清晰的层次关系


**模板中操作**
```
form.component.html

	<form #myform="ngForm" action="/register" (ngSubmit)="onSubmit(myform.value)">
	  
		<input type="text" ngModel  name="username" />
		<input type="" ngModel name="email" />
	  
		<div ngModelGroup="userPassword">
			<input type="password" #password="ngModel" ngModel  name="password"/>
			<input type="password" ngModel name="password_confirm"  />
		</div>

		<input type="submit" name="register"/>
	</form>
	<div class="">
	  {{myform.value | json }}
	</div>
	<div class="">
	  {{password.value}}
	</div>

form.component.ts

	onSubmit(value:any) {
	    console.log(value);
	  }


```

### 响应式表单

##### 响应式表单 (ReactiveFormsModule)

*定义*

使用响应式表单时，你通过编写TypeScript代码而不是HTML代码来创建一个底层的数据模型，这个模型定义好以后，你使用一些特定的指令，将模板上的HTML元素与底层的数据模型连接在一起

1. 需要通过编码创建数据模型
	
	数据模型：用来保存表单数据的数据结构

###### FormControl 

ngModel会为绑定的字段创建FormControl

###### FormgGroup

多个(全部)FormControl的集合(其中有一个FormControl是无效的，那么整个FormControl就是无效的)

###### FormArray

表示一个增长的集合(比如有多个Email地址)，只通过序号来访问FormArray中的元素


2. 需要一些指令将模板中HTML连接到数据模型上

| 类名  | 指 |  令 | 
| -- | -- | -- |
| FormGroup |  formGroup |  formGroupName |
| FormControl | formControl | formControlName |
| FormArray | | formArrayName |
|  |  需要属性绑定语法(加[])↑↑↑   | ↑↑↑|

**TypeScript中操作**

```
reactive-form.component.ts
	username: FormControl = new FormControl('a');

	formModel: FormGroup =  new FormGroup({
		start_date: new FormControl(),
		end_date: new FormControl()
	})

	emails: FormArray = new FormArray([
		new FormControl("zkerpy@163.com"),
		new FormControl("zkerpy@126.com"),
		new FormControl("zkerpy@gmail.com"),
		new FormControl("zkerpy@foxmail.com"),
	])
```

formControlName和formArrayName必须用在formGroup之内使用

formArrayName和*ngFor一起使用


###### 演习

```
reactive-form.component.html
	<form action="" [formGroup]="formModel" (submit)="onSubmit()" >
	  <div class="" formGroupName="dateRange">
	    start_date:<input type="date" formControlName="start_date">
	    end_date:<input type="date" formControlName="end_date">
	  </div>

	  <div class="">
	    <ul formArrayName='emailArray'>
	    		  获取reactive-form.component.ts的formModel内容↓↓↓	 下标↓↓		
	      <li *ngFor='let e of this.formModel.get("emailArray").controls;let i=index;'>
	        <input type="text" [formControlName]="i">
	      </li>
	    </ul>
	    <button type="button" (click)="addEmail()">add Email</button>
	  </div>
	  <div>
	    <button type="submit ">Submit</button>
	  </div>
	</form>

reactive-form.component.ts

	formModel: FormGroup =  new FormGroup({
	    dateRange: new FormGroup({
	      start_date: new FormControl(),
	      end_date: new FormControl()
	    }),
	    emailArray: new FormArray([
	      new FormControl("zkerpy@163.com"),
	      new FormControl("zkerpy@126.com"),
	    ])
	})

	添加Email
	addEmail() {
		获取当前ts下的'emailArray',push一个新的FormControl
		let email = this.formModel.get('emailArray') as FormArray;
		email.push(new FormControl("zkerpy@163.com"));
	}

	onSubmit(value:any){
		console.log(value)
	}

```

###### 实战

```
form.component.html
	<form action="/register" method="post" [formGroup]="formModel" (submit)="onSubmit()" >
	  <div><label for=""></label><input type="text"  formControlName="username"></div>
	  <div><label for=""></label><input type="text"  formControlName="iphone"></div>
	  <div formGroupName="passwordsGroupModel">
	    <div><label for=""></label><input type="text"  formControlName="password"></div>
	    <div><label for=""></label><input type="text"  formControlName="password_confirm"></div>
	  </div>
	  <div><input type="submit" name="" value="Submit" ></div>
	</form>

form.component.ts

	formModel: FormGroup;

	constructor(){

	}
	ngOnInit() {
		this.formModel = new  FormGroup({
		username: new FormControl(),
		iphone: new FormControl(),
		passwordsModel: new  FormGroup({
			password: new FormControl(),
			password_confirm: new FormControl()
			})
		})
	}

	onSubmit(){
		console.log(this.formModel.value)
	}

```
---
```
	使用FormBuilder重构响应式表单

	constructor(fd: FormBuilder) {
	    this.formModel = fd.group({
	      // 数组的第一个元素是FormControl的初始值，第二个元素是校验方法，第三个元素是异步校验方法（参照表单验证）
	      username: ['',...],
	      iphone: [''],
	      passwordsGroupModel: fd.group({
	        password: [''],
	        password_confirm: ['']
	      })
	    })
	}
	ngOnInit() {
	}
```


### 模板式表单和响应式表单的区别

- 不管是哪种表单，都有一个对应的数据模型来存储表单的数据。在模板式表单中，数据模型是由Angular基于你组件模板中的指令隐式创建的。而在响应式表单中，你通过编码明确的创建数据模型然后将模板上的HTML元素与底层的数据模型连接在一起。
- 数据模型并不是一个任意的对象，它是一个由Angular/forms模块中的一些特定的类，如FormControl，FormGroup，FormArray等组成的。在模板式表单中，你是不能直接访问到这些类的。
- 响应式表单并不会替你生成HTML，模板仍然需要你自己来编写。





