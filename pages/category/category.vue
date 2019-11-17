<template>
	<view class="content">
		<scroll-view scroll-y class="left-aside">
			<view v-for="item in parentType" :key="item.id" class="f-item b-b" :class="{active: item.id === currentId}" @click="tabtap(item)">
				{{item.pname}}
			</view>
		</scroll-view>
		<!-- 根据tabScrollTop设置滚屏位置 -->
		<scroll-view scroll-with-animation scroll-y class="right-aside" @scroll="asideScroll" :scroll-top="tabScrollTop">
			<!-- 为每个分类框设定唯一标识 -->
			<view v-for="item in parentType" :key="item.id" class="s-list" :id="'main-'+item.id">
				<text class="s-item">{{item.pname}}</text>
				<view class="t-list">
					<view @click="navToList(item.id, titem.id)" v-if="titem.parentId === item.id" class="t-item" v-for="titem in goodType" :key="titem.id">
						<image :src="titem.imgSrc"></image>
						<text>{{titem.cname}}</text>
					</view>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				sizeCalcState: false,//一个状态
				tabScrollTop: 0,
				currentId: 1,
				parentType:[],
				goodType:[]
			}
		},
		onLoad(){
			this.show()
		},
		methods: {
			
			//左侧栏点击
			tabtap(item){
				if(!this.sizeCalcState){//状态改变
					this.calcSize();//计算右侧栏每个tab的高度等信息
				}
				
				this.currentId = item.id;//通过id
				let index = this.parentType.findIndex(pitem=>pitem.id === item.id);//右侧栏父类型与左侧类型联动
				this.tabScrollTop = this.parentType[index].top;//设置匹配元素的滚动条的垂直位置
			},
			//右侧栏滚动
			asideScroll(e){
				if(!this.sizeCalcState){//改变状态
					this.calcSize();//计算右侧栏每个tab的高度等信息
				}
				let scrollTop = e.detail.scrollTop;
				let tabs = this.parentType.filter(item=>item.top <= scrollTop).reverse();
				if(tabs.length > 0){
					this.currentId = tabs[0].id;
				}
			},
			//计算右侧栏每个tab的高度等信息
			calcSize(){
				let h = 0;
				this.parentType.forEach(item=>{
					let view = uni.createSelectorQuery().select("#main-" + item.id);
					view.fields({
						size: true
					}, data => {
						item.top = h;
						h += data.height;
						item.bottom = h;
					}).exec();
				})
				this.sizeCalcState = true;
			},
			navToList(sid, tid){
				uni.navigateTo({
					url: `/pages/product/list?fid=${this.currentId}&sid=${sid}&tid=${tid}`
				})
			},
			// 从后台获取数据
			show(){
				uni.request({
				    url: 'http://127.0.0.1:8001/goods/showType',
				    success: (res) => {
						this.parentType = res.data.data2
						this.goodType = res.data.data
				    }
				});
			},
			
		}
	}
</script>

<style lang='scss'>
	page,
	.content {
		height: 100%;
		background-color: #f8f8f8;
	}

	.content {
		display: flex;
	}
	.left-aside {
		flex-shrink: 0;
		width: 200upx;
		height: 100%;
		background-color: #fff;
	}
	.f-item {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 100upx;
		font-size: 28upx;
		color: $font-color-base;
		position: relative;
		&.active{
			color: $base-color;
			background: #f8f8f8;
			&:before{
				content: '';
				position: absolute;
				left: 0;
				top: 50%;
				transform: translateY(-50%);
				height: 36upx;
				width: 8upx;
				background-color: $base-color;
				border-radius: 0 4px 4px 0;
				opacity: .8;
			}
		}
	}

	.right-aside{
		flex: 1;
		overflow: hidden;
		padding-left: 20upx;
	}
	.s-item{
		display: flex;
		align-items: center;
		height: 70upx;
		padding-top: 8upx;
		font-size: 28upx;
		color: $font-color-dark;
	}
	.t-list{
		display: flex;
		flex-wrap: wrap;
		width: 100%;
		background: #fff;
		padding-top: 12upx;
		&:after{
			content: '';
			flex: 99;
			height: 0;
		}
	}
	.t-item{
		flex-shrink: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		width: 176upx;
		font-size: 26upx;
		color: #666;
		padding-bottom: 20upx;
		
		image{
			width: 100upx;
			height: 100upx;
		}
	}
</style>
