<template>
	<div id="app">
		<Carousel :slides="slides">
			<template v-slot="slide">
				<div class="slide-content">
					{{ slide.id }}
				</div>
			</template>
		</Carousel>
	</div>
</template>

<script>
import Carousel from '@/components/Carousel.vue';

export default {
	name: 'app',
	components: {
		Carousel
	},
	data() {
		return {
			slideCount: 20,
			slides: []
		}
	},
	created() {
		for(var i = 1; i <= this.slideCount; i++ ) {
			this.slides.push({id: i});
		}
	}
}
</script>

<style scoped>
.slide-content {
	display: flex;
	justify-content: center;
	align-items: center;
	height: 500px;
	background-color: #f5f5f5;
	border: 1px solid #cccccc;
	font-size: 50px;
}

</style>
