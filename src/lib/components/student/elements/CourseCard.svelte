<script lang="ts">
	import { goto } from '$app/navigation';
	import { getContext } from 'svelte';

	const i18n = getContext('i18n');

	export let title: string;
	export let progress: number = 0;
	export let subject: string = 'computer-science';
	export let href: string = '';

	// Ajout d'une propriété accent pour des bordures visibles
	const subjectConfig: Record<string, { icon: string; color: string; border: string }> = {
		mathematics: { icon: '📊', color: 'bg-blue-50/50', border: 'border-blue-400' },
		science: { icon: '🔬', color: 'bg-teal-50/50', border: 'border-teal-400' },
		'computer-science': { icon: '💻', color: 'bg-indigo-50/50', border: 'border-indigo-400' },
		english: { icon: '📚', color: 'bg-purple-50/50', border: 'border-purple-400' }
		// ... les autres suivront le même modèle
	};

	$: subjectStyle = subjectConfig[subject] || subjectConfig['computer-science'];

	function handleClick() {
		goto('/student/chat');
	}
</script>

<button
	class="group relative flex flex-col justify-between w-full p-8 rounded-3xl border-2 transition-all duration-300 text-left min-h-[240px] shadow-sm
	{subjectStyle.color} {subjectStyle.border}
	hover:shadow-2xl hover:-translate-y-2 hover:bg-white dark:hover:bg-gray-800"
	on:click={handleClick}
>
	<div class="absolute top-4 right-4 px-3 py-1 bg-white dark:bg-gray-700 border {subjectStyle.border} rounded-full shadow-sm">
		<span class="text-[10px] font-black uppercase tracking-widest text-gray-700 dark:text-gray-200">
			Cours actif
		</span>
	</div>

	<div>
		<div class="mb-6 inline-flex p-4 rounded-2xl bg-white dark:bg-gray-700 border-2 {subjectStyle.border} group-hover:rotate-12 transition-transform shadow-md">
			<span class="text-3xl leading-none">{subjectStyle.icon}</span>
		</div>

		<h3 class="text-xl font-extrabold text-gray-900 dark:text-white leading-tight mb-2">
			{title}
		</h3>
		
		<div class="flex items-center gap-2 text-indigo-600 dark:text-indigo-400 font-bold text-sm opacity-0 group-hover:opacity-100 transition-opacity">
			<span>Continuer l'apprentissage</span>
			<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M13 7l5 5m0 0l-5 5m5-5H6" />
			</svg>
		</div>
	</div>

	<div class="w-full mt-6">
		<div class="flex justify-between items-center mb-2">
			<span class="text-xs font-black text-gray-500 uppercase">{progress || 0}% complété</span>
		</div>
		<div class="h-4 w-full bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden border border-gray-300 dark:border-gray-600">
			<div
				style="width: {progress || 0}%"
				class="h-full bg-indigo-600 transition-all duration-1000 ease-out flex items-center justify-end px-1"
			>
                <div class="h-1.5 w-1.5 bg-white rounded-full animate-ping"></div>
            </div>
		</div>
	</div>
</button>

<style>
    /* Pas besoin de line-clamp ici, on mise sur l'espace */
</style>