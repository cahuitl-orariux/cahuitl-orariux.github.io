<script lang="ts">
	import { Color } from '$lib/Color';
	import type { Materia } from 'kesos-ipnsaes-api';
	import { colorMateria } from '$lib/util';
	import Eliminar from './Eliminar.svelte';
	import SvgTick from '$lib/assets/SVGTick.svelte';
	import SvgWarn from '$lib/assets/SVGWarn.svelte';

	interface Props {
		materiasSeleccionadas: Materia[];
		tema: Color[];
		/**
		 * Materia para mostrar información detallada, se pasa hacia el VisualizadorHorario para mostrar el horario
		 */
		materiaParaDetalles?: Materia;
		/**
		 * Traslapes de materias, id de materia a lista de materias traslapadas
		 */
		traslapes: Map<string, Materia[]>;
	}

	let {
		materiasSeleccionadas = $bindable([]),
		tema,
		materiaParaDetalles = $bindable(),
		traslapes
	}: Props = $props();
	let materiasSeleccionadasAnterior: Materia[] = [];
	let articleListaMaterias: HTMLElement;
	let traslapesMateriaActiva = $derived(traslapes?.get(materiaParaDetalles?.id ?? ''));

	$effect(() => {
		if (materiasSeleccionadas.length > materiasSeleccionadasAnterior.length) {
			// Scroll lista
			articleListaMaterias?.scrollTo({
				behavior: 'smooth',
				top: articleListaMaterias?.scrollHeight
			});
		}

		materiasSeleccionadasAnterior = $state.snapshot(materiasSeleccionadas) as Materia[];
	});

	const eliminarMateriaPorId = (idMateria: string) => {
		if (materiaParaDetalles?.id === idMateria) materiaParaDetalles = undefined;
		materiasSeleccionadas = materiasSeleccionadas.filter((materia) => materia.id !== idMateria);
	};
</script>

<article
	class="flex overflow-y-auto not-md:h-[40svh] not-md:w-[100%] md:w-full"
	bind:this={articleListaMaterias}
>
	{#if materiasSeleccionadas.length > 0}
		<ul class="block w-full border-4 border-transparent" id="lista-materias">
			{#each materiasSeleccionadas as materia}
				<li
					class="text-textomateria my-0.5 inline-flex w-[100%] gap-2 rounded-xl border-2 border-transparent p-2 font-semibold hyphens-auto"
					style="background-color: {colorMateria(tema, materia).cssRGBA()}; {materiaParaDetalles
						? traslapesMateriaActiva?.find((m) => m.id === materia.id) // Bordear si se traslapa con la materia seleccionada
							? 'outline: 3px solid red;'
							: materiaParaDetalles.id !== materia.id // Transparentar si no es la materia seleccionada
								? 'opacity: 0.5;'
								: ''
						: ''} "
				>
					<div class="w-16 content-center text-wrap break-words">
						{materia.grupo}
					</div>
					<div class="w-[calc((100%-5rem)/2)] content-center">
						{materia.nombre}
					</div>
					<div class="w-[calc((100%-5rem)/2)] content-center">
						{materia.profesor}
					</div>
					<div class="w-8 content-center">
						<button
							class="has-tooltip ratio-1x1 center flex w-full items-center justify-center p-0!"
							aria-label="Mostrar información de la materia"
							onclick={() => {
								if (materiaParaDetalles === materia) materiaParaDetalles = undefined;
								else materiaParaDetalles = materia;
							}}
						>
							{#if traslapes.get(materia.id)?.length ?? 0 > 0}
								<SvgWarn />
							{:else}
								<SvgTick />
							{/if}
						</button>
						<div class="tooltip relative -left-12 h-0 w-0">
							<div
								class="relative -top-10 -left-28 z-50 w-40 rounded-lg bg-black px-3 py-1.5 font-sans text-sm font-normal break-words whitespace-normal text-white focus:outline-none"
							>
								{#if traslapes.get(materia.id)?.length ?? 0 > 0}
									Presiona para mostrar los traslapes
								{:else}
									No hay traslapes con esta materia
								{/if}
							</div>
						</div>
					</div>

					<div class="w-8 content-center">
						<Eliminar click={eliminarMateriaPorId} nombre={materia.id}></Eliminar>
					</div>
				</li>
			{/each}
		</ul>
	{:else}
		<h3 class="place-self-center text-center text-2xl">
			Empieza a seleccionar materias para generar un horario. Aquí aparecerán todas las materias
			seleccionadas.
		</h3>
	{/if}
</article>

<style>
	/* width */
	::-webkit-scrollbar {
		width: 8px;
	}

	/* Track */
	::-webkit-scrollbar-track {
		background: #f1f1f100;
	}

	/* Handle */
	::-webkit-scrollbar-thumb {
		background: #888;
	}

	/* Handle on hover */
	::-webkit-scrollbar-thumb:hover {
		background: #555;
	}

	ul {
		list-style-type: none;
		padding: 0;
	}
</style>
