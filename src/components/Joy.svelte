<script>
	import { onMount } from 'svelte'
	import { slide, fly } from 'svelte/transition'

	export let x = 0
	export let y = 0

	let el
	let drag = false
	let dp = {}

	const g = (e) => {
		if (drag) {
			x += e.movementX
			y += e.movementY
		}
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
		const elp = {
			w: el.offsetWidth,
			h: el.offsetHeight
		}
		const sp = {
			w: (dp.w - c.maxw) / 2 - elp.w,
			h: dp.h - elp.h
		}

		x = pos.x
		y = pos.y

		lineDraw(10, 300, 20, 600)
	})
</script>

<svelte:window on:mousemove={g} on:mouseup={() => (drag = false)} />

<div>
	<div class="absolute bottom-0 left-0 p-4">
		<div>screen: {dp.w} {dp?.h}</div>
	</div>
	<div
		class={`absolute hover:cursor-grab select-none px-4 md:p-0`}
		bind:this={el}
		on:mousedown={() => (drag = true)}
		style={`left: ${x}px; top: ${y}px; z-index:1;`}
		transition:slide
	>
		<div class={`border rounded-full w-32 aspect-square flex justify-center items-center`}>
			<div class="flex flex-col justify-center items-center text-xs">
				<div>drag: {drag}</div>
				<div>abs: {x} {y}</div>
				<div>rel: {x} {y}</div>
			</div>
		</div>
		<div class="h-0 md:h-12"></div>
	</div>
    <div class="w-20 aspect-square " style={`background-color: rgb(${x % 255} ${y % 255} 100)`} />
	<div id="line" />
</div>
