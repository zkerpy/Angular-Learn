### 表单验证

#### 校验响应式表单

```

	自定义校验器
	xxx(control: AbstractControl):{[key:string]:any}{
	    // 预定义校验器
	    // Validators.maxLength(),Validators.required等
	    return null;
	}


	username:['',[Validators.required,Validators.minLength(6)]]
	亦可以打印错误信息
	let errors:any = this.formModel.get("username").errors

```

##### 演习

```
	mobileValidator(control:FormControl):any{
		var myreg=/^(((13[0-9]{1})|(15[0-9]{1})|(18[0-9]{1}))+\d{8})$/;
		let valid=myreg.test(control.value);
		return valid ? null:{mobile: true}
	}

	mobile:['',this.mobileValidator]
```

**为FormGroup提供校验器**

```
	passwordEqualValidator(group:FormGroup):any{
		var password: FormControl=group.get("password") as FormControl;
		var password_confirm: FormControl=group.get("password_confirm") as FormControl;
		let valid:boolean = password.value === password_confirm.value;
		return valid ? null:{equal: true}
	}

	passwordsGroupModel: fd.group({
		        password: [''],
		        password_confirm: ['']
		      },{validator:this.passwordEqualValidator})
```

##### 定义全局校验方法 

新建validator/validators.ts
注入FormControl的方法放在FormGroup之前

```
	export function passwordEqualValidator(group:FormGroup):any{
		var password: FormControl=group.get("password") as FormControl;
		var password_confirm: FormControl=group.get("password_confirm") as FormControl;
		let valid:boolean = password.value === password_confirm.value;
		//return valid ? null:{equal: true}
		return valid ? null:{equal: {'describes':'请与新密码保持一致'}
	}

```

当整个表单的数据都合法后才可以向后台发送数据

```
onSubmit(){
	if(this.formModel.valid){
		console.log(this.formModel.value)
	}
}
```

前端显示不通过的信息

```
	<div formGroupName="passwordsGroupModel">
		<div><label for=""></label><input type="text"  formControlName="password"></div>
		<div class="">
			<div [hidden]="!formModel.hasError('minlength',['password','passwordsGroupModel'])">
				必填项
			</div>
			<div [hidden]="!formModel.hasError('minlength',['password','passwordsGroupModel'])">
				密码长度大于6位
			</div>		
		</div>
		<div><label for=""></label><input type="text"  formControlName="password_confirm"></div>
		<div [hidden]="formModel.hasError('equal')">请与新密码保持一致</div>
								返回结果名(key)↓，检查字段名↓
		<div [hidden]="!formModel.hasError('equal','passwordsGroupModel')">
			<!-- 请与新密码保持一致 -->
			{{formModel.getError('equal')?.describes}}
		</div>
	</div>
	
```

**异步校验器**

```
	mobileAsyncValidator(control:FormControl):any{
		var myreg=/^(((13[0-9]{1})|(15[0-9]{1})|(18[0-9]{1}))+\d{8})$/;
		let valid=myreg.test(control.value);
		return Observable.of(valid ? null:{mobile: true})
	}

	mobile:['',this.mobileValidator,mobileAsyncValidator]

```

**状态字段**

touched和untouched

用户是否**访问**过这个字段，也就是说这个字段是否获取过焦点（错误信息是显示）

```
	<div [hidden]="formModel.get('username').valid || formModel.get('username').untouched">
		<div [hidden]="!formModel.hasError('required','username')">
			必填项
		</div>
		<div [hidden]="!formModel.hasError('minlength','username')">
			密码长度大于6位
		</div>		
	</div>
```

dirty和pristine

用户是否修改过这个字段的**值**

```
	<div [hidden]="formModel.get('mobile').valid || formModel.get('mobile').pristine">
		<div [hidden]="!formModel.hasError('required','mobile')">
			必填项
		</div>
		<div [hidden]="!formModel.hasError('minlength','mobile')">
			密码长度大于6位
		</div>		
	</div>
```

pending

正在校验


```
	<div [hidden]="!formModel.get('mobile').penging">
		正在校验
	</div>
```




#### 校验模板式表单

只能通过指令来修改,将校验器方法包装成一个指令(ngFor,ngModel等)来使用

新建指令

```
	ng g directive directive/mobileValidator
```
---

```
mobile-validator.directive.ts

	@Directive({
				 ↓方括号告诉使用者,指令要作为属性来用
	  selector: '[appMobileValidator]',
	  							token↓↓↓↓↓		全局校验方法↓↓↓↓↓↓	一个token下面放多个值
	  providers : [{provide: NG_VALIDATORS , useValue : mobileValidator, multi : true }],
	})
	export class MobileValidatorDirective {

	  constructor() { }

	}

zform.component.html
														不使用HTML5默认的校验方式↓↓↓↓↓↓↓↓↓
	<form #myform="ngForm" action="/register" (ngSubmit)="onSubmit(myform.value) novalidate">
	  <div ngModelGroup="userInfo">
	  						自定义 ↓↓↓↓	 Angular预定义的校验↓↓↓↓
	    <input type="text"  appMobileValidator ngModel  required name="username" />
	    <input type="" ngModel name="email" />
	  </div>


	  <input type="password" #password="ngModel" ngModel  name="password" (click)="onValidate(myform)"/>
	  <div [hidden]="passwordValid  ||  passwordUntouched">
		  <div [hidden]="!myform.form.hasError('required','password')">
		    必填项
		  </div>
		  ...
	  </div>
	  <input type="password" ngModel name="password_form"  />
	  <input type="submit" name="register"/>
	</form>


zform.component.ts

	passwordValid:boolean = true;
	passwordUntouched:boolean = true;

	onValidate(form:NgForm){
		if(form){
			this.mobileValid=form.form.get('password').valid:
			this.passwordUntouched:boolean = form.form.get('mobile').untouched;
		}
	}


```



#### 总结

响应式表单在处理复杂表单的时候更有优势：是同步的，立即可用



