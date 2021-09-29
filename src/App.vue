<template>
	<!-- wrapper -->
	<div class="wrapper">

		min: {{ cmin }}, max: {{ cmax }}
		<br /> <br />

		<!-- plugin -->
		<div class="v-range-container" :ref="refRange" :style="{ width: `${width}px` }">
			<span class="v-range-interval" :style="{ left: `${coords.lx + 2}px`, right: `${coords.rx + 2}px` }"></span>

			<span class="v-range-item v-range-item--left" :style="{ left: `${coords.lx}px` }"
				@mousedown="grab($event, 'left')"></span>
			<span class="v-range-item v-range-item--right"
				:style="{ right: `${coords.rx}px` }"
				@mousedown="grab($event, 'right')"
			></span>
		</div>

	</div>
</template>

<script>
	export default {
		name: 'VRange',
		props: {
			min: {
				type: Number,
				default: 0
			},
			max: {
				type: Number,
				default: 500
			},
			width: {
				type: Number,
				default: 150
			},
			sizeCircle: {
				type: Number,
				default: 16
			}
		},
		data: () => ({
			isDrag: false,
			refRange: '',
			minRange: 0,
			maxRange: 0,
			coords: {
				lx: 0,
				rx: 0
			},
			grabX: 0,
			currGrab: '',
			cmin: 0,
			cmax: 0,
		}),
		methods: {
			grab(e, elem) {
				this.grabX = e.layerX
				this.isDrag = true
				this.currGrab = elem
			},
			generateRef(name) {
				return `${name}:${String(Math.random()).slice(2, 10)}`
			}
		},
		created() {
			this.refRange = this.generateRef('v-range')
			
			document.addEventListener('mousemove', e => {
				const offsetGrabX = this.sizeCircle - this.grabX
				const maxLeft = this.maxRange - this.sizeCircle - offsetGrabX - this.coords.rx
				const minRight = this.minRange + this.coords.lx + this.grabX + this.sizeCircle

				const { x } = e

				if (this.isDrag) {

					if (this.currGrab === 'left') {
						if (x - this.grabX >= this.minRange && x <= maxLeft) {
							this.coords.lx = x - this.minRange - this.grabX
							this.cmin = Math.round(this.coords.lx * this.max / (this.width - this.sizeCircle * 2))
						}
					} else {
						if (x >= minRight && x + offsetGrabX <= this.maxRange) {
							this.coords.rx = this.maxRange - x - offsetGrabX
							this.cmax = Math.round(this.max - (this.coords.rx * this.max / (this.width - this.sizeCircle)))
							this.cmax = this.max - Math.round(this.coords.rx * this.max / (this.width - this.sizeCircle * 2))
						}
					}

				}
			})

			document.addEventListener('mouseup', () => this.isDrag = false)
		},
		mounted() {
			this.minRange = this.$refs[this.refRange].offsetLeft
			this.maxRange = this.$refs[this.refRange].offsetLeft + this.width
		}
	}
</script>

<style lang="scss">
	* {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	.wrapper {
		width: 100%;
		height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.v-range-container {
		height: 5px;
		border-radius: 8px;
		position: relative;
		background: #aaa;
		user-select: none;
	}

	.v-range-item {
		width: 16px;
		height: 16px;
		background: #000;
		border-radius: 50%;
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		cursor: grab;

		&--left {
			left: 30px;
		}

		&--right {
			right: 0;
		}
	}

	.v-range-interval {
		height: inherit;
		background: #000;
		position: absolute;
	}
</style>