<script lang="ts">
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';
	import Swal from 'sweetalert2';

	let sidebarOpen = false;
	let profileMenuOpen = false;
	let currentPage = '';
	let memberName = '';
	let profileImage = '';
	let branchName = '';

	let currentTime = writable('');

	const updateTime = () => {
		const now = new Date();
		const options: Intl.DateTimeFormatOptions = {
			weekday: 'long',
			year: 'numeric',
			month: 'long',
			day: 'numeric',
			hour: '2-digit',
			minute: '2-digit',
			second: '2-digit',
			hour12: true,
			timeZone: 'Asia/Manila'
		};
		const formattedTime = now.toLocaleString('en-PH', options);
		currentTime.set(formattedTime);
	};

	onMount(() => {
		setInterval(updateTime, 1000);
		const path = window.location.pathname;
		if (path.includes('memberdb')) currentPage = 'dashboard';
		else if (path.includes('events')) currentPage = 'events';
		else if (path.includes('ssm')) currentPage = 'ssm';
		else if (path.includes('financial')) currentPage = 'financial';
		else if (path.includes('pr')) currentPage = 'prayer';

		checkTermsAcceptance();
		fetchMemberData();
	});

	async function checkTermsAcceptance() {
		const token = localStorage.getItem('authToken');
		const user = JSON.parse(localStorage.getItem('user') || '{}');

		// If not authenticated, redirect to login
		if (!token || !user.id) {
			window.location.href = '/login';
			return;
		}

		// If on terms page, allow access
		if (window.location.pathname === '/terms') {
			return;
		}

		// Check if terms need to be accepted
		try {
			const response = await fetch('https://slbcph.site/api/user', {
				headers: { Authorization: `Bearer ${token}` }
			});

			if (response.ok) {
				const userData = await response.json();
				if (userData.requires_terms_acceptance) {
					window.location.href = '/terms';
					return;
				}
			}
		} catch (error) {
			console.error('Error checking terms acceptance:', error);
		}
	}

	async function fetchMemberData() {
		const token = localStorage.getItem('authToken');
		if (!token) return;
		try {
			const response = await fetch('https://slbcph.site/api/user', {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (response.ok) {
				const memberData = await response.json();
				memberName = `${memberData.first_name} ${memberData.last_name}`;
				profileImage = memberData.profile_image;

				fetchBranchName(memberData.branch_id);
			}
		} catch (error) {
			console.error('Error fetching member data:', error);
		}
	}

	async function fetchBranchName(branchId: any) {
		try {
			const response = await fetch(`https://slbcph.site/api/branches/${branchId}`);
			if (response.ok) {
				const branchData = await response.json();
				branchName = branchData.name;
			}
		} catch (error) {
			console.error('Error fetching branch name:', error);
		}
	}

	function toggleSidebar() {
		sidebarOpen = !sidebarOpen;
	}

	function toggleProfileMenu() {
		profileMenuOpen = !profileMenuOpen;
	}

	function closeSidebar() {
		sidebarOpen = false;
	}

	async function logout() {
		const token = localStorage.getItem('authToken') || sessionStorage.getItem('authToken');
		const response = await fetch('https://slbcph.site/api/logout', {
			method: 'POST',
			headers: {
				Authorization: `Bearer ${token}`
			}
		});
		if (response.ok) {
			localStorage.removeItem('authToken');
			sessionStorage.removeItem('authToken');
			window.location.href = '/';
		}
	}
</script>

<link
	rel="stylesheet"
	href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css"
/>

<div class="flex">
	<aside class="sidebar text-white" class:open={sidebarOpen}>
		<!-- svelte-ignore a11y_consider_explicit_label -->
		<button class="mb-4 rounded p-2 text-white hover:bg-red-600 md:hidden" on:click={toggleSidebar}>
			<i class="fas fa-times"></i>
		</button>

		<div class="mt-2 mb-4 flex flex-col items-center">
			<img src="/logo.png" alt="Logo" class="w-32 max-w-full" />

			<div class="mt-2 text-center">
				<p class="text-xl leading-tight font-bold text-white md:text-xl lg:text-2xl">
					Shining Light Baptist Church
				</p>
				{#if branchName}
					<p class="mt-1 text-base text-white">Branch: {branchName}</p>
				{/if}
			</div>
		</div>

		<div class="divider">
			<nav class="text-base font-semibold md:text-base lg:text-xl">
				<ul class="space-y-2">
					<li>
						<a
							href="../../members/memberdb"
							class={`flex items-center space-x-2 rounded p-3 ${currentPage === 'dashboard' ? 'dashboard-highlight' : ''}`}
						>
							<i class="fas fa-home"></i><span>Dashboard</span>
						</a>
					</li>
					<li>
						<a
							href="../../members/events"
							class={`flex items-center space-x-2 rounded p-3 ${currentPage === 'events' ? 'highlight' : ''}`}
						>
							<i class="fas fa-calendar-alt"></i><span>Events</span></a
						>
					</li>
					<li>
						<a
							href="../../members/ssm"
							class={`flex items-center space-x-2 rounded p-3 ${currentPage === 'ssm' ? 'highlight' : ''}`}
						>
							<i class="fas fa-clipboard-check"></i><span>Service Monitoring</span></a
						>
					</li>
					<li>
						<a
							href="../../members/financial"
							class={`flex items-center space-x-2 rounded p-3 ${currentPage === 'financial' ? 'highlight' : ''}`}
						>
							<i class="fas fa-hand-holding-heart"></i><span>Track Financial</span></a
						>
					</li>
					<li>
						<a
							href="../../members/pr"
							class={`flex items-center space-x-2 rounded p-3 ${currentPage === 'prayer' ? 'highlight' : ''}`}
						>
							<i class="fas fa-pray"></i><span>Prayer Requests</span></a
						>
					</li>
				</ul>
			</nav>
		</div>
	</aside>

	<!-- svelte-ignore a11y_consider_explicit_label -->
	<button class="overlay {sidebarOpen ? 'show' : ''}" on:click={closeSidebar}></button>

	<div class="content relative min-h-screen overflow-x-auto p-6">
		<div
			class="absolute inset-0 z-0 bg-cover bg-center opacity-10"
			style="background-image: url('/logo.png');"
		></div>

		<div class="relative z-10">
			<div
	class="header mb-6 flex items-center justify-between rounded bg-gray-800 px-4 py-3 shadow text-white"
>
	<!-- Mobile Toggle -->
	<!-- svelte-ignore a11y_consider_explicit_label -->
	<button class="p-2 text-white md:hidden" on:click={toggleSidebar}>
		<i class="fas fa-bars text-xl"></i>
	</button>

	<!-- Left Side: Header Title -->
      <div class="flex min-w-0 text-2xl font-bold items-center gap-2 truncate">
        <slot name="header" />
      </div>

      <div class="flex items-center gap-4 whitespace-nowrap">
        <div class="hidden text-sm font-medium sm:block md:text-base">
          {$currentTime}
        </div>

        <!-- Profile -->
        <div class="relative">
          <button
            class="flex items-center gap-2 rounded-full bg-gray-700 px-2 py-1 transition hover:bg-gray-600"
            on:click={toggleProfileMenu}
          >
            <div class="h-8 w-8 overflow-hidden rounded-full border border-white">
              <img
                src={profileImage || '/placeholder.png'}
                alt="Profile"
                class="h-full w-full object-cover"
              />
            </div>

            <!-- Name + Arrow (hidden on mobile) -->
            <div class="hidden items-center gap-1 md:flex">
              <span class="text-sm lg:text-base">{memberName || 'Member Name'}</span>
              <i class="fas fa-caret-down text-sm"></i>
            </div>
          </button>

          {#if profileMenuOpen}
            <div
              class="absolute right-0 z-50 mt-2 w-40 rounded-lg bg-white py-2 text-gray-800 shadow-lg"
            >
              <a
                href="/members/up"
                class="block px-4 py-2 text-sm hover:bg-gray-100"
              >
                <i class="fas fa-user-circle mr-2"></i> Profile
              </a>
              <a
                href="/"
                on:click={logout}
                class="block px-4 py-2 text-sm hover:bg-gray-100"
              >
                <i class="fas fa-sign-out-alt mr-2"></i> Logout
              </a>
            </div>
          {/if}
        </div>
      </div>
    </div>


			<slot />
		</div>
	</div>
</div>

<style>
	.sidebar {
		background-color: #003366;
		color: black;
		transition: transform 0.3s ease-in-out;
		position: fixed;
		top: 0;
		left: 0;
		height: 100vh;
		width: 16rem;
		overflow-y: auto;
		padding: 0 1rem 1rem 1rem; /* changed: removed top padding */
		z-index: 1000;
	}

	.sidebar a {
		color: white;
	}

	.sidebar a.highlight {
		background-color: #2563eb;
		font-weight: medium;
	}

	.sidebar a.active {
		color: white;
	}

	.dashboard-highlight {
		background-color: #2563eb;
		font-weight: bold;
		color: black;
	}

	.dashboard-options {
		background-color: white;
		padding: 10px;
		border-radius: 8px;
		margin-top: 10px;
	}

	.divider {
		height: auto;
		width: 100%;
		padding: 1px;
		margin-top: 50px;
	}

	.content {
		background-color: #f0f0f0;
		margin-left: 16rem;
		width: calc(100% - 16rem);
		height: 100vh;
		overflow-y: auto;
		position: relative;
	}

	.overlay {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.5);
		z-index: 900;
		display: none;
	}

	.overlay.show {
		display: block;
	}

	.header {
		background: #003366;
		color: white;
		padding: 12px 16px;
		z-index: 1001;
		position: relative;
		transition:
			background 0.3s ease,
			transform 0.2s ease,
			box-shadow 0.3s ease;
		cursor: pointer; /* show pointer on hover */
	}

	.profile-placeholder {
		width: 50px;
		height: 50px;
		background: #ccc;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 50%;
		font-size: 1.5rem;
		color: black;
	}

	.profile-dropdown {
		position: absolute;
		right: 0;
		top: 50px;
		background: white;
		border-radius: 8px;
		box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
		width: 160px;
	}

	.datetime-container {
		font-size: 0.75rem; /* make the date smaller in mobile */
		max-width: 150px; /* optional: limit width */
		word-wrap: break-word;
	}

	.header-right {
		display: flex;
		justify-content: flex-end;
		align-items: center;
    min-width: 500px;
	}

	@media (max-width: 768px) {
		.sidebar {
			transform: translateX(-100%);
		}

		.sidebar.open {
			transform: translateX(0);
		}

		.content {
			margin-left: 0;
			width: 100%;
		}

		.branch-in-sidebar {
			display: block;
		}

		.branch-in-header {
			display: none;
		}
	}

	@media (min-width: 769px) {
		.branch-in-sidebar {
			display: none;
		}

		.branch-in-header {
			display: flex;
		}
	}
</style>
