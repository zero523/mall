<template>
	<div id="shopcart">
		<nav-bar class="nav-bar">
			<div slot="center">购物车({{cartCount}})</div>
		</nav-bar>
		<cart-list/>
		<cart-bottom-bar/>
	</div>
</template>

<script>
	import CartList from './childComps/CartList'
	import CartBottomBar from './childComps/CartBottomBar'

	import NavBar from 'components/common/navbar/NavBar'
	import {mapGetters} from 'vuex'
	export default {
		name: 'Shopcart',
		components:{
			NavBar,
			CartList,
			CartBottomBar
		},
		computed:{
			// mapGetters的两种用法 对象键值对重命名
			...mapGetters(['cartCount'])
		}
	}
</script>

<style scoped>
	.nav-bar{
		background: var(--color-tint);
		color: #fff;
	}
</style>