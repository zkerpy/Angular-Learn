### 安装nodejs

上官网下node.js安装包,选择LTS（长期维护的版本）

[node.tar.xz](https://nodejs.org/en/)

#### 解压

```
tar  -xvJ -f  node.tar.xz
```

####  添加环境变量

```
sudo vim /etc/profile
添加如下代码
export PATH=/usr/local/share/node/bin:$PATH

马上生效 source /etc/profile
```

#### 测试
```
node -v
```

### 安装angular-cli

#### 测试npm是否可用
```
npm -v
```

#### 改变npm镜像

npm全称Node Package Manager，是node.js的模块依赖管理工具。

```
npm config set registry https://registry.npm.taobao.org

// 配置后可通过下面方式来验证是否成功
npm config get registry
// 或
npm info express
```

#### 命令行安装cli
```
sudo npm install -g @angular/cli
```

#### 测试Angular-cli
```
ng -v
```


#### 启动

Run/Debug Configurations 中选择npm脚本命令

start

如果出现 node executable found in $PATH
说明WebStorm没有添加nodejs
打开Setting>Node.js and NPM>Node interpreter:添加nodejs的安装目录

##### 安装JQuery Bootstrap

```
npm install jquery --save
npm install bootstrap --save
```

##### 安装类型描述文件　

作用：让项目认识$符
```
npm install @types/jquery --save-dev
npm install @types/bootstrap --save-dev
```

### 开发

#### 创新新的项目
```
ng new <project_name>
```


#### 创建新的组件
```
ng g component <component_name>
```


###### 轮播图

利用bootstrap carousel 可以实现
```
	<div class="carousel slide" data-ride="carousel">
	  <ol class="carousel-indicators">
	    <li data-target="#myCarousel1"  class="active"></li>
	    <li data-target="#myCarousel1"  ></li>
	    <li data-target="#myCarousel1"  ></li>
	  </ol>


	  <div class="carousel-inner">
	    <div class="item active">
	      <img src="http://via.placeholder.com/800x300" alt="" class="silde-image">
	    </div>
	    <div class="item">
	      <img src="http://via.placeholder.com/800x300" alt="" class="silde-image">
	    </div>
	    <div class="item">
	      <img src="http://via.placeholder.com/800x300" alt="" class="silde-image">
	    </div>
	    <div class="item">
	      <img src="http://via.placeholder.com/800x300" alt="" class="silde-image">
	    </div>
	  </div>


	  <a  class="carousel-control left" href="javascript:$('.carousel').carousel('prev')">
	    <span class="glyphicon glyphicon-chevron-left"></span>
	  </a>
	  <a  class="carousel-control right" href="javascript:$('.carousel').carousel('next')">
	    <span class="glyphicon glyphicon-chevron-right"></span>
	  </a>
	</div>
```

**重点**
>Jquery要操作页面的DOM（操作页面属性）而Angluar通过数据和模板绑定起来，根据监听数据的变化来改变模板。
操作后台数据，通过数据的变化来改变前台页面。

### 实战

product(产品详情)组件给stars(星级评论)组件传递rating(等级)值

poroduct.component.html

```
  <div>
      <app-stars [rating]="product.rating" ></app-stars>
  </div>

stars.component.ts

	export class StarsComponent implements OnInit {
	  @Input()
	  public rating: number=0;

	  public stars: boolean[];

	  constructor() {
	  }

	  ngOnInit() {
	    this.stars = [];
	    for (let i = 1; i <= 5; i++) {
	      this.stars.push(i > this.rating);
	    }
	  }
	}

stars.component.html

	<span>{{rating}}</span>
```

#### 属性绑定中的样式绑定

```
  <span *ngFor="let star of stars" class="glyphicon glyphicon-star"
  [class.glyphicon-star-empty]="star">
  </span>
```


**参考**

[Linux环境下NodeJS的安装配置（HelloWorld）](https://www.cnblogs.com/zhoulf/p/4042888.html)

[Linux下tar.xz结尾的文件的解压方法](http://blog.csdn.net/silvervi/article/details/6325698/)

[Ubuntu的LTS版本什么意思](http://www.linuxidc.com/Linux/2011-08/40021.htm)

[国内优秀npm镜像](http://riny.net/2014/cnpm/)

[图片播放尺寸提供](https://placeholder.com/)

[Bootstrap 轮播（Carousel）插件](http://www.runoob.com/bootstrap/bootstrap-carousel-plugin.html)








