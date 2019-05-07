<p align="center">
    <!-- <img alt="logo" src="" width="120" style="margin-bottom: 10px;"> -->
</p>
<h3 align="center" style="margin: 30px 0 35px;">uni-app 下拉刷新上拉加载</h3>

---


## 安装

#### NPM

```shell
npm i uni-scroll -S
```

#### YARN

```shell
yarn add uni-scroll
```


## 快速上手
> 使用方法可以参考 mescroll-uni * [具体查看](http://www.mescroll.com/uni.html?v=190426)

#### 方式一. 直接传输

```js
<template>
	<view>
		<!--mescroll滚动区域的基本结构-->
		<mescroll-uni :down="downOption" @down="downCallback" :up="upOption" @up="upCallback" @init="mescrollInit" class="mescroll-box">
			<!--内容...-->
			<view v-for="(item,idx) in dataList" :key="idx">
				<view class="cu-card article" style="height: 200upx; ">
						{{idx}}
				</view>
			</view>
		</mescroll-uni>
		
	</view>
</template>

<script>
	import MescrollUni from "mescroll-uni/mescroll-uni.vue";
	export default {
		components: {
			MescrollUni // 注册mescroll组件
		},
		data() {
			return {
				mescroll: null, //mescroll实例对象
				// 下拉刷新的配置
				downOption: {
					use: true, // 是否启用下拉刷新; 默认true
					auto: true, // 是否在初始化完毕之后自动执行下拉刷新的回调; 默认true
				},
				// 上拉加载的配置
				upOption: {
					use: true, // 是否启用上拉加载; 默认true
					auto: true, // 是否在初始化完毕之后自动执行上拉加载的回调; 默认true
					isLock: false, //是否锁定上拉加载(设为true,可用于不触发upCallback,只保留回到顶部按钮的场景)
					page: {
						num: 0, // 当前页码,默认0,回调之前会加1,即callback(page)会从1开始
						size: 10 // 每页数据的数量,默认10
					},
					noMoreSize: 3, // 配置列表的总数量要大于等于5条才显示'-- END --'的提示
					empty: {
						tip: '空空如也 ~',
						icon: null,
					}
				},
				// 列表数据
				dataList: []
			}
		},
		// 必须注册滚动到底部的事件,使上拉加载生效
		onReachBottom() {
			this.mescroll && this.mescroll.onReachBottom();
		},
		// 必须注册列表滚动事件,使下拉刷新生效
		onPageScroll(e) {
			this.mescroll && this.mescroll.onPageScroll(e);
		},
		methods: {
			// mescroll组件初始化的回调,可获取到mescroll对象
			mescrollInit(mescroll) {
				this.mescroll = mescroll;
			},
			/*下拉刷新的回调, 有三种处理方式: */
			downCallback(mescroll) {
				// 第1种: 请求具体接口
				// uni.request({
				// 	url: 'xxxx',
				// 	success: () => {
				// 		// 成功隐藏下拉加载状态
				// 		mescroll.endSuccess()
				// 	},
				// 	fail: () => {
				// 		// 失败隐藏下拉加载状态
				// 		mescroll.endErr()
				// 	}
				// })
				// 第2种: 下拉刷新和上拉加载调同样的接口, 那以上请求可删, 直接用mescroll.resetUpScroll()代替
				mescroll.resetUpScroll(); // 重置列表为第一页 (自动执行 page.num=1, 再触发upCallback方法 )
				// 第3种: 下拉刷新什么也不处理, 可直接调用或者延时一会调用 mescroll.endSuccess() 结束即可
				// mescroll.endSuccess()
			},
			/*上拉加载的回调*/
			upCallback(mescroll) {
				// 此时mescroll会携带page的参数:
				let pageNum = mescroll.num; // 页码, 默认从1开始
				let pageSize = mescroll.size; // 页长, 默认每页10条
				setTimeout(()=>{
					let totalSize = 50
					let curPageData = [1, 2, 3, 4, 5, 6, 7, 8,9,10]
					// curPageData = []
					
					//方法二(推荐): 后台接口有返回列表的总数据量 totalSize
					mescroll.endBySize(curPageData.length, totalSize); 
					
					//设置列表数据
					if (mescroll.num == 1) this.dataList = []; //如果是第一页需手动制空列表
					this.dataList = this.dataList.concat(curPageData); //追加新数据
				},2000)
				// test
				
				return 
				uni.request({
					url: 'xxxx?pageNum=' + pageNum + '&pageSize=' + pageSize,
					success: (data) => {
						// 接口返回的当前页数据列表 (数组)
						let curPageData = data.xxx;
						// 接口返回的总页数 (比如列表有26个数据,每页10条,共3页; 则totalPage值为3)
						let totalPage = data.xxx;
						// 接口返回的总数据量(比如列表有26个数据,每页10条,共3页; 则totalSize值为26)
						let totalSize = data.xxx;
						// 接口返回的是否有下一页 (true/false)
						let hasNext = data.xxx;

						// 成功隐藏下拉加载状态
						//方法一(推荐): 后台接口有返回列表的总页数 totalPage
						mescroll.endByPage(curPageData.length, totalPage);

						//方法二(推荐): 后台接口有返回列表的总数据量 totalSize
						//mescroll.endBySize(curPageData.length, totalSize); 

						//设置列表数据
						if (mescroll.num == 1) this.dataList = []; //如果是第一页需手动制空列表
						this.dataList = this.dataList.concat(curPageData); //追加新数据
					},
					fail: () => {
						// 失败隐藏下拉加载状态
						mescroll.endErr()
					}
				})
			}
		},
	}
</script>
<style scope>
	.mescroll-box{
		height: auto;
	}
</style>

```





## 贡献代码

使用过程中发现任何问题都可以提 [Issue](https://github.com/13725102796/uni-scroll/issues) 



## 开源协议

本项目基于 [MIT](https://zh.wikipedia.org/wiki/MIT%E8%A8%B1%E5%8F%AF%E8%AD%89) 协议，请自由地享受和参与开源。