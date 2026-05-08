<script lang="ts">
	import { getContext } from 'svelte';

	const i18n = getContext('i18n');
</script>

<div class="max-w-4xl mx-auto py-10 px-4">
	<div class="text-center mb-12">
		<h2 class="text-4xl font-extrabold text-gray-900 dark:text-white mb-4">
			🌟 Prêt pour une nouvelle leçon, Bechi ?
		</h2>
		<p class="text-lg text-gray-600 dark:text-gray-400">
			Tes classes t'attendent. C'est le moment idéal pour explorer de nouveaux algorithmes !
		</p>
	</div>

	<div class="bg-white dark:bg-gray-800 rounded-3xl p-8 shadow-xl border border-gray-100 dark:border-gray-700 flex flex-col items-center">
		<div class="mb-6 p-4 bg-indigo-50 dark:bg-indigo-900/30 rounded-full">
			<svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 text-indigo-600 dark:text-indigo-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
			</svg>
		</div>
		
		<h3 class="text-xl font-semibold text-gray-800 dark:text-gray-200 mb-6">
			Accéder à mon espace de cours
		</h3>

		<button class="group relative inline-flex items-center justify-center px-10 py-4 font-bold text-white transition-all duration-200 bg-indigo-600 font-pj rounded-full focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-600 hover:bg-indigo-700 shadow-lg hover:shadow-indigo-500/50 transform hover:-translate-y-1">
			🚀 REJOINDRE LA CLASSE
		</button>
	</div>
</div>
