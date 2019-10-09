# MUI 使用 mui.getjson获取接口数据，并且对数据进行处理。然后用vue进行数据绑定代码。

```html
<!DOCTYPE html>
<html>

	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no" />
		<title>首页</title>
		<script src="js/mui.min.js"></script>
		<link href="css/mui.min.css" rel="stylesheet" />
		<link href="css/style.css" rel="stylesheet" />
		<style>
			html,
			body {
				background-color: white;
			}
			
			.wrap-search {
				margin: 15px;
				background: #E6E6E6;
				height: 30px;
				border-radius: 5px;
				text-align: center;
			}
			
			.item-img {
				width: 60px;
				height: 90px;
				margin-right: 10px;
			}
		</style>
	</head>

	<body>
		<div class="mui-content " style="background: #fff;">
			<div class="wrap-search">
				<span class="mui-icon mui-icon-search" style="line-height: 30px;color: #aaa;font-size: 16px;"></span>
				<span style="line-height: 30px;color: #AAA;font-size: 14px;"> 电影/电视剧/影人</span>
			</div>
			<div class="mui-scroll-wrapper" style="top: 50px;">
				<div class="mui-scroll">
					<ul  id="movies" class="mui-table-view">
						<li class="mui-table-view-cell" v-for="item in movies">
							<img class="mui-pull-left item-img" :src="item.cover">
							<div class="mui-ellipsis dark-big">
								{{item.title}}
							</div>
							<div class="mui-ellipsis ">
								<span class="gray-small">{{item.genres}}</span>&nbsp;
								<span v-if="item.score>0" class="orange-small">{{item.score}}</span>
								<span v-else class="orange-small">暂无评分</span>
							</div>
							<div class="mui-ellipsis ">
								<span class="gray-small">导演：</span>&nbsp;
								<span class="gray-small">{{item.directors}}</span>
							</div>
							<div class="mui-ellipsis ">
								<span class="gray-small">主演：</span>&nbsp;
								<span class="gray-small">{{item.casts}}</span>
							</div>
						</li>
					</ul>
				</div>
			</div>
		</div>
		<script src="js/util.js"></script>
		<script src="js/vue.min.js"></script>
		<script type="text/javascript">
			//Vue数据
			var data_movies = new Vue({
				el:'#movies',
				data:{
					movies:[]
				}
			})
			
			
			
			//mui初始化
			mui.init();
			//mui底部窗口组件
			mui.plusReady(function() {
				var self = plus.webview.currentWebview(),
					nviews = self.getSubNViews(),
					subpages = util.options.subpages;
				//创建子webview窗口 并初始化
				util.initSubpage();
				var activePage = plus.webview.currentWebview();

				//给每个view 添加监听点击切换
				for (var i = 0; i < 3; i++) {
					nviews[i].addEventListener('click', clickEvent, false);
				}

				// 自定义 tab 点击事件
				function clickEvent(e) {
					var currId = e.target.id,
						currIndex = parseInt(currId.substr(currId.length - 1, 1) - 1),
						currView = self.getStyle().subNViews[currIndex];

					// 匹配对应tab窗口	
					if (currIndex > 0) {
						targetPage = plus.webview.getWebviewById(subpages[currIndex - 1]);
					} else {
						targetPage = plus.webview.currentWebview();
					}

					if (targetPage == activePage) {
						return;
					}

					if (currIndex !== 3) {
						//底部选项卡切换
						util.toggleNview(currView, currIndex);
						// 子页面切换
						util.changeSubpage(targetPage, activePage);
						//更改当前活跃的页面
						activePage = targetPage;
					}
				}
			});

			//给搜索框添加点击事件
			mui('.wrap-search')[0].addEventListener('tap', function() {
				mui.openWindow({
					id: 'search',
					url: './html/search.html'
				});
			});
			mui('.mui-scroll-wrapper').scroll({
				indicators: false
			});
			
			mui.getJSON('https://api.couban.com/v2/movie/in_theaters', {}, function(resp) {
				data_movies.movies = convert(resp.subjects);
				console.log(newMovies.length)
			});
			//将返回来的接口数据,进行重新组合成自己需要的.
			function convert(items) {
				var newItems = [];
				//遍历items
				items.forEach(function(item) {
					var genres = item.genres.toString().replace(/,/g, "/");
					//导演
					var directors = '';
					for (var i = 0; i < item.directors.length; i++) {
						directors += item.directors[i].name;
						if (i != item.directors.lenth - 1) {
							directors += '/';
						}
					}
					//演员
					var casts = '';
					for (var i = 0; i < item.casts.length; i++) {
						casts += item.casts[i].name;
						if (i != item.casts.lenth - 1) {
							casts += '/';
						}
					}
					newItems.push({
						id: item.id,
						title: item.title,
						genres: genres,
						cover: item.images.large,
						score: item.rating.average,
						directors: directors,
						casts: casts
					});
				});
				return newItems;
			}
		</script>
	</body>

</html>

```

# mui实现下拉刷新功能

### 实现思路：使用mui.init()中的pullRefresh实现下拉刷新和上拉加载

```javascript
操作示例（官方示例）：
mui.init({
  pullRefresh : {
    container:"#refreshContainer",//下拉刷新容器标识，querySelector能定位的css选择器均可，比如：id、.class等
    down : {
      style:'circle',//必选，下拉刷新样式，目前支持原生5+ ‘circle’ 样式
      color:'#2BD009', //可选，默认“#2BD009” 下拉刷新控件颜色
      height:'50px',//可选,默认50px.下拉刷新控件的高度,
      range:'100px', //可选 默认100px,控件可下拉拖拽的范围
      offset:'0px', //可选 默认0px,下拉刷新控件的起始位置
      auto: true,//可选,默认false.首次加载自动上拉刷新一次
      callback :pullfresh-function //必选，刷新函数，根据具体业务来编写，比如通过ajax从服务器获取新数据；
    }
  }
});
```

```javascript
//container要给页面的需要刷新和加载的地方的标签加上id属性
mui.init({
				swipeBack: true, //启用右滑关闭功能
				pullRefresh: {
					container: "#refreshContainer", //下拉刷新容器标识，querySelector能定位的css选择器均可，比如：id、.class等
					down: {
						auto: false, //可选,默认false.首次加载自动下拉刷新一次
						callback: refreshData //必选，刷新函数，根据具体业务来编写，比如通过ajax从服务器获取新数据；
					},
					up: {
						height: 50, //可选.默认50.触发上拉加载拖动距离
						auto: true, //可选,默认false.自动上拉加载一次
						contentrefresh: "正在加载...", //可选，正在加载状态时，上拉加载控件上显示的标题内容
						contentnomore: '没有更多数据了', //可选，请求完毕若没有更多数据时显示的提醒内容；
						callback: loadMoreData //必选，刷新函数，根据具体业务来编写，比如通过ajax从服务器获取新数据；
					}
				}
			});



//刷新数据，重新调用接口
			function refreshData() {
				//请求热映列表接口
				mui.getJSON('https://api.douban.com/v2/movie/in_theaters', {
					start:0,
					count:10
				}, function(resp) {
					data_movies.movies.splice(0,data_movies.movies.length);
					data_movies.movies = data_movies.movies.concat(convert(resp.subjects));
					mui('#refreshContainer').pullRefresh().endPulldownToRefresh();
					mui('#refreshContainer').pullRefresh().refresh(true);
				});
			}

//请求下一页数据
function loadMoreData(){//请求热映列表接口
mui.getJSON('https://api.douban.com/v2/movie/in_theaters', {
					start:data_movies.movies.length,
					count:10
				}, function(resp) {
					data_movies.movies = data_movies.movies.concat(convert(resp.subjects));
					mui('#refreshContainer').pullRefresh().endPullupToRefresh(data_movies.movies.length > resp.total);
				});
			}

```

# 预加载

## 实现思路：使用mui.preload实现预加载

```html
使用mui.preload实现预加载能立即返回对应的webview的引用
```

# 电影详情页面跳转和数据传递

## 学习目标

`1.从热映列表页跳转至电影详情页`

`2.从热映列表页传递参数至电影详情页`

### 实现思路

1. 使用Mui.openWindow打开详情页面
2. 使用自定义事件实现参数传递

```javascript
通过mui.fire()方法可以触发目标窗口的自定义事件

mui.fire(目标窗口的webview,'自定义事件名',{参数列表});

目标窗口监听这个自定义事件

window.addEventListener('自定义事件名', function() {
XXXXX
}, false);

```

```javascript
			操作实例
//添加movieId自定义事件监听
window.addEventListener('moveId',function(event){
    //获取时间参数
    var id = event.detaile.id;
})
//触发详情页面的movieId事件
mui.fire(detailPage,'movieId',{
    id:item.id
})

```

在详情页面要添加一个movieID的自定义事件监听

```
window.addEventListener('moveId',function(event){
    //获取时间参数
    var id = event.detaile.id;
})
```

目录页面

```
//预加载电影想情页面
			var detailPage =  mui.preload({
				id:'movie-detail',
				url:'./html/movie-detail.html'
			})
			
			//打开电影详情页面
			function open_detail(item){
				//触发详情页面的movieId事件
				mui.fire(detailPage,'movieId',{
					id:item.id
				})
				//打开
				mui.openWindow({
					id:'movie-detail'
				});
			}
```

使用预加载可以通过自定义事件传递参数

没有使用预加载页面可以通过extras这些参数

css

```css
vertical-align 属性设置元素的垂直对齐方式。
该属性定义行内元素的基线相对于该元素所在行的基线的垂直对齐。允许指定负长度值和百分比值。这会使元素降低而不是升高。在表单元格中，这个属性会设置单元格框中的单元格内容的对齐方式。
white-space 规定段落中的文本不进行换行;
text-overflow: ellipsis;显示省略符号来代表被修剪的文本。	
```

# 详情页面获取接口数据

```<script>
<script>		
        mui.init()
			//默认数据
			function getDefaultData() {
				return {
					title: '',
					cover '',
					score: '',
					ratingCount: '',
					summary: '',
					countries: '',
					year: '',
					genres: '',
					casts: [],
					directors: []
				}
			}
			//vue 对象
			var data_detail = new Vue({
				el: '#content',
				data: getDefaultData(),
				methods: {
					resetData: function() { //重制数据
						Object.assign(this.data, getDefaultData());
					},
					open_detail: function(item) {
						//打开演员详情页面
					}
				}
			})
			
			
			mui.plusReady(function(){
				var self = plus.webview.currentWebview();
				//添加hide事件,清空页面数据,滚到最顶部
				self.addEventListener('hid',function(){
					window.scrollTo(0,0);
					data_detail.resetData();
				},false)
			})
			
			//添加自定义事件
			window.addEventListener("movieId", function(event) {
				var id = event.detail.id
				plus.nativeUI.showWaiting('加载中', {
					width: 100 px,
					height: 100 px
				});
				mui.getJSON('http://api.douban.com/v2/movie/subject/' + id, function(resp) {
					data_detail.title = resp.title;
					data_detail.cover = resp.images.large;
					data_detail.score = resp.rating.average;
					data_detail.ratingCount = resp.ratings_count;
					data_detail.summary = resp.summary;
					data_detail.countries = resp.countries.toString('/,/g', ' / ');
					data_detail.year = resp.year;
					data_detail.genres = resp.genres.toString('/,/g', ' / ');
					data_detail.casts = resp.casts;
					data_detail.directors = resp.directors;
					plus.nativeUI.closeWaiting();
				})
			})
		</script>
```

//演员详情页面，

首先先给影片详情页面的 演员标签那里，注册个Vue的点击事件，@click="open_detail",这个方法，vue的实例对象中有

```
var data_detail = new Vue({
				el: '#content',
				data: getDefaultData(),
				methods: {
					resetData: function() { //重制数据
						Object.assign(this.data, getDefaultData());
					},
					open_detail: function(item) {
						//打开演员详情页面
						//非预加载可以通过extras传递参数
						mui.openWindow({
							id:'cast-detail',
							url:'./cast-detail.html',
							extras:{
								castId:item.id
							}
						})
					}
				}
			})
```

在演员详情页面去接受影片详情传递过来的演员id

```

mui.plusReady(function() {
	var self = plue.webview.currentWebview();
	
}）
```

```html
<div style="overflow-x:scroll; white-space:nowrap;padding-left:15px ">
			<div style="width: 150px;height: 50px;border: 1px solid #ccc; display:inline-block"></div>
			<div style="width: 150px;height: 50px;border: 1px solid #ccc; display:inline-block"></div>
</div>
```

```javascript
JavaScript forEach() 方法  列出数组的每个元素：
forEach() 方法用于调用数组的每个元素，并将元素传递给回调函数。
注意: forEach() 对于空数组是不会执行回调函数的。
array.forEach(function(currentValue, index, arr), thisValue)
currentValue	必需。当前元素
index	可选。当前元素的索引值。
arr	可选。当前元素所属的数组对象。
/************************************/
replace() 方法用于在字符串中用一些字符替换另一些字符，或替换一个与正则表达式匹配的子串。
stringObject.replace(regexp/substr,replacement)
regexp/substr	
必需。规定子字符串或要替换的模式的 RegExp 对象。

请注意，如果该值是一个字符串，则将它作为要检索的直接量文本模式，而不是首先被转换为 RegExp 对象。

replacement	必需。一个字符串值。规定了替换文本或生成替换文本的函数。
```

# 搜索历史记录实现

- 通过plus.storage实现本地数据存储
- 通过plus.storage.setItem 实现存
- 通过plus.storage.getItem 实现取