<script lang="ts">
	import { onMount } from 'svelte';
	import Swal from 'sweetalert2';

	/* ────────── state ────────── */
	let username = '';
	let password = '';
	let rememberMe = false;
	let showPassword = false;
	let errorMessage = '';
	let showErrorPopup = false;
	let forgotEmail = '';
	let showForgotModal = false;
	let forgotStatusMessage = '';
	let forgotErrorMessage = '';
	let isLoading = false;
	let resetSuccess = false;

	let countdown = 0; // seconds left
	let attemptCount = 0; // purely for UI
	let timerId: any; // interval handle

	function startCountdown(sec: any) {
		clearInterval(timerId);
		countdown = sec;

		timerId = setInterval(() => {
			countdown -= 1;
			if (countdown <= 0) clearInterval(timerId);
		}, 1000);
	}

	onMount(() => {
		const rememberedUsername = localStorage.getItem('rememberedUsername');
		if (rememberedUsername) {
			username = rememberedUsername;
			forgotEmail = rememberedUsername;
			rememberMe = true;
		}

		const params = new URLSearchParams(window.location.search);
		if (params.get('reset') === 'success') {
			resetSuccess = true;
			history.replaceState({}, document.title, window.location.pathname);
		}
	});

	async function login() {
		errorMessage = '';
		showErrorPopup = false;

		Swal.fire({
			title: 'Logging in…',
			allowOutsideClick: false,
			allowEscapeKey: false,
			didOpen: () => Swal.showLoading()
		});

		try {
			const response = await fetch('http://localhost:8000/api/login', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ username, password })
			});

			const data = await response.json();

			Swal.close();

			if (response.status === 429) {
				const waitSecs = data.retry_after ?? 60;
				startCountdown(waitSecs);

				errorMessage = data.message ?? 'Too many login attempts.';
				showErrorPopup = true;

				if (data.recommend_reset) {
					showForgotModal = true;
					forgotEmail = username;
				}
				return;
			}

			if (!response.ok) {
				errorMessage = data.message ?? 'Login failed';
				showErrorPopup = true;
				return;
			}

			localStorage.setItem('authToken', data.token);
			localStorage.setItem('user', JSON.stringify(data.user));
			localStorage.setItem('branchId', data.user.branch_id);

			if (rememberMe) {
				localStorage.setItem('rememberedUsername', username);
			} else {
				localStorage.removeItem('rememberedUsername');
			}

			await Swal.fire({
				icon: 'success',
				title: 'Login successful!',
				timer: 1200,
				showConfirmButton: false
			});

			if (data.user.requires_terms_acceptance) {
				window.location.href = '/terms';
			} else {
				window.location.href = '/members/memberdb';
			}
		} catch (err) {
			Swal.close();
			errorMessage = err instanceof Error ? err.message : 'An unknown error occurred';
			showErrorPopup = true;
		}
	}

	async function submitForgotPassword() {
		forgotStatusMessage = '';
		forgotErrorMessage = '';
		isLoading = true; // ⏳ start loading

		try {
			const response = await fetch('https://slbcph.site/api/forgot-password', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email: forgotEmail })
			});

			const data = await response.json();

			if (!response.ok) {
				throw new Error(data.message || 'Failed to send reset link');
			}

			forgotStatusMessage = 'Password reset link sent! Please check your email.';
		} catch (err) {
			forgotErrorMessage = err instanceof Error ? err.message : 'An error occurred';
		} finally {
			isLoading = false;
		}
	}
</script>

<svelte:head>
	<title>Shining Light Baptist Church</title>
</svelte:head>

<div
	class="fixed inset-0"
	style="background:url('/logo.png') center/100% no-repeat; opacity:0.10; z-index:0;"
></div>

<div
	class="absolute top-1/2 left-1/2 z-10 flex min-h-[500px] w-full max-w-5xl -translate-x-1/2 -translate-y-1/2 transform flex-col overflow-hidden rounded-lg bg-white text-gray-800 shadow-lg md:flex-row"
>
	<div
		class="relative flex w-full items-center justify-center bg-[#002B5C] p-6 text-white md:w-1/2 md:p-10"
	>
		<img src="/logo.png" alt="" class="absolute inset-0 h-full w-full object-cover opacity-20" />
		<div class="relative z-10 px-4 text-center md:px-0">
			<h1 class="mb-3 text-2xl font-bold md:mb-4 lg:text-3xl">Welcome!</h1>
			<p class="mb-4 text-xl md:mb-6 lg:text-2xl">Bible verse for welcoming a person</p>
			<p class="text-sm leading-relaxed font-semibold italic md:text-base lg:text-lg">
				"Therefore welcome one another as Christ has welcomed you, for the glory of God." –<br />
				Romans&nbsp;15:7
			</p>
		</div>
	</div>

	<div class="flex w-full flex-col justify-center bg-gray-100 p-6 md:w-1/2 md:p-10">
		<h2 class="mb-4 text-center text-2xl font-bold text-[#002B5C] lg:text-3xl">Member Login</h2>

		{#if resetSuccess}
			<div class="mb-4 rounded border border-green-300 bg-green-100 p-3 text-center text-green-800">
				Your password was reset successfully. Please log in.
			</div>
		{/if}

		<form on:submit|preventDefault={login} class="space-y-4">
			<div>
				<label for="username" class="block text-lg font-medium text-gray-700 lg:text-xl"
					>Username</label
				>
				<input
					id="username"
					type="text"
					bind:value={username}
					required
					class="w-full rounded-lg border px-4 py-2 focus:ring-2 focus:ring-[#0054A4] focus:outline-none"
				/>
			</div>
	<label for="password" class="block text-lg font-medium text-gray-700 lg:text-xl">
		Password
	</label>

	<div class="relative">
		<input
			id="password"
			type={showPassword ? 'text' : 'password'}
			bind:value={password}
			required
			class="w-full rounded-lg border px-4 py-2 pr-10 focus:ring-2 
                   focus:ring-[#0054A4] focus:outline-none"
		/>

		<button
			type="button"
			class="absolute inset-y-0 right-2 flex items-center h-full text-gray-600"
			on:click={() => (showPassword = !showPassword)}
			tabindex="-1"
		>
					{#if showPassword}
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-5 w-5"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
							/>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M2.458 12C3.732 7.943 7.523 5 12 5s8.268 2.943 9.542 7c-1.274 4.057-5.065 7-9.542 7s-8.268-2.943-9.542-7z"
							/>
						</svg>
					{:else}
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-5 w-5"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.542-7a9.953 9.953 0 012.42-3.737m1.354-1.238A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.542 7a9.958 9.958 0 01-4.042 5.132M15 12a3 3 0 00-3-3m0 0a3 3 0 00-3 3m6 0a3 3 0 01-3 3m0 0a3 3 0 01-3-3m0 0L3 3"
							/>
						</svg>
					{/if}
				</button>
			</div>

			<div class="flex items-center justify-between text-sm">
				<label class="inline-flex items-center">
					<input type="checkbox" bind:checked={rememberMe} class="form-checkbox text-[#0054A4]" />
					<span class="ml-2 text-base text-gray-600 lg:text-lg">Remember me</span>
				</label>
				<button
					class="text-base text-[#0054A4] hover:underline"
					on:click={() => (showForgotModal = true)}
					>Forgot Password?
				</button>
			</div>

			<button
				type="submit"
				class="w-full rounded-lg bg-[#E50914] py-2 text-base font-semibold
               text-white transition hover:bg-[#b90710] disabled:opacity-60 lg:text-lg"
				disabled={countdown > 0}
			>
				{#if countdown > 0}
					Retry in {countdown}s
				{:else}
					Login
				{/if}
			</button>
		</form>
	</div>
</div>

<!-- Error Modal -->
{#if showErrorPopup}
	<div class="fixed inset-0 z-50 flex items-center justify-center backdrop-brightness-45">
		<div class="w-full max-w-sm rounded-lg bg-white p-6 shadow-lg">
			<h3 class="text-lg font-bold text-red-600">Login Failed</h3>
			<p class="mt-2 text-gray-700">{errorMessage}</p>
			<button
				class="mt-4 rounded bg-red-600 px-4 py-2 text-white hover:bg-red-700"
				on:click={() => (showErrorPopup = false)}
			>
				Close
			</button>
		</div>
	</div>
{/if}

<!-- Forgot Password Modal -->
{#if showForgotModal}
	<div class="fixed inset-0 z-50 flex items-center justify-center backdrop-brightness-50">
		<div class="w-full max-w-md rounded-lg bg-white p-6 shadow-lg">
			<h3 class="mb-2 text-lg font-bold text-[#002B5C]">Forgot Password</h3>
			<p class="mb-4 text-sm text-gray-600">Enter your email to receive a password reset link.</p>

			<input
				type="email"
				placeholder="Email address"
				bind:value={forgotEmail}
				class="w-full rounded-lg border px-4 py-2 focus:ring-2 focus:ring-[#0054A4] focus:outline-none"
			/>

			{#if forgotStatusMessage}
				<p class="mt-2 text-sm text-green-600">{forgotStatusMessage}</p>
			{/if}
			{#if forgotErrorMessage}
				<p class="mt-2 text-sm text-red-600">{forgotErrorMessage}</p>
			{/if}

			<div class="mt-4 flex justify-end space-x-2">
				<button
					class="rounded bg-gray-300 px-4 py-2 text-gray-700 hover:bg-gray-400"
					on:click={() => (showForgotModal = false)}
					disabled={isLoading}
				>
					Cancel
				</button>
				<button
					class="flex items-center justify-center rounded bg-[#0054A4] px-4 py-2 text-white hover:bg-[#003f91]"
					on:click={submitForgotPassword}
					disabled={isLoading}
				>
					{#if isLoading}
						<svg
							class="mr-2 h-5 w-5 animate-spin text-white"
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
						>
							<circle
								class="opacity-25"
								cx="12"
								cy="12"
								r="10"
								stroke="currentColor"
								stroke-width="4"
							></circle>
							<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"
							></path>
						</svg>
						Sending...
					{:else}
						Send Link
					{/if}
				</button>
			</div>
		</div>
	</div>
{/if}
