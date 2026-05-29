<script lang="ts">
	import '../app.postcss';
	import { page } from '$app/stores';
	import { onMount } from 'svelte';
	import {
		DarkMode,
		Navbar,
		NavBrand,
		NavLi,
		NavUl,
		NavHamburger,
		Sidebar,
		SidebarGroup,
		SidebarItem,
		SidebarWrapper,
		Drawer,
		CloseButton,
		SidebarDropdownWrapper
	} from 'flowbite-svelte';
	import { sineIn } from 'svelte/easing';

	let { children, data } = $props();

	let transitionParams = { x: -320, duration: 200, easing: sineIn };
	let breakPoint: number = 1024;
	let width: number = $state(0);
	let backdrop: boolean = false;
	let activateClickOutside = $state(true);
	let drawerHidden: boolean = $state(false);

	$effect(() => {
		if (width >= breakPoint) {
			drawerHidden = false;
			activateClickOutside = false;
		} else {
			drawerHidden = true;
			activateClickOutside = true;
		}
	});

	onMount(() => {
		if (width >= breakPoint) {
			drawerHidden = false;
			activateClickOutside = false;
		} else {
			drawerHidden = true;
			activateClickOutside = true;
		}
	});

	const toggleSide = () => {
		if (width < breakPoint) drawerHidden = !drawerHidden;
	};
	const toggleDrawer = () => { drawerHidden = false; };

	let activeUrl = $derived($page.url.pathname);
	let spanClass = 'pl-2 self-center text-md text-gray-900 whitespace-nowrap dark:text-white';
	let darkmodebtn = 'text-gray-500 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-700 focus:outline-none focus:ring-4 focus:ring-gray-200 dark:focus:ring-gray-700 rounded-lg text-lg p-2.5 fixed right-2 top-12 md:top-3 md:right-2 z-50';
	let divClass = 'w-full md:block md:w-auto pr-8';
	let ulClass = 'flex flex-col p-4 mt-4 md:flex-row md:space-x-8 md:mt-0 md:text-lg md:font-medium';
</script>

<svelte:window bind:innerWidth={width} />

<Navbar>
	<NavHamburger onclick={toggleDrawer} btnClass="ml-3 lg:hidden" />
	<NavBrand href="/" class="lg:ml-64">
		<span class="text-xl">🏥</span>
		<span class="self-center whitespace-nowrap text-xl font-semibold dark:text-white pl-4">
			永佳診所
		</span>
	</NavBrand>
	<NavUl {divClass} {ulClass}>
		<NavLi href="/">主頁</NavLi>
		<NavLi href="/pages/about">簡介</NavLi>
		<NavLi href="/stats">門診統計</NavLi>
		<NavLi href="/chronic">三高管理</NavLi>
	</NavUl>
	
</Navbar>

<DarkMode btnClass={darkmodebtn} />

<Drawer
	transitionType="fly"
	{backdrop}
	{transitionParams}
	bind:hidden={drawerHidden}
	bind:activateClickOutside
	width="w-64"
	class="overflow-scroll pb-32"
	id="sidebar"
>
	<div class="flex items-center">
		<CloseButton onclick={() => (drawerHidden = true)} class="mb-4 dark:text-white lg:hidden" />
	</div>
	<Sidebar asideClass="w-54">
		<SidebarWrapper divClass="overflow-y-auto py-4 px-3 rounded dark:bg-gray-800">
			<SidebarGroup>
				<SidebarItem label="主頁" href="/" onclick={toggleSide} active={activeUrl === `/`} />
				{#each data.pages as { meta, path }}
					<SidebarItem
						label={meta.title}
						href={`/pages/${path}`}
						{spanClass}
						onclick={toggleSide}
						active={activeUrl === `/pages/${path}`}
					/>
				{/each}
				<SidebarDropdownWrapper label="實作工場">
					{#each data.articles as { meta, path }}
						<SidebarItem
							label={meta.title}
							href={`/blog/${path}`}
							{spanClass}
							onclick={toggleSide}
							active={activeUrl === `/blog/${path}`}
						/>
					{/each}
				</SidebarDropdownWrapper>
			</SidebarGroup>
		</SidebarWrapper>
	</Sidebar>
</Drawer>

<div class="flex px-4 mx-auto w-full">
	<main class="lg:ml-72 w-full mx-auto">
		{@render children()}
	</main>
</div>