<script>
	import { onMount } from 'svelte'
	import { slide } from 'svelte/transition'
    import { pinch, rotate } from 'svelte-gestures';

	export let x = 0
	export let y = 0

	let el
	let drag = false
	let dp = {}

    let data
    let scale = 1
    let rotation = 0

    let prevScale = 1
    let prevRotation = 0

    let previousTouch

    const pinchHandler = (e) => {
        scale = e.detail.scale;
        // x = e.detail.center.x;
        // y = e.detail.center.y;
    }

    const rotateHandler = (e) => {
        rotation = e.detail.rotation;
        // prevRotation = rotation
        // x = e.detail.center.x;
        // y = e.detail.center.y;
    }

	const g = (e) => {
        const touch = e.touches[0];

        if (previousTouch) {
            // be aware that these only store the movement of the first touch in the touches array
            let changeX = touch.pageX - previousTouch.pageX;
            let changeY = touch.pageY - previousTouch.pageY;

            if (drag) {
			    x += changeX
			    y += changeY
                data = changeX
		    }
        };

        previousTouch = touch;
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
</script>

<svelte:window on:touchmove={g} on:touchend={() => {drag = false; previousTouch = undefined}} />

<div style={`background-color: rgba(${x / dp.w * 255}, ${y / dp.h * 255}, ${((scale - 1) % 3) * 255}, ${(360 - rotation) / 360})`} class="w-[100vw] h-[100vh]">
	<div class="absolute bottom-0 left-0 p-4">
		<div>screen: {dp.w} {dp.h} {prevRotation}</div>
		<!-- <div>data: {scale} {rotation}</div> -->
	</div>
	<div
		class={`relative hover:cursor-grab select-none`}
		bind:this={el}
		on:touchstart={() => (drag = true)}
		style={`left: ${x}px; top: ${y}px; z-index:1;transform:scale(${scale}) rotate(${rotation}deg)`}
		transition:slide
	>
		<div
            class={`border w-32 aspect-square flex justify-center items-center bg-neutral-50/60 text-neutral-900 rounded-full text-center`}
            use:pinch
            on:pinch="{pinchHandler}"
            use:rotate
            on:rotate="{rotateHandler}"
        >
			<div class="flex flex-col justify-center items-center text-xs">
				<div>drag: {drag}</div>
				<div>{Math.floor(x / dp.w * 255)} {Math.floor(y / dp.h * 255)} {Math.floor(((scale - 1) % 3) * 255)} {Math.floor((360 - rotation) / 360 * 1000) / 1000}</div>
			</div>
		</div>
		<div class="h-0 md:h-12"></div>
	</div>
    <!-- <div class="w-20 aspect-square " style={`background-color: rgb(${x % 255} ${y % 255} 100)`} /> -->
	<div id="line" />
</div>
