<script lang="ts">
	import { states, selectedLanguage, motion } from '$lib/Stores';
	import { onMount } from 'svelte';
	import type { HassEntity } from 'home-assistant-js-websocket';
	import { getName } from '$lib/Utils';

	export let entity_id: string | undefined = undefined;
	export let name: string | undefined = undefined;
	export let strokeWidth: number = 12;
	export let maxValue: number = 100;
	export let iconUrl = '/path/to/icon.svg'; // Update to the correct path
	let entity: HassEntity;
	let width = 0;
	let mounted = false;
	export let hideOverflow: boolean = false;

	$: if (entity_id) entity = $states?.[entity_id];
	let current_level: string | undefined = undefined;
	let state_state: number | undefined = undefined;

	if (entity_id) {
		current_level = $states?.[entity_id]?.attributes?.current?.level;
		state_state = Number($states?.[entity_id]?.state);
	}

	let fillColor;
	if (
		current_level === 'low' ||
		(state_state !== undefined && state_state >= 0 && state_state <= 35)
	) {
		fillColor = 'green';
	} else if (
		current_level === 'moderate' ||
		(state_state !== undefined && state_state >= 36 && state_state <= 75)
	) {
		fillColor = 'orange';
	} else if (
		current_level === 'high' ||
		(state_state !== undefined && state_state >= 76 && state_state <= 100)
	) {
		fillColor = 'red';
	} else {
		fillColor = 'rgb(255, 255, 255, 0.9)';
	}

	const color = {
		stroke: 'var(--theme-navigate-background-color)',
		fillColor: fillColor
	};

	onMount(() => {
		setTimeout(() => {
			mounted = true;
		}, $motion);
	});

	$: state = Math.min(Math.max(Number(entity?.state || 0), 0), maxValue);

	$: stroke = strokeWidth === null || !strokeWidth ? 9 : strokeWidth;

	$: attributes = {
		cx: width / 2,
		cy: width / 2,
		r: (width - stroke) / 2,
		fill: 'none',
		'stroke-width': stroke,
		'vector-effect': 'non-scaling-stroke'
	};
	$: circumference = 2 * Math.PI * attributes.r;

	function formatNumber(value: number): string {
		return Number.isInteger(value) ? value.toString() : value.toFixed(2);
	}
</script>

<div class="container">
	<div class="bar" bind:clientWidth={width}>
		<svg width="100%" viewBox="0 0 {width} {width}">
			{#if width}
				<circle
					stroke={color.stroke}
					{...attributes}
					transform="rotate(-90 {width / 2} {width / 2})"
				/>

				<circle
					class="progress"
					{...attributes}
					stroke={color.fillColor}
					stroke-dasharray={circumference}
					style:--dashoffset={circumference * (1 - state / maxValue)}
					style:transition="stroke-dashoffset {mounted ? $motion : 0}ms ease"
					transform="rotate(-90 {width / 2} {width / 2})"
				/>
				<!-- Icoon in het midden -->
				<image href={iconUrl} class="icon" width="26" height="26" x="35%" y="35%" />
			{/if}
		</svg>
	</div>

	<div class="overflow" style:overflow={hideOverflow ? 'hidden' : 'visible'}>
		{getName({ name }, entity)}

		<br />
		{formatNumber(Number(entity?.state))}
		{entity?.attributes?.unit_of_measurement}
	</div>
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.5rem;
		padding: var(--theme-sidebar-item-padding);
		overflow: hidden;
		pointer-events: none;
		text-shadow: 0px 0px 5px rgba(0, 0, 0, 0.1);
	}

	.bar {
		width: 5.5rem;
		height: 5.5rem;
		margin-top: 0.1rem;
	}

	.progress {
		stroke-dashoffset: var(--dashoffset);
	}

	.overflow {
		white-space: nowrap;
		text-overflow: ellipsis;
		overflow: hidden;
		text-align: center;
	}

	/* Class for centering the icon */
	.icon {
		transform-origin: center;
	}
</style>
