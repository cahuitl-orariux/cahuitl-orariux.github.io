<script lang="ts">
	import { obtenerNombresDeMaterias, obtenerProfesores } from '$lib/data/db';
	import { Materia } from 'kesos-ipnsaes-api';
	import Agregar from './Agregar.svelte';
	import Eliminar from './Eliminar.svelte';
	import FlechaDropdown from './FlechaDropdown.svelte';
	import Dropdown from './Dropdown.svelte';
	import SelectorElementoMateria from './SelectorElementoMateria.svelte';

	interface Props {
		todasLasMaterias: Materia[];
		materiasSeleccionadas: Materia[];
	}

	let { todasLasMaterias, materiasSeleccionadas = $bindable([]) }: Props = $props();

	/**
	 * Nombres de todas las materias
	 */
	let nombresDeMaterias: string[] = $derived(
		obtenerNombresDeMaterias(todasLasMaterias).values().toArray()
	);
	let profesores: string[] = $derived(obtenerProfesores(todasLasMaterias).values().toArray());

	/**
	 * Proxy para el menú de detalle de un elemento seleccionable
	 */
	export interface elementoDetalle {
		/**
		 * Nombre a mostrar en el menú de detalle
		 */
		nombre: string;
		/**
		 * Id de la materia para agregarla
		 */
		idMateria: string;
	}
	export interface ModoSeleccion {
		/**
		 * Determina los nombres de los elementos seleccionables, dado el modo de seleccion
		 */
		selector: () => string[];
		/**
		 * Determina los elementos que se mostraran en el menu de detalle de un elemento, dado el modo de seleccion.
		 *
		 */
		menuDetalle: (nombre: string) => elementoDetalle[];
		/**
		 * Determina el evento de click en agregar en el selector de elementos, dado el modo de seleccion
		 */
		agregar: (nombre: string) => void;
		/**
		 * Determina el evento de click en eliminar en el selector de elementos, dado el modo de seleccion
		 */
		eliminar: (nombre: string) => void;
	}

	/*
         TODO: refactorizar
         No recuerdo exactamente la razón de la separación en la función y el diccionario, parece ser un proxy que en algo se necesita, pero no distingo exactamente qué. Por ende no sé como nombrar la diferencia.
        */
	const modosSeleccion = (modo: string): ModoSeleccion => {
		const modoSeleccion = modosSeleccionA[modo];

		return {
			selector: () => {
				return modoSeleccion.selector();
			},
			menuDetalle: (nombre: string) => {
				return new Set(modoSeleccion.menuDetalle(nombre)).values().toArray();
			},
			agregar: (nombre: string) => modoSeleccion.agregar(nombre),
			eliminar: (nombre: string) => modoSeleccion.eliminar(nombre)
		};
	};

	const modosSeleccionA: Record<string, ModoSeleccion> = {
		'Por materia': {
			selector: () => nombresDeMaterias,
			menuDetalle: (materiaNombre: string) => {
				const materiasDeAsignatura = todasLasMaterias.filter(
					(materia) => materia.nombre === materiaNombre
				);

				/**
				 * Implementación de los tres modos de detalle:
				 * 1. Por profesor: ideal cuando son pocos grupos ( < 5)
				 * 2. Por turno: ideal cuando hay muchos grupos ( >= 5)
				 * 3. Inteligente: define entre ambos modos automáticamente
				 */
				const determinarModoDetalle = (materias: Materia[]): 'por_profesor' | 'por_turno' => {
					// Modo inteligente: decide automáticamente basado en la cantidad de grupos
					return materias.length < 5 ? 'por_profesor' : 'por_turno';
				};

				const modoDetalle = determinarModoDetalle(materiasDeAsignatura);

				if (modoDetalle === 'por_profesor') {
					// Mostrar por profesor (nombre del profesor)
					return materiasDeAsignatura.map((materia) => ({
						nombre: materia.profesor,
						idMateria: materia.id
					}));
				} else {
					// Mostrar por turno (grupo + horario)
					return materiasDeAsignatura.map((materia) => {
						// Extraer información del horario para mostrar el turno
						const primeraClase = materia.horario[0];
						const horarioInfo = primeraClase
							? `${primeraClase.dia} ${primeraClase.horaInicio}-${primeraClase.horaFin}`
							: 'Sin horario';

						return {
							nombre: `${materia.grupo}`,
							idMateria: materia.id
						};
					});
				}
			},
			agregar: (materiaNombre: string) => {
				console.log('Agregar por materia: ', materiaNombre);

				let materiasAgregar = todasLasMaterias.filter((materia) => {
					return materia.nombre === materiaNombre;
				});

				materiasAgregar = materiasAgregar.filter((materia) => {
					return !materiasSeleccionadas.some((materiaSeleccionada) => {
						return materia == materiaSeleccionada;
					});
				});

				materiasSeleccionadas.push(...materiasAgregar);
			},
			eliminar: (materiaNombre: string) => {
				console.log('Eliminar por materia: ', materiaNombre);

				materiasSeleccionadas = materiasSeleccionadas.filter((materia) => {
					return materia.nombre !== materiaNombre;
				});
			}
		},
		'Por profesor': {
			selector: () => profesores,
			menuDetalle: (profesorNombre: string) => {
				return todasLasMaterias
					.filter((materia) => materia.profesor === profesorNombre)
					.map((materia) => ({
						nombre: `${materia.nombre} (${materia.grupo})`,
						idMateria: materia.id
					}));
			},
			agregar: (profesorNombre: string) => {
				console.log('Agregar por profesor: ', profesorNombre);

				let materiasAgregar = todasLasMaterias.filter((materia) => {
					return materia.profesor === profesorNombre;
				});

				materiasAgregar = materiasAgregar.filter((materia) => {
					return !materiasSeleccionadas.some((materiaSeleccionada) => {
						return materia == materiaSeleccionada;
					});
				});

				materiasSeleccionadas.push(...materiasAgregar);
			},
			eliminar: (profesorNombre: string) => {
				console.log('Eliminar por profesor: ', profesorNombre);

				materiasSeleccionadas = materiasSeleccionadas.filter((materia) => {
					return materia.profesor !== profesorNombre;
				});
			}
		},
		'Por horario (no disponible)': {
			selector: () => [],
			menuDetalle: (horario: string) => {
				return [];
			},
			agregar: (horario: string) => {
				console.log('Agregar por horario: ', horario);
			},
			eliminar: (horario: string) => {
				console.log('Eliminar por horario: ', horario);
			}
		},
		'Por grupo (no disponible)': {
			selector: () => [],
			menuDetalle: (grupo: string) => {
				return [];
			},
			agregar: (grupo: string) => {
				console.log('Agregar por grupo: ', grupo);
			},
			eliminar: (grupo: string) => {
				console.log('Eliminar por grupo: ', grupo);
			}
		}
	};

	const agregarMateriaPorId = (idMateria: string) => {
		let materia = todasLasMaterias.find((materia) => materia.id === idMateria);
		if (!materia) throw new Error('No se encontró la materia con ese id');
		if (materiasSeleccionadas.some((materia) => materia.id === idMateria)) return; // La materia ya está seleccionada
		materiasSeleccionadas.push(materia);
	};
	const eliminarMateriaPorId = (idMateria: string) => {
		materiasSeleccionadas = materiasSeleccionadas.filter((materia) => materia.id !== idMateria);
	};

	let modoSeleccion: string = $state('Por materia');
	let dropdownOpen = $state(false);
	let activeClass = ' hover:text-green-700 dark:hover:text-green-500';

	let visible = $state(true);
	/**
	 * Cuando se hace click en un elemento del selector, se abre el detalle de ese elemento
	 */
	let seleccionMenuExpandido: string = $state('');
	$effect(() => {
		if (seleccionMenuExpandido === '') {
			return;
		}
		const offset = document.getElementById('selector-' + seleccionMenuExpandido)?.offsetTop;
		const selector = document.getElementById('selector-materias');
		const offsetInicial: number | undefined = (selector?.childNodes[0] as any)?.offsetTop;

		if (offset === undefined || offsetInicial === undefined)
			throw new Error('No hay nada en el selector');

		selector?.scrollTo({
			behavior: 'smooth',
			top: offset - offsetInicial
		});
	});
</script>

<aside
	class=" relative block transition-[width] transition-discrete not-has-checked:w-0 not-md:h-[60svh] not-md:w-[100%] md:h-full has-checked:md:w-[40%] has-checked:lg:w-[40%]"
>
	{#if visible}
		<input hidden class="peer" type="checkbox" checked />
	{:else}
		<input hidden class="peer" type="checkbox" />
	{/if}

	<div
		class="bg-accent absolute top-0 bottom-0 -left-12 w-12 flex-col justify-center p-2 not-md:hidden md:flex"
	>
		<button
			aria-label="Mostrar u ocultar selector de materias"
			class="contents h-full w-full"
			onclick={() => (visible = !visible)}
		>
			{#if visible}
				<FlechaDropdown orientacion="izquierda"></FlechaDropdown>
			{:else}
				<FlechaDropdown orientacion="derecha"></FlechaDropdown>
			{/if}
		</button>
	</div>

	<div class="relative h-11 w-full peer-not-checked:hidden">
		<Dropdown bind:open={dropdownOpen} class="w-[100%]">
			{#each Object.keys(modosSeleccionA) as modo}
				<li
					class="bg-primary h-10 w-full content-center {modo === modoSeleccion ? activeClass : ''}"
				>
					<button
						class="nostyle"
						onclick={() => {
							modoSeleccion = modo;
							dropdownOpen = false;
						}}
					>
						{modo}
					</button>
				</li>
			{/each}

			{#snippet titulo()}
				{modoSeleccion}
			{/snippet}
		</Dropdown>
	</div>
	<ul
		class="block h-[calc(100%-2.75rem)] overflow-y-auto p-1 peer-not-checked:hidden"
		id="selector-materias"
	>
		{#each modosSeleccion(modoSeleccion).selector() as seleccionable}
			<SelectorElementoMateria
				{seleccionable}
				modoSeleccion={modosSeleccion(modoSeleccion)}
				{agregarMateriaPorId}
				{eliminarMateriaPorId}
			/>
		{/each}
	</ul>
</aside>

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

	button.nostyle:focus {
		background-color: transparent;
		outline: 0;
	}

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
