<template>
	<div ref="theCarousel" class="carousel-wrapper">
		<div :style="carouselStyle" class="carousel">
			<div
				v-for="(slide, index) in activeSlides"
				:key="index"
				:style="slideWrapperStyle"
				class="slide-wrapper"
			>
				<div class="slide">
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
			default: false
		},
		transitionSpeed: {
			type: Number,
			default: 1000
		},
		spaceBetweenSlides: {
			type: Number,
			default: 20
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
		window.addEventListener("resize", this.onResize);
		this.calculateSlideWidth();
	},
	computed: {
		carouselStyle() {
			let style = {};

			style.transform = `translateX(calc(${this.currentPage * -100}% - ${this.currentPage * this.spaceBetweenSlides}px )`;
			style.transition = `transform ${this.transitionSpeed / 1000}s ease-out`;

			return style;
		},
		slideWrapperStyle() {
			let style = {};

			style.width = this.slideWidth + "px";
			style["margin-right"] = this.spaceBetweenSlides + "px";

			style.transform = [];
			style.transform.push(`translateX(${this.calculateSlidePosition()}px)`);

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
		onResize() {
			this.calculateSlideWidth();
		},
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

			this.slideWidth = carousel.offsetWidth / this.slidesPerPage - ((this.slidesPerPage - 1) * this.spaceBetweenSlides / this.slidesPerPage);
		},
		calculateSlidePosition(index) {
			if(this.lazyLoadedSlides) {
				return (this.currentPage - 1 >= 0 ? this.currentPage - 1 : 0)  * this.slidesPerPage * this.slideWidth;
			} else {
				return 0;
			}

		}
	}
}
</script>

<style scoped>
	.carousel-wrapper {
		position: relative;
		width: 100%;
		overflow: hidden;
		box-sizing: border-box;
	}

	.carousel {
		display: flex;
		flex-direction: row;
		width: 100%;
	}

	.slide-wrapper {
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
