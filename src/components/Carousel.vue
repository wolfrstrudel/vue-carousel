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
			type: [Number, Object],
			default: 1000
		},
		spaceBetweenSlides: {
			type: Number,
			default: 20
		}
	},
	data() {
		return {
			isMounted: false,
			isDragging: false,
			windowWidth: window.innerWidth,
			slideCount: 100, //temporary
			slideWidth: null,
			currentPage: 0,
			slides: [],
			startDragPosition: {x: 0, y: 0},
			draggedOffset: {x: 0, y: 0},
			dragType: ""
		}
	},
	created() {
		for(var i = 1; i <= 20; i++ ) {
			this.slides.push(i);
		}

		let defaultValue = 1000;
	},
	mounted() {
		this.isMounted = true;

		window.addEventListener("resize", this.onResize);
		this.calculateSlideWidth();

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
	computed: {
		offset() {
			if(this.isMounted) 
				return {x: this.currentPage * -(this.$refs.theCarousel.offsetWidth + this.spaceBetweenSlides) - this.draggedOffset.x};
			else
				return {x: 0};
		},
		carouselStyle() {
			let style = {};

			style.transform = `translateX(${this.offset.x}px)`;
			style.transition = this.isDragging ? "none" : `transform ${this.activeTransitionSpeed / 1000}s ease-out`;

			return style;
		},
		slideWrapperStyle() {
			let style = {};

			style.width = this.slideWidth + "px";
			style["padding-right"] = this.spaceBetweenSlides + "px";

			style.transform = [];
			style.transform.push(`translateX(${this.calculateSlidePosition()}px)`);

			return style;
		},
		pageCount() {
			return Math.ceil(this.slides.length / this.activeSlidesPerPage);
		},
		activeSlidesPerPage() {
			let defaultValue = 1;
			return this.extractActiveProperty(this.slidesPerPage, defaultValue);
		},
		activeTransitionSpeed() {
			let defaultValue = 1000;
			return this.extractActiveProperty(this.transitionSpeed, defaultValue);
		},
		activeSlides() {
			if (this.lazyLoadedSlides) {
				let firstSlideIndex = ( this.currentPage - 1 ) * this.activeSlidesPerPage;
				firstSlideIndex = firstSlideIndex >= 0 ? firstSlideIndex : 0;

				let lastSlideIndex = firstSlideIndex + ( 3 * this.activeSlidesPerPage) - 1;
				lastSlideIndex = lastSlideIndex < this.slides.length ? lastSlideIndex : ( this.slides.length - 1);

				return this.slides.slice(firstSlideIndex, lastSlideIndex + 1);
			} else {
				return this.slides;
			}
		}
	},
	methods: {
		extractActiveProperty(property, defaultValue) {
			if(!this.isObject(property) && typeof property !== 'undefined') {
				return property;
			} else if (this.isObject(property)) {
				let windowWidth = this.windowWidth,
					activeProperty = property[Math.min.apply(null,Object.keys(property).filter(x => {return x >= windowWidth}))];

				if (Number.isInteger(activeProperty)) return activeProperty;
				else if (typeof property.default !== 'undefined') return property.default;
			}

			return defaultValue;
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
			let carousel = this.$refs.theCarouselWrapper;

			this.slideWidth = carousel.offsetWidth / this.activeSlidesPerPage - ((this.activeSlidesPerPage - 1) * this.spaceBetweenSlides / this.activeSlidesPerPage);
		},
		calculateSlidePosition() {
			if(this.lazyLoadedSlides) {
				return (this.currentPage - 1 >= 0 ? this.currentPage - 1 : 0)  * this.activeSlidesPerPage * this.slideWidth;
			} else {
				return 0;
			}
		},
		isObject(value) {
			return value && typeof value === 'object' && value.constructor === Object;
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
			console.log("here");
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
				this.advance(Math.sign(this.draggedOffset.x));
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
				this.updateWindowWidth();
			});
			this.calculateSlideWidth();
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
