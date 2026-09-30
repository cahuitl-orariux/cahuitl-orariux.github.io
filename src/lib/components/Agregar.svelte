<script lang="ts">
	interface Props {
		paramsClick: any;
		click: (paramsClick: any) => void;
	}
	let { click, paramsClick }: Props = $props();
</script>

<button
	aria-label="Agregar"
	class="ratio-1x1 center flex w-full items-center justify-center p-0!"
	onclick={() => click(paramsClick)}
>
	<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
		<path
			d="M4 12H20M12 4V20"
			stroke="#ffffff"
			stroke-width="2"
			stroke-linecap="round"
			stroke-linejoin="round"
		/>
	</svg>
</button>
