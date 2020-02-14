<template>
	<div id="detail">
		<detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
		<scroll class="content" ref="scroll"
		:probe-type="3"
		@scroll="contentScroll"
		>
			<detail-swiper :top-images="topImages"></detail-swiper>
			<detail-base-info :goods="goods"/>
			<detail-shop-info :shop="shop"/>
			<detail-goods-info :detail-info="detailInfo" @detailImageLoad="detailImageLoad"/>
			<detail-param-info :param-info="paramInfo" ref="params"/>
			<detail-comment-info :comment-info="commentInfo" ref="comments"/>
			<goods-list :goods="recommends" ref="recommends"/>
		</scroll>
		<back-top v-show="isShowBackTop" @click.native="backClick"></back-top>
		<detail-bottom-bar @addToCart="addToCart"></detail-bottom-bar>

	</div>
</template>

<script>
	import DetailNavBar from './childComps/DetailNavBar'
	import DetailSwiper from './childComps/DetailSwiper'
	import DetailBaseInfo from './childComps/DetailBaseInfo'
	import DetailShopInfo from './childComps/DetailShopInfo'
	import DetailGoodsInfo from './childComps/DetailGoodsInfo'
	import DetailParamInfo from './childComps/DetailParamInfo'
	import DetailCommentInfo from './childComps/DetailCommentInfo'
	import DetailBottomBar from './childComps/DetailBottomBar'

	import Scroll from 'components/common/scroll/Scroll'
	import BackTop from 'components/content/backTop/BackTop'
	import GoodsList from 'components/content/goods/GoodsList'

	import {debounce} from 'common/utils'
	import {getDetail,getRecommend,
		Goods,
		Shop,
		GoodsParam} from 'network/detail'
	
	// import {itemListenerMixin} from 'common/mixin'
	import {mapActions} from 'vuex'


	export default {
		name:'Detail',
		data(){
			return {
				iid:null,
				topImages:[],
				goods:{},
				shop:{},
				detailInfo:{},
				paramInfo:{},
				commentInfo:{},
				recommends:[],
				isShowBackTop:false,
				itemImageListener:null,
				themeTopYs:[0]
			}
		},
		// mixin:[itemListenerMixin],
		components:{
			DetailNavBar,
			DetailSwiper,
			DetailBaseInfo,
			DetailShopInfo,
			DetailGoodsInfo,
			DetailParamInfo,
			DetailCommentInfo,
			DetailBottomBar,
			Scroll,
			BackTop,
			GoodsList
		},
		created(){
			this.iid=this.$route.params.iid;
			this.getDetail(this.iid)
			this.getRecommend()

			// 给getThemeTopY赋值
			this.getThemeTopY= debounce (() => {
				this.themeTopYs=[];
			  this.themeTopYs.push(0);
			  this.themeTopYs.push(-this.$refs.params.$el.offsetTop);
			  this.themeTopYs.push(-this.$refs.comments.$el.offsetTop);
			  this.themeTopYs.push(-this.$refs.recommends.$el.offsetTop);
			  this.themeTopYs.push(-Number.MAX_SAFE_INTEGER);
			  // console.log(this.themeTopYs)
			},100)
		},
		mounted(){
			//1.图片加载事件监听
	    const refresh = debounce(this.$refs.scroll.refresh, 400)

	    //3.监听item中图片加载完成
	    this.itemImageListener = 
		  this.$bus.$on('itemImageLoad',() => {
		    refresh();
		  })
		  
		},
		destroyed(){
			// 取消全局事件
			this.$bus.$off('itemImageLoad',this.itemImageListener)
		},
		methods:{ 
			...mapActions(['addCart']),
			detailImageLoad(){
      	// //1.图片加载事件监听
	      const refresh = debounce(this.$refs.scroll.refresh, 400)

	      // navbar切换与滚动内容绑定
	      this.getThemeTopY();
			},
			backClick(){
				this.$refs.scroll.scrollTo(0,0,500)
			},
			titleClick(index){
				this.$refs.scroll.scrollTo(0 ,this.themeTopYs[index],500)
			},
			contentScroll(position){
				this.isShowBackTop = -position.y>1000
				// 1.获取y值
				const positionY=-position.y;
				// 2.将positionY的值和themeTopY的值对比
				let length=this.themeTopYs.length;
				// for(let i = 0; i < length; i++){
	      //   if(this.currentIndex !== i && ((i < length - 1 && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i + 1]) || (i === length - 1 && positionY >=  this.themeTopYs[i]))){
	      //     this.currentIndex = i;
	      //     this.$refs.nav.currentIndex = this.currentIndex;
	      //   }
	      // }
	      for(let i=0;i<length-1;i++){
	      	if(this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])){
	      		this.currentIndex=i;
	      		this.$refs.nav.currentIndex=this.currentIndex
	      	}
	      }
			},
			addToCart(){
				console.log(this.goods)
				// 获取购物车需要展示的信息
				const product={}
				product.image=this.topImages[0];
				product.title=this.goods.title;
				product.desc=this.goods.desc;
				product.price=this.goods.realPrice;
				product.iid=this.iid;
				// this.$store.state.cartList.push(product)
				// //mutation的提交
	      //this.$store.commit('addCart', product);
	      //action的提交
	      //this.$store.dispatch('addCart', product).then({
	      //   console.log(res);
	      // })
	      
	      // promise store/actions.js
				// this.$store.dispatch('addCart',product).then(res=>{
				// 	// 提示弹窗 toast
				// 	console.log(res)
				// })
				// mapActions
				this.addCart(product).then(res => {
						this.$toast.show(res)
				})

			},
			/*
			 *网络请求
			 */
			getDetail(iid){
				getDetail(iid).then(res=>{
					// 获取轮播信息
					const data=res.result;
					this.topImages=data.itemInfo.topImages;
					// 获取商品信息
					this.goods=new Goods(data.itemInfo,data.columns,data.shopInfo)
					// 获取店铺信息
					this.shop=new Shop(data.shopInfo)
					// 商品详情
					this.detailInfo=data.detailInfo;
					// 获取商品尺寸参数。。。
					this.paramInfo=new GoodsParam(data.itemParams.info,data.itemParams.rule)
					// 获取评论信息
					if(data.rate.cRate!==0){
						this.commentInfo=data.rate.list[0]
					}

				})
			},
			getRecommend(){
				getRecommend().then(res=>{
					this.recommends=res.data.list
					// console.log(this.recommends)
				})
			}
		}
	}
</script>

<style scoped>
	#detail{
		position: relative;
		z-index: 10;
		background: #fff;
		height:100vh;
	}
	.detail-nav{
		position: relative;
		z-index: 10;
		background: #fff;
	}
	.content{
		position: absolute;
		top: 44px;
		bottom: 58px;
	}
	.back-top{
		position: fixed;
		right: 10px;
		bottom:65px;
	}
</style>