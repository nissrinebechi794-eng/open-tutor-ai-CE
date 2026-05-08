<script lang="ts">
	import { createEventDispatcher, onMount, getContext } from 'svelte';
	const i18n = getContext('i18n');
	import { goto } from '$app/navigation';
	import { user } from '$lib/stores';

	export let username: string = '';
	export let toggleSidebar: () => void;
	export let isDarkMode: boolean = false;

	let searchQuery: string = '';
	let notificationCount: number = 2; // Exemple pour le badge
	let isSearchFocused: boolean = false;
	let showNotifications: boolean = false;
	let showMobileMenu: boolean = false;
	let showUserDropdown: boolean = false;

	function toggleUserDropdown() {
		showUserDropdown = !showUserDropdown;
	}

	const dispatch = createEventDispatcher();

	function toggleDarkMode() {
		isDarkMode = !isDarkMode;
		dispatch('darkModeToggle', { isDarkMode });
	}

	function handleSearch(e: KeyboardEvent) {
		if (e.key === 'Enter') {
			dispatch('search', { query: searchQuery });
		}
	}

	function toggleNotificationPanel() {
		showNotifications = !showNotifications;
	}

	let notificationRef: HTMLDivElement;
	let mobileMenuRef: HTMLDivElement;

	onMount(() => {
		const handleClickOutside = (event: MouseEvent) => {
			if (notificationRef && !notificationRef.contains(event.target as Node) && showNotifications) {
				showNotifications = false;
			}
			const userDropdownRef = document.getElementById('user-dropdown-container');
			if (userDropdownRef && !userDropdownRef.contains(event.target as Node) && showUserDropdown) {
				showUserDropdown = false;
			}
		};

		document.addEventListener('click', handleClickOutside);
		return () => document.removeEventListener('click', handleClickOutside);
	});
</script>

<header
	class="sticky top-0 w-full transition-all duration-300 z-[999] border-b {isDarkMode ? 'bg-gray-900/80 border-gray-800 text-white' : 'bg-white/80 border-gray-100 text-gray-800'} backdrop-blur-md px-6 py-3 flex items-center justify-between"
>
	<div class="flex items-center gap-4">
		<button
			class="md:hidden p-2 rounded-xl transition-colors {isDarkMode ? 'hover:bg-gray-800' : 'hover:bg-gray-100'}"
			on:click={toggleSidebar}
		>
			<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
			</svg>
		</button>

		<div class="flex flex-col">
			<h1 class="text-lg font-bold tracking-tight">
				{username ? $i18n.t('Bonjour') + ' ' + username : $i18n.t('Bonjour')} 👋
			</h1>
			<p class="text-xs {isDarkMode ? 'text-gray-400' : 'text-gray-500'} font-medium hidden sm:block">
				{$i18n.t("Apprenons quelque chose de nouveau aujourd'hui !")}
			</p>
		</div>
	</div>

	<div class="flex items-center gap-2 lg:gap-4">
		<div class="hidden md:flex items-center relative group">
			<div class="absolute left-3 {isDarkMode ? 'text-gray-500' : 'text-gray-400'}">
				<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
				</svg>
			</div>
			<input
				type="text"
				placeholder={$i18n.t('Recherche...')}
				class="pl-10 pr-4 py-2 w-48 lg:w-64 text-sm rounded-xl border transition-all duration-200 outline-none
				{isDarkMode ? 'bg-gray-800 border-gray-700 focus:bg-gray-700 focus:border-blue-500' : 'bg-gray-50 border-gray-200 focus:bg-white focus:ring-4 focus:ring-blue-50/50 focus:border-blue-400'}"
				bind:value={searchQuery}
				on:keydown={handleSearch}
			/>
		</div>

		<div class="flex items-center gap-1 sm:gap-2">
            <button class="hidden sm:flex items-center gap-2 px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-semibold transition-all shadow-sm hover:shadow-indigo-200 active:scale-95">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
                </svg>
                { $i18n.t('Soutien') }
            </button>

			<div class="relative" bind:this={notificationRef}>
				<button
					class="p-2 rounded-xl transition-all {isDarkMode ? 'text-gray-400 hover:bg-gray-800 hover:text-white' : 'text-gray-500 hover:bg-gray-100 hover:text-indigo-600'}"
					on:click={toggleNotificationPanel}
				>
					<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
					</svg>
					{#if notificationCount > 0}
						<span class="absolute top-2 right-2 h-2.5 w-2.5 bg-red-500 border-2 {isDarkMode ? 'border-gray-900' : 'border-white'} rounded-full animate-pulse"></span>
					{/if}
				</button>
			</div>

			<button
				class="p-2 rounded-xl transition-all {isDarkMode ? 'text-yellow-400 hover:bg-gray-800' : 'text-gray-500 hover:bg-gray-100 hover:text-yellow-500'}"
				on:click={toggleDarkMode}
			>
				{#if isDarkMode}
					<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
					</svg>
				{:else}
					<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" />
					</svg>
				{/if}
			</button>

			<div class="h-8 w-[1px] mx-2 {isDarkMode ? 'bg-gray-700' : 'bg-gray-200 hidden sm:block'}"></div>

			<div class="relative" id="user-dropdown-container">
				<button
					class="flex items-center gap-2 p-1 pr-3 rounded-full transition-all {isDarkMode ? 'hover:bg-gray-800' : 'hover:bg-gray-100'}"
					on:click={toggleUserDropdown}
				>
					<div class="h-8 w-8 rounded-full bg-gradient-to-tr from-indigo-500 to-purple-500 p-0.5 overflow-hidden ring-2 ring-transparent group-hover:ring-indigo-300 transition-all">
						<img src="/static/student-avatar.png" alt="Avatar" class="h-full w-full object-cover rounded-full bg-white" />
					</div>
					<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 {isDarkMode ? 'text-gray-400' : 'text-gray-500'} {showUserDropdown ? 'rotate-180' : ''} transition-transform" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
					</svg>
				</button>

				{#if showUserDropdown}
					<div
						class="absolute right-0 mt-3 w-56 rounded-2xl shadow-xl border overflow-hidden animate-slideIn
						{isDarkMode ? 'bg-gray-800 border-gray-700' : 'bg-white border-gray-100'}"
					>
						<div class="p-4 border-b {isDarkMode ? 'border-gray-700' : 'border-gray-100'} bg-gradient-to-r {isDarkMode ? 'from-gray-800 to-gray-700' : 'from-gray-50 to-white'}">
							<p class="font-bold truncate">{$user.name}</p>
							<p class="text-xs text-gray-500 truncate">{$user.email}</p>
						</div>
						<div class="p-2">
							<a href="/student/settings" class="flex items-center gap-3 px-3 py-2 text-sm rounded-xl transition-colors {isDarkMode ? 'text-gray-300 hover:bg-gray-700' : 'text-gray-600 hover:bg-indigo-50 hover:text-indigo-600'}">
								<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" /></svg>
								{$i18n.t('Profil')}
							</a>
							<button
								on:click={() => { localStorage.removeItem('token'); location.href = '/auth'; }}
								class="w-full flex items-center gap-3 px-3 py-2 text-sm text-red-500 rounded-xl transition-colors {isDarkMode ? 'hover:bg-red-950/30' : 'hover:bg-red-50'}"
							>
								<svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" /></svg>
								{$i18n.t('Déconnexion')}
							</button>
						</div>
					</div>
				{/if}
			</div>
		</div>
	</div>
</header>

<style>
	@keyframes slideIn {
		from { opacity: 0; transform: translateY(10px); }
		to { opacity: 1; transform: translateY(0); }
	}
	.animate-slideIn {
		animation: slideIn 0.2s ease-out forwards;
	}
</style>