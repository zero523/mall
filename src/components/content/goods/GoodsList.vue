<template>
	<div class="goods-list">
		<goods-list-item v-for="item in goods" :product="item" :key="item.id"/>
	</div>
</template>

<script>
	import GoodsListItem from 'components/content/goods/GoodsListItem'
	export default {
		name:'GoodsList',
		props:{
			goods:{
				type:Array,
				default(){
					return []
				}
			}
		},
		components:{
			GoodsListItem
		},
		methods:{
			itemClick(){
				// this.$router.push('/detail');
			}
		}
	}
</script>

<style scoped>
	.goods-list{
		display: flex;
		flex-wrap:wrap;
		justify-content: space-around;
		padding: 2px;
	}
</style>