<template>
	<div ref="theCarousel" class="carousel">
		<div :style="wrapperStyle" class="carousel-wrapper">
			<div
				v-for="(slide, index) in activeSlides"
				:key="index"
				:style="{'transform': `translateX(${calculateSlidePosition(index)})`}"
				class="slide"
				>
				<div class="slide-wrapper">
					{{ slide }}
				</div>
			</div>
		</div>

		<div class="pagination-button pagination-button-previous" @click="advance(-5)">
			Previous 5
		</div>

		<div class="pagination-button pagination-button-previous" @click="advance(-1)">
			Previous
		</div>

		<div class="pagination-button pagination-button-next" @click="advance(1)">
			Next
		</div>

		<div class="pagination-button pagination-button-next" @click="advance(5)">
			Next 5
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
			slideWidth: null,
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

			style["flex-basis"] = this.slideWidth + "px";
			style.transform = `translateX(${this.currentPage * -100}%)`;
			style.transition = "transform 1s ease-out";

			return style;
		},
		pageCount() {
			return Math.ceil(this.slides.length / this.slidesPerPage);
		},
		activeSlides() {
			if (this.lazyLoadedSlides) {
				let firstSlideIndex = ( this.currentPage - 1 ) * this.slidesPerPage;
				firstSlideIndex = firstSlideIndex >= 0 ? firstSlideIndex : 0;

				let lastSlideIndex = firstSlideIndex + ( 3 * this.slidesPerPage) - 1;
				lastSlideIndex = lastSlideIndex < this.slides.length ? lastSlideIndex : ( this.slides.length - 1);

				return this.slides.slice(firstSlideIndex, lastSlideIndex + 1);
			} else {
				return this.slides;
			}
		}
	},
	methods: {
		advance(value) {
			if (this.currentPage + value < 0) {
				this.currentPage = 0;
			} else if (this.currentPage + value > this.pageCount - 1) {
				this.currentPage = this.pageCount - 1;
			} else {
				this.currentPage += value;
			}
		},
		calculateSlideWidth() {
			let carousel = this.$refs.theCarousel;

			this.slideWidth = carousel.offsetWidth / this.slidesPerPage;
		},
		// For Lazy Loading Slides
		calculateSlidePosition(index) {
			return (this.currentPage - 1 >= 0 ? this.currentPage - 1 : 0)  * this.slidesPerPage * this.slideWidth + "px";
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
