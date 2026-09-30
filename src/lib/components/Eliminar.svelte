<script lang="ts">
	interface Props {
		paramsClick: any;
		click: (paramsClick: any) => void;
	}
	let { click, paramsClick }: Props = $props();
</script>

<button
	aria-label="Eliminar"
	class="ratio-1x1 center flex w-full items-center justify-center p-0!"
	onclick={() => click(paramsClick)}
>
	<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none">
		<path
			d="M17 12C17 11.4477 16.5523 11 16 11H8C7.44772 11 7 11.4477 7 12C7 12.5523 7.44771 13 8 13H16C16.5523 13 17 12.5523 17 12Z"
			fill="#ffffff"
		/>
	</svg>
</button>
