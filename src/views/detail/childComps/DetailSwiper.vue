<template>
	<swiper v-if="topImages.length" class="detail-swiper">
		<swiper-item v-for="item in topImages">
			<img :src="item" alt="" @load="imgLoad">
		</swiper-item>
	</swiper>
</template>

<script>
	import {Swiper,SwiperItem} from 'components/common/swiper'
	export default {
		name:'DetailSwiper',
		props:{
			topImages:{
				type:Array,
				default() {
					return []
				}
			}
		},
		components:{
			Swiper,
			SwiperItem
		},
		methods:{
			imgLoad(){
				
			}
		}
	}
</script>

<style scoped>
	.detail-swiper {
		overflow: hidden;
		height: 300px;
	}
</style>