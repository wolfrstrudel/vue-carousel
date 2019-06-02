<template>
	<div ref="theCarouselWrapper" class="carousel-wrapper">
		<div ref="theCarousel" :style="carouselStyle" class="carousel">
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

		<div class="pagination-button pagination-button-previous" @click="advancePage(-5)">
			Previous 5
		</div>

		<div class="pagination-button pagination-button-previous" @click="advancePage(-1)">
			Previous
		</div>

		<div class="pagination-button pagination-button-next" @click="advancePage(1)">
			Next
		</div>

		<div class="pagination-button pagination-button-next" @click="advancePage(5)">
			Next 5
		</div>
	</div>
</template>

<script>

export default {
	name: "Carousel",
	props: {
		initialPage: {
			type: Number,
			default: 0
		},
		slidesPerPage: {
			type: Number,
			default: 3
		},
		lazyLoadedSlides: {
			type: Boolean,
			default: true
		},
		transitionSpeed: {
			type: [Number, Object],
			default: 1000
		},
		spaceBetweenSlides: {
			type: Number,
			default: 20
		},
		loop: {
			type: Boolean,
			default: false
		}
	},
	data() {
		return {
			slideCount: 20, //temporary
			isDragging: false,
			windowWidth: window.innerWidth,
			carouselWidth: null,
			currentPage: this.initialPage,
			slides: [],
			startDragPosition: {x: 0, y: 0},
			draggedOffset: {x: 0, y: 0},
			dragType: ""
		}
	},
	created() {
		for(var i = 1; i <= this.slideCount; i++ ) {
			this.slides.push(i);
		}
	},
	mounted() {
		window.addEventListener("resize", this.onResize);
		this.updateCarouselWidth();

		//Touch Interaction
		let carousel = this.$refs.theCarousel;
		carousel.addEventListener("mousedown", this.onMouseDown);
		carousel.addEventListener("touchstart", this.onTouchStart);
	},
	beforeDestroy() {
		let carousel = this.$refs.theCarousel;
		carousel.removeEventListener("mousedown", this.onMouseDown);
		carousel.removeEventListener("touchstart", this.onTouchStart);
	},
	watch: {
		currentPage(newPage) {
			if(!this.loop) {
				if(newPage < 0) this.currentPage = 0;
				if(newPage > this.pageCount - 1) this.currentPage = this.pageCount - 1;
			}
		}
	},
	computed: {
		canTransition() {
			return !this.isDragging;
		},
		offset() {
			return {x: this.currentPage * -(this.carouselWidth + this.spaceBetweenSlides) - this.draggedOffset.x};
		},
		carouselStyle() {
			let style = {};

			style.transform = `translateX(${this.offset.x}px)`;
			style.transition = !this.canTransition ? "none" : `transform ${this.transitionSpeed / 1000}s ease-out`;

			return style;
		},
		slideWrapperStyle() {
			let style = {};

			style.width = this.slideWidth + "px";
			style["padding-right"] = this.spaceBetweenSlides + "px";

			style.transform = [];
			style.transform.push(`translateX(${this.slidePosition}px)`);

			return style;
		},
		slidePosition() {
			if(this.lazyLoadedSlides) {
				return this.firstLoadedPage * this.slidesPerPage * this.slideWidth;
			} 
				
			return 0;
		},
		slideWidth() {
			return (this.carouselWidth + this.spaceBetweenSlides) / this.slidesPerPage;
		},
		pageCount() {
			return Math.ceil(this.slideCount / this.slidesPerPage);
		},
		firstLoadedPage() {
			if(this.lazyLoadedSlides && this.loop) return this.currentPage - 1;
			else if (this.lazyLoadedSlides) return this.currentPage - 1 >= 0 ? this.currentPage - 1 : 0;
			else if (this.loop) {
				// Need to figure out what happens when looping with no lazy loading
			} else return 0;
		},
		firstSlideIndex() {
			let index = this.firstLoadedPage * this.slidesPerPage;

			if (this.loop) {
				index = index % this.slideCount;
				return index >= 0 ? index : index + this.slideCount;
			} else {
				return index >= 0 ? index : 0;
			}

			return index;
		},
		lastSlideIndex() {
			let index = this.firstSlideIndex + ( 3 * this.slidesPerPage ) - 1;

			if (this.loop) {
				index = index % this.slideCount;
				return index >= 0 ? index : index + this.slideCount;
			} else {
				return index >= 0 ? index : 0;
			}
		},
		activeSlides() {
			if (this.lazyLoadedSlides) {
				if(this.firstSlideIndex > this.lastSlideIndex) {
					let firstSlides = this.slides.slice(this.firstSlideIndex, this.slideCount);
					let lastSlides = this.slides.slice(0, this.lastSlideIndex + 1);

					return [...firstSlides, ...lastSlides]
				}

				return this.slides.slice(this.firstSlideIndex, this.lastSlideIndex + 1);
			} else {
				return this.slides;
			}
		}
	},
	methods: {
		advancePage(value) {
			this.currentPage += value;
		},
		updateCarouselWidth() {
			this.carouselWidth = this.$refs.theCarousel.offsetWidth;
		},
		updateWindowWidth() {
			this.windowWidth = window.innerWidth;
		},

		// Event Listener Functions
		onMouseDown(e) {
			this.dragType = "mouse";
			this.onDragStart(e);
		},
		onTouchStart(e) {
			this.dragType = "touch";
			this.onDragStart(e);
		},
		onDragStart(e) {
			this.isDragging = true;

			let carousel = this.$refs.theCarousel,
				dragEvent = null;

			if (this.dragType === "mouse") {
				dragEvent = e;
				carousel.addEventListener("mousemove", this.onDrag);
				carousel.addEventListener("mouseup", this.onDragEnd);
			} else {
				dragEvent = e.touches[0];
				carousel.addEventListener("touchmove", this.onDrag);
				carousel.addEventListener("touchend", this.onDragEnd);
			}

			this.startDragPosition.x = dragEvent.clientX;
			this.startDragPosition.y = dragEvent.clientY;
		},
		onDrag(e) {
			let dragEvent = (this.dragType === "mouse" ? event : event.touches[0]),
				dragPosition = {x: dragEvent.clientX};

			this.draggedOffset.x = this.startDragPosition.x - dragPosition.x;
		},
		onDragEnd() {
			if (Math.abs(this.draggedOffset.x) > 100) {
				this.advancePage(Math.sign(this.draggedOffset.x));
			}

			this.draggedOffset.x = 0;

			let carousel = this.$refs.theCarousel
			if (this.dragType === "mouse") {
				carousel.removeEventListener("mousemove", this.onDrag);
				carousel.removeEventListener("mouseup", this.onDragEnd);
			} else {
				carousel.removeEventListener("touchmove", this.onDrag);
				carousel.removeEventListener("touchend", this.onDragEnd);
			}

			this.isDragging = false;
		},
		onResize() {
			window.requestAnimationFrame(() => {
				this.updateCarouselWidth();
				this.updateWindowWidth();
			});
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
}

.slide-wrapper {
	flex-grow: 0;
	flex-shrink: 0;
	height: 500px;
	box-sizing: border-box;
}

.slide {
	height: 100%;
	background-color: #f5f5f5;
	border: 1px solid #cccccc;
}

.pagination-button {
	display: inline;
	cursor: pointer;
}
</style>
