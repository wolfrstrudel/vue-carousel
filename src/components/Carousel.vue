<template>
	<div ref="theCarousel" class="carousel">
		<div :style="wrapperStyle" class="carousel-wrapper">
			<div v-for="(slide, index) in activeSlides" :key="index" class="slide">
				<div class="slide-wrapper">
					{{ slide }}
				</div>
			</div>
		</div>

		<div class="pagination-button pagination-button-previous" @click="advance(-1)">
			Previous
		</div>

		<div class="pagination-button pagination-button-next" @click="advance(1)">
			Next
		</div>
	</div>
</template>

<script>

export default {
	name: "Carousel",
	props: {
		slidesPerPage: {
			type: Number,
			default: 3
		},
		lazyLoadedSlides: {
			type: Boolean,
			default: true
		}
	},
	data() {
		return {
			slideCount: 100, //temporary
			slideWidth: '100%',
			currentPage: 0,
			slides: []
		}
	},
	created() {
		for(var i = 1; i <= 100; i++ ) {
			this.slides.push(i);
		}
	},
	mounted() {
		this.calculateSlideWidth();
	},
	computed: {
		wrapperStyle() {
			let style = {};

			style["flex-basis"] = this.slideWidth;
			style.transform = `translateX(${this.currentPage * -100}%)`;
			style.transition = "transform 1s ease-out";

			return style;
		},
		activeSlides() {
			if (this.lazyLoadedSlides) {
				let firstSlideIndex = ( this.currentPage - 1 ) * this.slidesPerPage;
				firstSlideIndex = firstSlideIndex >= 0 ? firstSlideIndex : 0;

				let lastSlideIndex = ( this.currentPage + 1 ) * this.slidesPerPage;
				lastSlideIndex = lastSlideIndex < this.slides.length ? lastSlideIndex : ( this.slides.length - 1);

				return this.slides.slice(firstSlideIndex, lastSlideIndex + 1);
			}
		}
	},
	methods: {
		advance(value) {
			this.currentPage += value;
		},
		calculateSlideWidth() {
			let carousel = this.$refs.theCarousel;

			this.slideWidth = (carousel.offsetWidth / this.slidesPerPage) + "px";
		}
	}
}
</script>

<style scoped>
	.carousel {
		position: relative;
		width: 100%;
		overflow: hidden;
		box-sizing: border-box;
	}

	.carousel-wrapper {
		display: flex;
		flex-direction: row;
		width: 100%;
	}

	.slide {
		flex-basis: inherit;
		flex-grow: 0;
		flex-shrink: 0;
		height: 500px;
		background-color: #f5f5f5;
		border: 1px solid #cccccc;
		box-sizing: border-box;
	}

	.pagination-button {
		display: inline;
		cursor: pointer;
	}
</style>
