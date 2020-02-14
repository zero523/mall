<template>
	<div>
		<nav-bar>
			<div slot="left" class="back" @click="backClick()">
				<img src="~assets/img/common/back.svg" alt="">
			</div>
			<div slot="center" class="title">
				<div class="title-item"
				v-for="(item,index) in titles"
				@click="titleClick(index)"
				:class="{active:index===currentIndex}">{{item}}</div>
			</div>
		</nav-bar>
	</div>
</template>

<script>
	import NavBar from 'components/common/navbar/NavBar'
	export default {
		name:'Detail',
		components:{
			NavBar
		},
		data(){
			return{
				titles:['商品','参数','评论','推荐'],
				currentIndex:0
			}
		},
		methods:{
			titleClick(index){
				this.currentIndex=index;
				this.$emit('titleClick',index)
			},
			backClick(){
				this.$router.back()
			}
		}
	}
</script>

<style scoped>
	.back img{
		margin-top: 10px;
	}
	.title{
		display: flex;
	}
	.title-item{
		flex:1;
	}
	.active{
		color: var(--color-high-text)
	}
</style>