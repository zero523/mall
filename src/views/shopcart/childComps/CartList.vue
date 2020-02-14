<template>
	<div class="cart-list">
		<scroll class="content" ref="scroll">
			<cart-list-item v-for="item in cartList" :key="item.iid" :item-info="item"/>
		</scroll>
	</div>
</template>

<script>
	import CartListItem from './CartListItem'

	import Scroll from 'components/common/scroll/Scroll'

	import {mapGetters} from 'vuex'
	export default {
		name:'CartList',
		components:{
			Scroll,
			CartListItem
		},
		computed:{
			...mapGetters(['cartList'])
		},
		activated(){
			this.$refs.scroll.refresh()
      // console.log(this.$refs.scroll.scroll);
			// 
		}
	}
</script>

<style scoped>
	.cart-list{
  	/*height: calc(100% - 143px);*/
		position: fixed;
		top: 44px;
		bottom: 89px;
		left: 0;
		right: 0;
	}
	.content{
		height: 100%;
	}
	
</style>