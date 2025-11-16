<script lang="ts">
	import Swal from 'sweetalert2';
	import Layout from '../../layout/layout.svelte';
	import { onMount } from 'svelte';

	let requestType = 'Prayer Request';
	let requestMessage = '';
	let submissions: any = [];
	let allSubmissions: any = [];
	let loading = false;
	let error = '';
	let showAllSubmissions = false;

	let startDate = '';
	let endDate = '';

	let currentPage = 1;
	const itemsPerPage = 5;

	$: totalPages = Math.ceil(submissions.length / itemsPerPage);
	$: paginatedSubmissions = submissions.slice(
		(currentPage - 1) * itemsPerPage,
		currentPage * itemsPerPage
	);

	function goToPage(page: any) {
		if (page >= 1 && page <= totalPages) {
			currentPage = page;
		}
	}
	$: if (currentPage > totalPages) currentPage = 1;

	const API_URL = 'https://slbcph.site/api';

	let showModal = false;
	let selectedMessage = '';

	async function fetchRequests() {
		const token = localStorage.getItem('authToken'); 

		if (!token) {
			error = 'You must be logged in to view prayer requests.';
			return;
		}

		loading = true;
		try {
			const res = await fetch(`${API_URL}/user/prayer-requests`, {
				headers: { Authorization: `Bearer ${token}` }
			});

			if (!res.ok) {
				const errorData = await res.json();
				throw new Error(errorData.message || 'Failed to load prayer requests.');
			}

			const data = await res.json();
			allSubmissions = data.map((req: any) => ({
				date: new Date(req.created_at),
				type: req.type,
				message: req.request,
				status: req.status
			}));

			// Filter submissions to only include those from the last week
			const oneWeekAgo = new Date();
			oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
			submissions = allSubmissions.filter((req: any) => req.date >= oneWeekAgo);
		} catch (err: any) {
			error = err.message; // Use the error message from the API
		} finally {
			loading = false;
		}
	}

	// 🔹 Submit Prayer Request
	async function submitRequest() {
		if (requestMessage.trim() === '') return;

		const token = localStorage.getItem('authToken');

		if (!token) {
			error = 'You must be logged in to submit a prayer request.';
			return;
		}

		try {
			const res = await fetch(`${API_URL}/prayer-requests`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Bearer ${token}`
				},
				body: JSON.stringify({ type: requestType, request: requestMessage })
			});

			if (!res.ok) {
				const errorData = await res.json();
				throw new Error(errorData.message || 'Failed to submit request.');
			}

			const newRequest = {
				date: new Date(),
				type: requestType,
				message: requestMessage,
				status: 'Pending'
			};

			submissions = [newRequest, ...submissions];
			allSubmissions = [newRequest, ...allSubmissions];
			requestMessage = '';

			// ✅ Show success alert
			Swal.fire({
				icon: 'success',
				title: 'Submitted!',
				text: 'Your request has been submitted successfully.',
				timer: 1500,
				showConfirmButton: false
			});
		} catch (err: any) {
			error = err.message;
		}
	}

	// 🔹 Filter submissions by date range
	function filterByDate() {
		if (startDate && endDate) {
			const start = new Date(startDate);
			const end = new Date(endDate);
			submissions = allSubmissions.filter((req: any) => req.date >= start && req.date <= end);
		} else {
			submissions = allSubmissions; // Reset to all submissions if no date range is selected
		}
	}

	// 🔹 Toggle to show all submissions
	function toggleShowAll() {
		showAllSubmissions = !showAllSubmissions;
		submissions = showAllSubmissions
			? allSubmissions
			: allSubmissions.filter(
					(req: any) => req.date >= new Date(new Date().setDate(new Date().getDate() - 7))
				);
	}

	function openModal(message: string) {
		selectedMessage = message;
		showModal = true;
	}

	// 🔹 Close modal
	function closeModal() {
		showModal = false;
		selectedMessage = '';
	}

	// 🔹 Auto-load prayer requests on page load
	onMount(fetchRequests);
</script>

<head>
	<title>Shining Light Baptist Church</title>
</head>

<Layout>
	<div slot="header">Prayer Request and Blessing</div>

	<div class="rounded-lg bg-white p-6 shadow-md">
		<h2 class="text-xl font-semibold md:text-xl lg:text-2xl">
			Submit Your Prayer Request and Blessing
		</h2>
		<p class="text-base text-gray-600 md:text-base lg:text-lg">
			Submit your prayer requests and blessings for this week
		</p>

		<div class="mt-4">
			<select
				bind:value={requestType}
				class="w-full rounded border p-2 text-base md:text-base lg:text-lg"
			>
				<option value="Prayer Request">Prayer Request</option>
				<option value="Blessing">Blessing</option>
				<option value="Reflection">Reflection</option>
			</select>
			<textarea
				bind:value={requestMessage}
				class="mt-2 w-full rounded border p-2 text-lg md:text-lg lg:text-xl"
				rows="3"
				placeholder="Write your prayer request or blessing here..."
			></textarea>
			<button
				on:click={submitRequest}
				class="mt-2 cursor-pointer rounded bg-blue-500 px-4 py-2 text-base text-white transition-transform duration-300 hover:scale-[1.02] hover:bg-blue-600 md:text-base lg:text-lg"
			>
				Submit
			</button>
		</div>
	</div>

	<div class="mt-6 rounded-lg bg-white p-6 text-lg shadow-md md:text-lg lg:text-xl">
		{#if loading}
			<p class="text-center text-gray-500">Loading...</p>
		{:else if error}
			<p class="text-center text-red-500">{error}</p>
		{:else if submissions.length === 0}
			<p class="text-center text-gray-500">No prayer requests submitted yet.</p>
		{:else}
			<div class="scrollable mt-4">
				<table class="w-full border-collapse border">
					<thead>
						<tr class="bg-gray-200">
							<th class="border p-2 text-center">Date Submitted</th>
							<th class="border p-2 text-center">Type</th>
							<th class="border p-2 text-center">Message</th>
							<th class="border p-2 text-center">Status</th>
						</tr>
					</thead>
					<tbody>
						{#each paginatedSubmissions as submission}
							<tr>
								<td class="border p-2 text-center">{submission.date.toLocaleDateString()}</td>
								<td class="border p-2 text-center">{submission.type}</td>
								<td
									class="border p-2 text-center"
									title="Click to view full message"
									on:click={() => openModal(submission.message)}
								>
									<span class="block w-full cursor-pointer truncate text-blue-600 hover:underline">
										{submission.message.length > 50
											? `${submission.message.slice(0, 50)}...`
											: submission.message}
									</span>
								</td>
								<td
									class="border p-2 text-center"
									class:text-yellow-500={submission.status === 'Pending'}
									class:text-green-500={submission.status === 'Reviewed'}
								>
									{submission.status}
								</td>
							</tr>
						{/each}
					</tbody>
				</table>

				<div class="mt-4 flex items-center justify-center gap-2">
					<button
						class="cursor-pointer rounded bg-gray-200 px-3 py-1 hover:bg-gray-300"
						on:click={() => goToPage(currentPage - 1)}
						disabled={currentPage === 1}
					>
						Prev
					</button>
					{#each Array(totalPages) as _, i}
						<button
							class="rounded px-3 py-1 {currentPage === i + 1
								? 'bg-blue-500 text-white'
								: 'bg-gray-200 hover:bg-gray-300'}"
							on:click={() => goToPage(i + 1)}
						>
							{i + 1}
						</button>
					{/each}
					<button
						class="cursor-pointer rounded bg-gray-200 px-3 py-1 hover:bg-gray-300"
						on:click={() => goToPage(currentPage + 1)}
						disabled={currentPage === totalPages}
					>
						Next
					</button>
				</div>
			</div>
		{/if}
	</div>
</Layout>

{#if showModal}
	<div class="modal-overlay">
		<div
			class="relative max-h-[80vh] w-full max-w-lg overflow-y-auto rounded-lg border border-gray-300 bg-white p-6 shadow-lg"
		>
			<button
				class="absolute top-2 right-2 cursor-pointer text-2xl font-bold text-gray-500 hover:text-red-600"
				on:click={closeModal}>&times;</button
			>
			<h3 class="mb-4 text-lg font-semibold">Full Message</h3>
			<div class="break-words whitespace-pre-line">{selectedMessage}</div>
		</div>
	</div>
{/if}

<style>
	.modal-overlay {
		position: fixed;
		inset: 0;
		z-index: 1000;
		display: flex;
		align-items: center;
		justify-content: center;
		background: rgba(0, 0, 0, 0.5); /* Darker background for better contrast */
		backdrop-filter: blur(0.5px); /* Increased blur effect */
	}
	.scrollable {
		max-width: 100%;
		overflow-x: auto;
		white-space: nowrap;
	}
	.truncate {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		max-width: 250px;
		width: 100%;
		display: block;
		margin: 0 auto;
	}
</style>
