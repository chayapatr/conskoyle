<script>
	import { onMount } from 'svelte'
	import { pinch, rotate } from 'svelte-gestures'

	export let x = 0
	export let y = 0

	let el
	let drag = false
	let dp = {}
	let cors = []

	let data
	let scale = 1
	let rotation = 0

	let prevScale = 1
	let prevRotation = 0

	let previousTouch

	const pinchHandler = (e) => {
		scale = prevScale * e.detail.scale
		if (scale < 1) scale = 1
		else if (scale > 4) scale = 4
		// x = e.detail.center.x;
		// y = e.detail.center.y;
	}

	const rotateHandler = (e) => {
		rotation = Math.min(Math.abs(prevRotation + e.detail.rotation), 360)
		// //prevRotation = rotation
		// x = e.detail.center.x;
		// y = e.detail.center.y;
	}

	const g = (e) => {
		const touch = e.touches[0]

		// if(e.touches.length === 2) {
		//     cors = [[e.touches[0].pageX, e.touches[0].pageY], [e.touches[1].pageX, e.touches[1].pageY]]
		// }
		cors = []
		for (let i = 0; i < e.touches.length; i++) {
			cors = [...cors, [e.touches[i].pageX, e.touches[i].pageY]]
		}

		if (previousTouch) {
			// be aware that these only store the movement of the first touch in the touches array
			let changeX = touch.pageX - previousTouch.pageX
			let changeY = touch.pageY - previousTouch.pageY

			if (drag) {
				x += changeX
				y += changeY
				if (dp) {
					if ((x / dp.w) * 255 < 0) x = 0
					else if ((x / dp.w) * 255 > 255) x = dp.w
					if ((y / dp.h) * 255 < 0) y = 0
					else if ((y / dp.h) * 255 > 255) y = dp.h
				}
			}
		}

		previousTouch = touch
	}

	const lineDraw = (ax, ay, bx, by) => {
		if (ax > bx) {
			bx = ax + bx
			ax = bx - ax
			bx = bx - ax

			by = ay + by
			ay = by - ay
			by = by - ay
		}

		let distance = Math.sqrt(Math.pow(bx - ax, 2) + Math.pow(by - ay, 2))
		let calc = Math.atan((by - ay) / (bx - ax))
		let degree = (calc * 180) / Math.PI

		let line = document.createElement('div')
		line.style.css({
			position: 'absolute',
			height: '1px',
			transformOrigin: 'top left',
			width: distance,
			top: ay + 'px',
			left: ax + 'px',
			transform: `rotate(${degree}deg)`,
			backgroundColor: 'back'
		})
		document.body.appendChild(line)
	}

	onMount(() => {
		let pos = {
			x,
			y
		}

		const c = {
			maxw: 768,
			s: 10
		}

		dp = {
			w: document.documentElement.clientWidth,
			h: document.documentElement.clientHeight,
			sh: document.documentElement.scrollHeight
		}

		x = pos.x
		y = pos.y
	})
	const range = (min, x, max) => {
		return Math.min(Math.max(min, x), max)
	}
	const length = (x1, x2, y1, y2) => {
		return Math.sqrt((x1 - x2) * (x1 - x2) + (y1 - y2) * (y1 - y2))
	}
</script>

<svelte:window
	on:touchmove={g}
	on:touchend={() => {
		drag = false
		previousTouch = undefined
		cors = []
	}}
	on:resize={() => {
		dp = {
			w: document.documentElement.clientWidth,
			h: document.documentElement.clientHeight,
			sh: document.documentElement.scrollHeight
		}
	}}
/>

{#each cors as cor}
	<div
		class="absolute w-12 aspect-square bg-neutral-50/20 border rounded-full"
		style={`top:${cor[1]}px;left:${cor[0]}px`}
	/>
	<div
		class="absolute"
		style={`
            background-color: #eee;
            opacity: 0.5;
            height: 1px;
            width: ${length(cor[0], x + 24, cor[1], y + 24)}px;
            left: ${cor[0] + 24}px;
            top: ${cor[1] + 24}px;
            transform-origin: left;
            transform: rotate(${(Math.atan2(y + 24 - cor[1], x + 24 - cor[0]) * 180) / Math.PI}deg);
        `}
	>
		{Math.floor(((Math.atan2(y + 24 - cor[1], x + 24 - cor[0]) * 180) / Math.PI) * 1000) / 1000}
	</div>
{/each}
<div
	style={`background-color: rgba(${(x / dp.w) * 255}, ${(y / dp.h) * 255}, ${
		Math.min(scale - 1, 3) * (255 / 3)
	}, ${(360 - rotation) / 360})`}
	class="w-[100vw] h-[100vh]"
>
	<div class="absolute bottom-0 left-0 p-4 text-xs">
		<div>({dp.w}, {dp.h})</div>
		<!-- <div>data: {scale} {rotation}</div> -->
	</div>
	<div
		class={`relative hover:cursor-grab select-none w-[100vw] h-[100vh] flex justify-center items-center`}
		bind:this={el}
		on:touchstart={() => (drag = true)}
		use:pinch
		on:pinch={pinchHandler}
		on:pinchup={() => {
			prevScale = scale
		}}
		use:rotate
		on:rotate={rotateHandler}
		on:rotateup={() => {
			prevRotation = rotation
		}}
	>
		<div
			class="absolute h-24 aspect-square bg-neutral-50/50 rounded-full flex justify-center items-center border"
			style={`left: ${range(0, x, dp.w - 96)}px; top: ${range(
				0,
				y,
				dp.h - 96
			)}px; z-index:1; transform:scale(${scale}) rotate(${rotation}deg);`}
		>
			⤴
		</div>
		<div class="flex flex-col justify-center items-center text-xs">
			<div>touch: {drag}</div>
			<div>
				rgba({Math.floor((x / dp.w) * 255)}, {Math.floor((y / dp.h) * 255)}, {Math.floor(
					Math.min(scale - 1, 3) * (255 / 3)
				)}, {Math.floor(((360 - rotation) / 360) * 1000) / 1000})
			</div>
		</div>
	</div>
	<!-- <div class="w-20 aspect-square " style={`background-color: rgb(${x % 255} ${y % 255} 100)`} /> -->
	<div id="line" />
</div>

<!-- style={`left: ${x}px; top: ${y}px; z-index:1;transform:scale(${scale}) rotate(${rotation}deg);`} -->
