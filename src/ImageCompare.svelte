<svelte:window
	on:touchmove={move}
	on:touchend={end}
	on:mousemove={move}
	on:mouseup={end}
	on:resize={resize}
/>

<div class="container" {style} on:touchstart={start} on:mousedown={start}>
	<img
		bind:this={img}
		src={after}
		alt="after"
		on:mousedown|preventDefault
		on:load={resize}
		{style}
	>
	<img
		src={before}
		alt="before"
		on:mousedown|preventDefault
		style="{style}clip:{vertical ? `rect(0, ${w}px, ${h * offset}px, 0)` : `rect(0, ${w * offset}px, ${h}px, 0)`};"
	>
	{#if overlay}
	<div class="overlay" style="opacity:{opacity}"></div>
	{/if}
	<div class="before-label" style="opacity:{opacity}">
		<slot name="before"></slot>
	</div>
	<div class="after-label" style="opacity:{opacity}">
		<slot name="after"></slot>
	</div>
	<div class={vertical ? "handle vertical" : "handle horizontal"}
		style="{vertical
			? `top: calc(${offset * 100}% - 20px); left: calc(50% - 20px);`
			: `left: calc(${offset * 100}% - 20px); top: calc(50% - 20px);`
		}">
		<div class={vertical ? "arrow-up" : "arrow-left"}></div>
		<div class={vertical ? "arrow-down" : "arrow-right"}></div>
	</div>
</div>

<script>
	let hideOnSlide = true,
		imgOffset = null,
		sliding = false,
		contain = false,
		overlay = true,
		offset = 0.5,
		vertical = false,
		before = '',
		after = '',
		img;

	function resize(e) {
		imgOffset = (e.type === 'load' ? e.target : img).getBoundingClientRect();
	}

	function move(e) {
		if (sliding && imgOffset) {
			let position = vertical
			? (e.touches ? e.touches[0].pageY : e.pageY) - imgOffset.top
			: (e.touches ? e.touches[0].pageX : e.pageX) - imgOffset.left;

			let limit = vertical ? h : w;
			position = position < 0 ? 0 : (position > limit ? limit : position);
			offset = position / limit;
		}
	}

	function start(e) {
		sliding = true;
		move(e);
	}

	function end() {
		sliding = false;
	}

	$: w = imgOffset && imgOffset.width;
	$: h = imgOffset && imgOffset.height;
	$: x = w * offset;
	$: opacity = hideOnSlide && sliding ? 0 : 1;
	$: style = contain ? `width:100%;height:100%;` : `width:${w}px;height:${h}px;`;

	export { before, after, offset, vertical, overlay, contain, hideOnSlide };
</script>

<style>
	.container {
		overflow: hidden;
		position: relative;
		box-sizing: content-box;
	}
	.container img {
		top: 0;
		left: 0;
		z-index: 20;
		display: block;
		max-width: 100%;
		user-select: none;
		object-fit: cover;
		position: absolute;
	}
	.overlay {
		top: 0;
		opacity: 0;
		z-index: 25;
		width: 100%;
		height: 100%;
		position: absolute;
		transition: opacity .5s;
		background: rgba(0, 0, 0, .5);
	}
	.before-label, .after-label {
		top: 0;
		bottom: 0;
		z-index: 25;
		user-select: none;
		position: absolute;
	}
	.before-label { left: 0; }
	.after-label { right: 0; }
	.container:hover > .overlay { opacity: 1; }
	.handle {
		z-index: 30;
		width: 40px;
		height: 40px;
		cursor: move;
		background: none;
		margin-top: -4px;
		margin-left: -4px;
		user-select: none;
		position: absolute;
		border-radius: 50px;
		top: calc(50% - 20px);
		border: 4px solid white;
	}
	.handle:before, .handle:after {
		content: "";
		height: 9999px;
		position: absolute;
		left: calc(50% - 2px);
		border: 2px solid white;
	}
	.handle.vertical:before, .handle.vertical:after {
		width: 9999px;
		top: calc(50% - 2px);
		height: auto;
		left: auto;
	}
	.handle:before { top: 40px; }
	.handle.vertical:before {
		transform: translateX(-100%);
		left: 0;
	}
	.handle:after { bottom: 40px; }
	.handle.vertical:after {
		transform: translateX(100%);
    	right: 0;
	}
	.arrow-right, .arrow-left {
		width: 0;
		height: 0;
		user-select: none;
		position: relative;
		border-top: 10px solid transparent;
		border-bottom: 10px solid transparent;
	}
	.arrow-right {
		left: 23px;
		bottom: 10px;
		border-left: 10px solid white;
	}
	.arrow-left {
		left: 7px;
		top: 10px;
		border-right: 10px solid white;
	}
	.arrow-up, .arrow-down {
		width: 0;
		height: 0;
		user-select: none;
		position: relative;
		border-left: 10px solid transparent;
		border-right: 10px solid transparent;
	}
	.arrow-up {
		top: 8px;
		left: 10px;
		border-bottom: 10px solid white;
	}

	.arrow-down {
		top: 12px;
		left: 10px;
		border-top: 10px solid white;
	}
</style>
