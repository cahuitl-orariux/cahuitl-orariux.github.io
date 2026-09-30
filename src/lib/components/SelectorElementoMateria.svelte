<script lang="ts">
	import Agregar from './Agregar.svelte';
	import Eliminar from './Eliminar.svelte';
	import FlechaDropdown from './FlechaDropdown.svelte';
	import type { ModoSeleccion } from './SelectorMaterias.svelte';

	interface Props {
		seleccionable: string;
		modoSeleccion: ModoSeleccion;
		agregarMateriaPorId: (idMateria: string) => void;
		eliminarMateriaPorId: (idMateria: string) => void;
	}
	let {
		seleccionable,
		modoSeleccion: modosSeleccion,
		agregarMateriaPorId,
		eliminarMateriaPorId
	}: Props = $props();
	let seleccionMenuExpandido: string = $state('');
</script>

<li
	class="gradiente my-1 flex min-h-12 flex-col justify-center border-b-2 border-solid text-start last:border-b-0"
	id={'selector-' + seleccionable}
>
	<div class="flex h-auto w-full flex-row items-center justify-between py-3">
		<button
			class="nostyle ml-0.5 flex h-full w-[25px] flex-row"
			onclick={() =>
				(seleccionMenuExpandido = seleccionMenuExpandido === seleccionable ? '' : seleccionable)}
		>
			{#if seleccionMenuExpandido === seleccionable}
				<FlechaDropdown orientacion="abajo" class="w-full" />
			{:else}
				<FlechaDropdown orientacion="derecha" class="w-full " />
			{/if}
		</button>
		<div class="w-full px-2 text-wrap break-words hyphens-auto">
			{seleccionable}
		</div>
		<div class="mr-1 flex h-full w-[75px] flex-row gap-1 p-0">
			<Agregar click={modosSeleccion.agregar} paramsClick={seleccionable} />
			<Eliminar click={modosSeleccion.eliminar} paramsClick={seleccionable} />
		</div>
	</div>
	{#if seleccionMenuExpandido === seleccionable}
		<ul class="flex h-auto w-full flex-col">
			{#each modosSeleccion.menuDetalle(seleccionable) as detalle}
				<li class="flex min-h-12 flex-row text-start">
					<div class="flex w-full flex-row items-center justify-between gap-2">
						<div class="flex h-full w-[50px] flex-row gap-1 p-2"></div>
						<div class=" w-full px-2 text-wrap break-words hyphens-auto">
							{detalle.nombre}
						</div>
						<div class="flex h-full w-[100px] flex-row gap-1 p-2">
							<Agregar click={agregarMateriaPorId} paramsClick={detalle.idMateria} />
							<Eliminar click={eliminarMateriaPorId} paramsClick={detalle.idMateria} />
						</div>
					</div>
				</li>
			{/each}
		</ul>
	{/if}
</li>

<style>
	.gradiente {
		border-image-source: linear-gradient(
			to right,
			transparent 0%,
			rgba(255, 255, 255, 0.7) 50%,
			transparent 100%
		);
		border-image-slice: 1;
	}
</style>
