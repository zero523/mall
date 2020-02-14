<template>
	<div id="home"> 
		<nav-bar class="home-nav">
			<div slot="center">购物街</div>
		</nav-bar>
		<tab-control class="tab-control"
			:titles="['流行','新款','精选']"
			@tabClick="tabClick"
			ref="tabControl1"
			v-show="isTabFixed">
			</tab-control>
		<scroll class="content" ref="scroll"
			:probe-type="3"
			:pull-up-load="true"
			@scroll="contentScroll"
			@pullingUp="loadMore">
			<home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
			<recommend-view :recommends="recommends"></recommend-view>
			<feature-view/>
			<tab-control class="tab-control"
			:titles="['流行','新款','精选']"
			ref="tabControl2"
			@tabClick="tabClick">
			</tab-control>
			<goods-list :goods="showGoods"></goods-list>
		</scroll>
		<back-top  @click.native="backClick" v-show="isShowBackTop"></back-top>
	</div>
</template>

<script>
	import HomeSwiper from './childComps/HomeSwiper';
	import RecommendView from './childComps/RecommendView';
	import FeatureView from './childComps/FeatureView';

	import NavBar from 'components/common/navbar/NavBar';
	import TabControl from 'components/content/tabControl/TabControl'
	import GoodsList from 'components/content/goods/GoodsList';
	import Scroll from 'components/common/scroll/Scroll';
	import BackTop from 'components/content/backTop/BackTop';

	import {getHomeMultidata,getHomeGoods} from 'network/home';

	import {itemListenerMixin} from 'common/mixin'

	export default {
		name: 'Home',
		data() {
			return {
				banners:[],
				recommends:[],
				goods:{
					'pop':{page:0,list:[]},
					'new':{page:0,list:[]},
					'sell':{page:0,list:[]}
				},
				currentType:'pop',
				tabOffsetTop:0,
				isTabFixed:false,
				saveY:0,
				isShowBackTop:false
			}
		},
	  mixin:[itemListenerMixin],
		components:{
			HomeSwiper,
			RecommendView,
			FeatureView,
			NavBar,
			TabControl,
			GoodsList,
			Scroll,
			BackTop
		},
		computed:{
			showGoods(){
				return this.goods[this.currentType].list
			}
		},
		created() {
			this.getHomeMultidata(); 
			this.getHomeGoods('pop')
			this.getHomeGoods('new')
			this.getHomeGoods('sell')
		},
	  mounted(){
	    // //1.图片加载事件监听
	    // const refresh = debounce(this.$refs.scroll.refresh, 400)

	    // //3.监听item中图片加载完成
	   //  this.itemImageListener = () => {
		  //   refresh();
		  // }
		  // this.$bus.$on('itemImageLoad',this.itemImageListener)
		  // console.log('mix')
	  },
		activated(){
			//再次进入时滚动到保存位置
			this.$refs.scroll.scrollTo(0,this.saveY,0)
			this.$refs.scroll.refresh()
		},
		deactivated(){
			// 保存位置
			this.saveY=this.$refs.scroll.getScrollY();
			// 取消全局事件的监听
			this.$bus.$off('itemImageLoad',this.itemImageListener)
		},
		methods:{
			/**
			 * 事件操作
			 */
			tabClick(index){
				switch(index){
					case 0:
						this.currentType='pop'
						break;
					case 1:
						this.currentType='new'
						break;
					case 2:
						this.currentType='sell'
				}
				this.$refs.tabControl1.currentIndex=index
				this.$refs.tabControl2.currentIndex=index
			},
			backClick(){
				this.$refs.scroll.scrollTo(0,0,500)
			},
			contentScroll(position){
				//1.判断BackTop是否显示
				this.isShowBackTop=(-position.y)>1000
				//2.决定tabControl是否吸顶
				this.isTabFixed= (-position.y) > this.tabOffsetTop
			},
			loadMore(){
				this.getHomeGoods(this.currentType);
				this.$refs.scroll.refresh()
				console.log(this.goods)
			},
			swiperImageLoad(){
				this.tabOffsetTop=this.$refs.tabControl2.$el.offsetTop
			},
			getScrollY(){
				return this.scroll.y ? this.scroll.y : 0
			},
			/**
			 *网络请求
			 */
			getHomeMultidata(){
				getHomeMultidata().then(res => {
					this.banners=res.data.banner.list
					this.recommends=res.data.recommend.list
				})
			},
			getHomeGoods(type){
				const page=this.goods[type].page + 1;
				getHomeGoods(type,page).then(res => {
        	this.goods[type].list.push(...res.data.list);
					this.goods[type].page +=1;
					console.log(this.goods['pop'])
				})
			},
		}
	}
</script>

<style scoped>
	#home{
		padding-top: 44px;
		height:100vh;
	}
	.home-nav{
		background:var(--color-tint);
		color:#fff;
		box-shadow:0 1px 1px rgba(0,0,0,.1);
		position: fixed;
		top: 0;
		left:0;
		right: 0;
		z-index:3;
	}
	.tab-control{
		/*position:sticky;*/
		position: relative;
		z-index: 2;
	}
	.content{
		/*height:calc(100%-98px);
		overflow: hidden;
		margin-top: 44px;*/
		position: absolute;
		top:44px;
		bottom: 49px;
	}
</style>