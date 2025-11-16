<script lang="ts">
	import Layout from '../../layout/layout.svelte';
	import { onMount } from 'svelte';

	interface Event {
		id: number;
		title: string;
		event_date: string;
		event_time: string;
		location: string;
		description: string;
	}

	let events: Event[] = [];
	let selectedEvent: Event | null = null;
	let loading = false;
	let error = '';

	const API_BASE = 'https://slbcph.site';

	function convertToStandardTime(time24: string): string {
		const [hourStr, minute, second] = time24.split(':');
		let hour = parseInt(hourStr, 10);
		const ampm = hour >= 12 ? 'PM' : 'AM';
		hour = hour % 12;
		if (hour === 0) hour = 12;
		return `${hour}:${minute} ${ampm}`;
	}

	function formatEventTime(timeRange: string): string {
		if (!timeRange) return '';
		const times = timeRange.split(' - ');
		if (times.length !== 2) return timeRange;
		const start = convertToStandardTime(times[0]);
		const end = convertToStandardTime(times[1]);
		return `${start} - ${end}`;
	}

	async function fetchEvents() {
		const token = localStorage.getItem('authToken');

		if (!token) {
			error = 'You must be logged in to view events.';
			return;
		}

		loading = true;
		try {
			const res = await fetch(`${API_BASE}/api/user/upcoming-events`, {
				headers: {
					Authorization: `Bearer ${token}`,
					Accept: 'application/json'
				}
			});

			if (!res.ok) {
				const body = await res.text();
				throw new Error(`HTTP ${res.status}: ${body}`);
			}

			const data = await res.json();
			if (!data.events) {
				events = [];
			} else {
				events = Array.isArray(data.events) ? data.events : [];
			}
			error = '';
		} catch (err: any) {
			error = err.message ?? 'Failed to fetch events.';
		} finally {
			loading = false;
		}
	}

	const openModal = (event: Event) => (selectedEvent = event);
	const closeModal = () => (selectedEvent = null);

	onMount(fetchEvents);
</script>

<Layout class={selectedEvent ? 'blur' : ''}>
	<div slot="header">Upcoming Events</div>

	<div class="rounded-lg bg-white p-6 text-lg shadow-md md:text-lg lg:text-xl">
		<p class="mb-4 text-gray-600">Stay updated with the latest church events.</p>

		{#if loading}
			<p class="text-center text-gray-500">Loading events...</p>
		{:else if error}
			<p class="text-center text-red-500">{error}</p>
		{:else if events.length === 0}
			<p class="text-center text-gray-500">No upcoming events.</p>
		{:else}
			<div class="space-y-4">
				{#each events as event}
					<div
						class="flex flex-col space-y-2 overflow-hidden rounded-lg bg-gray-100 p-4 sm:flex-row sm:items-center sm:justify-between sm:space-y-0"
					>
						<div class="min-w-0 text-xl md:text-xl lg:text-2xl">
							<h3 class="truncate font-semibold">{event.title}</h3>
							<p class="truncate text-sm text-gray-500">
								{event.event_date} | {formatEventTime(event.event_time)}
							</p>
							<p class="truncate text-sm text-gray-500">{event.location}</p>
						</div>
						<div class="shrink-0">
							<button
								on:click={() => openModal(event)}
								class="w-full transform cursor-pointer rounded-lg bg-blue-600 px-6 py-2 text-base font-semibold text-white transition-transform
                  duration-300 hover:scale-105 hover:bg-blue-700 sm:w-auto md:text-base lg:text-lg"
							>
								View Details
							</button>
						</div>
					</div>
				{/each}
			</div>
		{/if}
	</div>
</Layout>

{#if selectedEvent}
	<!-- svelte-ignore a11y_click_events_have_key_events -->
	<!-- svelte-ignore a11y_no_static_element_interactions -->
	<div
		class="fixed inset-0 z-[9999] flex items-center justify-center backdrop-brightness-80"
		on:click={closeModal}
	>
		<div
			class="z-[10000] mx-4 w-full max-w-md overflow-hidden rounded-lg bg-white p-6 break-words shadow-lg"
			on:click|stopPropagation
		>
			<div class="mb-4 flex items-center justify-between border-b pb-2">
				<div class="max-w-full overflow-x-auto">
					<h3 class="text-lg font-bold whitespace-nowrap">{selectedEvent.title}</h3>
				</div>

				<button
					on:click={closeModal}
					class="text-2xl leading-none text-gray-600 hover:text-gray-900">&times;</button
				>
			</div>

			<div class="space-y-2 text-sm break-words text-gray-800 md:text-lg lg:text-xl">
				<p><strong>Date:</strong> {selectedEvent.event_date}</p>
				<p><strong>Time:</strong> {formatEventTime(selectedEvent.event_time)}</p>
				<p><strong>Location:</strong> {selectedEvent.location}</p>
				<p class="break-words whitespace-pre-wrap">{selectedEvent.description}</p>
			</div>

			<button
				on:click={closeModal}
				class="mt-4 w-full transform cursor-pointer rounded-lg bg-red-500 px-6 py-2 font-semibold text-white
         transition-transform duration-300 hover:scale-105 hover:bg-red-600 sm:w-auto"
			>
				Close
			</button>
		</div>
	</div>
{/if}
