<template>
	<div class="carousel-wrapper">
		<div ref="theCarousel" :style="carouselStyle" class="carousel">
			<div 
				v-for="(slide,index) in activeSlides" :key="index"
				class="slide-wrapper"
				:style="slideWrapperStyle"
			>
				<slot v-bind="slide" />
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
		slides: {
			type: Array,
			default: () => []
		},
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
			default: false
		},
		transitionSpeed: {
			type: Number,
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
			isDragging: false,
			windowWidth: window.innerWidth,
			carouselWidth: null,
			currentPage: this.initialPage,
			startDragPosition: {x: 0, y: 0},
			draggedOffset: {x: 0, y: 0},
			dragType: ""
		}
	},
	mounted() {
		window.addEventListener("resize", this._onResize);
		this._updateCarouselWidth();

		//Touch Interaction
		let carousel = this.$refs.theCarousel;
		carousel.addEventListener("mousedown", this._onMouseDown);
		carousel.addEventListener("touchstart", this._onTouchStart);
	},
	beforeDestroy() {
		let carousel = this.$refs.theCarousel;
		carousel.removeEventListener("mousedown", this._onMouseDown);
		carousel.removeEventListener("touchstart", this._onTouchStart);
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
		slideCount() {
			return this.slides.length;
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
		// Public Functions
		advancePage(value) {
			this.currentPage += value;
		},

		// Private Functions
		_updateCarouselWidth() {
			this.carouselWidth = this.$refs.theCarousel.offsetWidth;
		},
		_updateWindowWidth() {
			this.windowWidth = window.innerWidth;
		},

		// Event Listener Functions
		_onMouseDown(e) {
			this.dragType = "mouse";
			this._onDragStart(e);
		},
		_onTouchStart(e) {
			this.dragType = "touch";
			this._onDragStart(e);
		},
		_onDragStart(e) {
			this.isDragging = true;

			let carousel = this.$refs.theCarousel,
				dragEvent = null;

			if (this.dragType === "mouse") {
				dragEvent = e;
				carousel.addEventListener("mousemove", this._onDrag);
				carousel.addEventListener("mouseup", this._onDragEnd);
			} else {
				dragEvent = e.touches[0];
				carousel.addEventListener("touchmove", this._onDrag);
				carousel.addEventListener("touchend", this._onDragEnd);
			}

			this.startDragPosition.x = dragEvent.clientX;
			this.startDragPosition.y = dragEvent.clientY;
		},
		_onDrag(e) {
			let dragEvent = (this.dragType === "mouse" ? event : event.touches[0]),
				dragPosition = {x: dragEvent.clientX};

			this.draggedOffset.x = this.startDragPosition.x - dragPosition.x;
		},
		_onDragEnd() {
			if (Math.abs(this.draggedOffset.x) > 100) {
				this.advancePage(Math.sign(this.draggedOffset.x));
			}

			this.draggedOffset.x = 0;

			let carousel = this.$refs.theCarousel
			if (this.dragType === "mouse") {
				carousel.removeEventListener("mousemove", this._onDrag);
				carousel.removeEventListener("mouseup", this._onDragEnd);
			} else {
				carousel.removeEventListener("touchmove", this._onDrag);
				carousel.removeEventListener("touchend", this._onDragEnd);
			}

			this.isDragging = false;
		},
		_onResize() {
			window.requestAnimationFrame(() => {
				this._updateCarouselWidth();
				this._updateWindowWidth();
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
	box-sizing: border-box;
}

.pagination-button {
	display: inline;
	cursor: pointer;
}
</style>
