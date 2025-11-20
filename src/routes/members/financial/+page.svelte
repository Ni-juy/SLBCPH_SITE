<script lang="ts">
	import type { Donation } from '$lib/type';
	import { Chart, registerables } from 'chart.js';
	import Layout from '../../layout/layout.svelte';
	import { onMount } from 'svelte';
	import Swal from 'sweetalert2';
	Chart.register(...registerables);

	let donations: Donation[] = [];
	let filteredDonations: Donation[] = [];
	let filterStartDate = '';
	let filterEndDate = '';
	let categoryFilter = 'all';
	let searchQuery = '';
	let filterMonth: number | null = null;
	let filterYear: number | null = null;
	let pieChartInstance: any = null;
	let barChartInstance: any = null;
	let errorMessage = '';
	let today = new Date().toISOString().split('T')[0];
	let currentBranch = '';
	let selectedBranch = 'current';
	let hasNewReport = false;
	let reportPdfLink: any = null;

	let currentPage = 1;
	const itemsPerPage = 5;

	$: totalPages = Math.ceil(displayedDonations.length / itemsPerPage);
	$: paginatedDonations = displayedDonations.slice(
		(currentPage - 1) * itemsPerPage,
		currentPage * itemsPerPage
	);
	$: if (currentPage > totalPages) currentPage = totalPages || 1;

	function goToPage(page: number) {
		if (page >= 1 && page <= totalPages) currentPage = page;
	}
	function nextPage() {
		if (currentPage < totalPages) currentPage++;
	}
	function prevPage() {
		if (currentPage > 1) currentPage--;
	}

	/* ─────────── Fetch Donations ─────────── */
	async function fetchDonations() {
		const params = new URLSearchParams();

		if (filterStartDate && filterEndDate) {
			params.append('from', filterStartDate);
			params.append('to', filterEndDate);
		}

		if (categoryFilter && categoryFilter !== 'all') {
			params.append('event', categoryFilter);
		}

		try {
			const res = await fetch(
				`https://slbcph.site/api/financial-tracking-data?${params.toString()}`,
				{
					headers: {
						Authorization: `Bearer ${localStorage.getItem('authToken')}`
					}
				}
			);

			if (!res.ok) {
				const errorText = await res.text();
				throw new Error(`HTTP error! status: ${res.status}, message: ${errorText}`);
			}

			const data = await res.json();
			donations = data.donations || [];
			filteredDonations = donations;
			currentBranch = data.currentBranch || '';
		} catch (error) {
			console.error('Failed to fetch donations', error);
		}
	}

	/* ─────────── Group by Branch ─────────── */
	function groupByBranch(donations: Donation[]): Record<string, Donation[]> {
		return donations.reduce((acc: Record<string, Donation[]>, d) => {
			const branch = d.branch_name || 'N/A';
			if (!acc[branch]) acc[branch] = [];
			acc[branch].push(d);
			return acc;
		}, {});
	}

	/* ─────────── Displayed Donations ─────────── */
	$: displayedDonations = (() => {
		const grouped = groupByBranch(filteredDonations);

		if (selectedBranch === 'all') {
			return filteredDonations;
		} else if (selectedBranch === 'current') {
			return grouped[currentBranch] || [];
		} else {
			return grouped[selectedBranch] || [];
		}
	})();

	/* ─────────── Charts ─────────── */
	function createCharts() {
		const pieCanvas = document.getElementById('pieChart') as HTMLCanvasElement;
		const barCanvas = document.getElementById('barChart') as HTMLCanvasElement;

		if (!pieCanvas || !barCanvas) return;

		const pieCtx = pieCanvas.getContext('2d');
		const barCtx = barCanvas.getContext('2d');

		// Category totals
		const categoryTotals = displayedDonations.reduce(
			(acc, donation) => {
				acc[donation.category] = (acc[donation.category] || 0) + Number(donation.amount);
				return acc;
			},
			{} as Record<string, number>
		);

		const categories = Object.keys(categoryTotals);
		const pieData = Object.values(categoryTotals);

		// Pie chart
		if (pieCtx) {
			if (pieChartInstance) pieChartInstance.destroy();
			pieChartInstance = new Chart(pieCtx, {
				type: 'pie',
				data: {
					labels: categories,
					datasets: [
						{
							label: 'Fund Allocation',
							data: pieData,
							backgroundColor: [
								'rgba(255, 99, 132, 0.2)',
								'rgba(54, 162, 235, 0.2)',
								'rgba(255, 206, 86, 0.2)',
								'rgba(75, 192, 192, 0.2)',
								'rgba(153, 102, 255, 0.2)'
							],
							borderColor: [
								'rgba(255, 99, 132, 1)',
								'rgba(54, 162, 235, 1)',
								'rgba(255, 206, 86, 1)',
								'rgba(75, 192, 192, 1)',
								'rgba(153, 102, 255, 1)'
							],
							borderWidth: 1
						}
					]
				},
				options: {
					responsive: true,
					plugins: {
						legend: { position: 'top' },
						title: { display: true, text: 'Fund Allocation Overview' }
					}
				}
			});
		}

		// Bar chart
		const monthlyData: Record<string, number> = {};
		displayedDonations.forEach((donation) => {
			const month = new Date(donation.date).toLocaleString('default', {
				month: 'long',
				year: 'numeric'
			});
			monthlyData[month] = (monthlyData[month] || 0) + Number(donation.amount);
		});

		const labels = Object.keys(monthlyData);
		const amounts = Object.values(monthlyData);

		if (barCtx) {
			if (barChartInstance) barChartInstance.destroy();
			barChartInstance = new Chart(barCtx, {
				type: 'bar',
				data: {
					labels,
					datasets: [
						{
							label: 'Total Amount Donated per Month',
							data: amounts,
							backgroundColor: 'rgba(75, 192, 192, 0.2)',
							borderColor: 'rgba(75, 192, 192, 1)',
							borderWidth: 1
						}
					]
				},
				options: {
					responsive: true,
					scales: {
						y: {
							beginAtZero: true,
							ticks: {
								stepSize: Math.ceil(Math.max(...amounts) / 5) || 1
							}
						}
					},
					plugins: {
						legend: { position: 'top' },
						title: { display: true, text: 'Monthly Donations Overview' }
					}
				}
			});
		}
	}

	/* ✅ Charts only rebuild when donations change */
	$: if (displayedDonations) {
		if (displayedDonations.length > 0) {
			createCharts();
		} else {
			if (pieChartInstance) pieChartInstance.destroy();
			if (barChartInstance) barChartInstance.destroy();
		}
	}

	/* ─────────── Date Validation ─────────── */
	function validateDates() {
		if (!filterStartDate || !filterEndDate) {
			errorMessage = 'Please select both start and end dates.';
			return false;
		}
		const startDate = new Date(filterStartDate);
		const endDate = new Date(filterEndDate);
		if (startDate > endDate) {
			errorMessage = 'Start date cannot be later than end date.';
			return false;
		}
		errorMessage = '';
		return true;
	}

	/* ─────────── Download Report ─────────── */
	async function downloadReport() {
		const params = new URLSearchParams();
		if (filterStartDate && filterEndDate) {
			params.append('from', filterStartDate);
			params.append('to', filterEndDate);
		}

		try {
			const res = await fetch(`https://slbcph.site/api/download-report?${params.toString()}`, {
				headers: { Authorization: `Bearer ${localStorage.getItem('authToken')}` }
			});

			if (!res.ok) {
				const data = await res.json().catch(() => ({}));
				throw new Error(data.error || `HTTP ${res.status}`);
			}

			const blob = await res.blob();
			const url = URL.createObjectURL(blob);
			const a = document.createElement('a');
			a.href = url;
			a.download = 'donations_report.pdf';
			document.body.appendChild(a);
			a.click();
			a.remove();

			Swal.fire('Success', 'PDF downloaded.', 'success');
		} catch (err: any) {
			console.error(err);
			Swal.fire('Error', err.message || 'Download failed.', 'error');
		}
	}

	async function confirmDownload() {
		if (!validateDates()) {
			Swal.fire('Invalid dates', errorMessage, 'warning');
			return;
		}
		const { isConfirmed } = await Swal.fire({
			title: 'Download report?',
			text: 'Generate and download the PDF for this date range.',
			icon: 'question',
			showCancelButton: true,
			confirmButtonText: 'Yes, download',
			cancelButtonText: 'Cancel'
		});
		if (isConfirmed) await downloadReport();
	}

	/* ─────────── Load Chart.js then fetch ─────────── */
	onMount(() => {
		const script = document.createElement('script');
		script.src = 'https://cdn.jsdelivr.net/npm/chart.js';

		script.onload = () => {
			fetchDonations();
			checkNewReport(); // ✅ Call this to detect NEW report
		};

		script.onerror = () => console.error('Failed to load Chart.js');
		document.head.appendChild(script);
	});

	/* ─────────── Filters ─────────── */
	$: {
		filteredDonations = donations.filter((donation) => {
			const donationDate = new Date(donation.date);
			const matchesMonth = filterMonth ? donationDate.getMonth() + 1 === +filterMonth : true;
			const matchesYear = filterYear ? donationDate.getFullYear() === +filterYear : true;
			const matchesSearch =
				donation.category.toLowerCase().includes(searchQuery.toLowerCase()) ||
				donation.amount.toString().includes(searchQuery);
			return matchesMonth && matchesYear && matchesSearch;
		});
	}
	// async function goToTransparency() {
	//   try {
	//     const branchId = JSON.parse(localStorage.getItem("user") ?? "{}")?.branch_id;
	//     if (!branchId) throw new Error("Branch ID not found");

	//     const res = await fetch(`https://slbcph.site/api/transparency?id=${branchId}`);
	//     const data = await res.json();

	//     if (!data.success || !data.pdf_link) {
	//       Swal.fire({
	//         icon: 'error',
	//         title: 'Transparency PDF Not Found',
	//         text: data.message || 'No PDF available for this branch.'
	//       });
	//       return;
	//     }
	//     if (data.pdf_link.startsWith('http')) {
	//       window.open(data.pdf_link, '_blank');
	//       return;
	//     }
	//     window.open(`https://slbcph.site${data.pdf_link}`, '_blank');
	//   } catch (error: any) {
	//     console.error(error);
	//     Swal.fire({
	//       icon: 'error',
	//       title: 'Error',
	//       text: error.message || 'Something went wrong'
	//     });
	//   }
	// }

	let newData: any = null;
	function goToTransparency() {
		if (!newData) {
			Swal.fire({ icon: 'error', title: 'Error', text: 'Report data not loaded yet' });
			return;
		}
		localStorage.setItem('lastViewedPdfLink', newData.pdf_link);
		hasNewReport = false;
		const branchId = JSON.parse(localStorage.getItem('user') ?? '{}')?.branch_id;
		if (!branchId) return Swal.fire({ icon: 'error', title: 'Error', text: 'Branch ID not found' });

		window.location.href = `https://slbcph.site/api/transparency?id=${branchId}`;
	}

	async function checkNewReport() {
		try {
			const branchId = JSON.parse(localStorage.getItem('user') ?? '{}')?.branch_id;
			if (!branchId) return;
			const res = await fetch(`https://slbcph.site/api/transparency-info?id=${branchId}`, {
				headers: {
					Authorization: `Bearer ${localStorage.getItem('authToken')}`
				}
			});
			const data = await res.json();
			newData = data;
			// API now returns { has_new: true/false, pdf_link: string }
			const lastViewedPdfLink = localStorage.getItem('lastViewedPdfLink');
			if (!lastViewedPdfLink || lastViewedPdfLink !== data.pdf_link) {
				hasNewReport = true;
				// Trigger notification popup
				Swal.fire({
					title: 'New Monthly Report Available!',
					text: 'Your branch admin has uploaded a new financial report. Check it out in the Monthly Church Financial Report section.',
					icon: 'info',
					confirmButtonText: 'OK',
					timer: 5000
				});
			}
			reportPdfLink = data.pdf_link; // Store for viewing
		} catch (e) {
			console.error('Failed to fetch report info:', e);
			// Optional: Add user notification on failure
		}
	}
</script>

<svelte:head>
	<title>Financial Tracking - SLBCPH Members</title>
</svelte:head>

<Layout>
	<div slot="header">Financial Tracking</div>

	<div class="rounded-lg bg-white p-6 shadow-md">
		<!-- Report Download Filters -->
		<!-- svelte-ignore a11y_label_has_associated_control -->
		<label class="text-lg font-semibold md:text-lg lg:text-xl"> Download Report Date Range: </label>
		<!-- === REPORT ACTIONS (SIDE-BY-SIDE) === -->
		<div class="mt-4 grid grid-cols-1 gap-6 md:grid-cols-2">
			<!-- === PERSONAL DONATION REPORT === -->
			<div class="rounded-lg border bg-blue-50 p-4 shadow-sm">
				<h2 class="mb-2 text-lg font-semibold text-blue-700">Personal Giving Report</h2>

				<!-- svelte-ignore a11y_label_has_associated_control -->
				<label class="mb-1 block text-base font-medium"> Select Date Range: </label>

				<div class="flex flex-col gap-3">
					<input
						type="date"
						bind:value={filterStartDate}
						class="w-full rounded border p-2"
						max={today}
					/>

					<input
						type="date"
						bind:value={filterEndDate}
						class="w-full rounded border p-2"
						max={today}
					/>

					<button
						on:click={confirmDownload}
						class="w-full rounded bg-blue-600 px-4 py-2 font-medium text-white hover:bg-blue-700"
					>
						Download Personal Report
					</button>
				</div>
			</div>

			<!-- === MONTHLY CHURCH REPORT (FROM ADMIN) === -->
			<div class="rounded-lg border bg-green-50 p-10 shadow-sm">
				<h2 class="mb-[3%] flex items-center gap-2 text-xl font-semibold text-green-700">
					Monthly Church Financial Report

					{#if hasNewReport}
						<span class="h-3 w-3 animate-pulse rounded-full bg-red-600"></span>
					{/if}
				</h2>

				<p class="text-md mb-8 text-gray-600">
					This document serves as the monthly financial report, thoroughly prepared and reviewed by
					your branch administrator to provide an accurate overview of the branch’s financial
					activities.
				</p>

				<button
					on:click={goToTransparency}
					class="w-full rounded bg-green-600 px-4 py-2 font-medium text-white hover:bg-green-700"
				>
					View Monthly Church Report
				</button>
			</div>
		</div>

		<!-- Donations Table -->
		<div class="mt-4">
			<strong class="mb-2 block text-lg font-semibold md:text-lg lg:text-xl">
				Financial Tracking Overview:
			</strong>

			<div class="rounded border bg-gray-100 p-3">
				<div class="mb-4 flex items-center justify-between">
					<h3 class="text-lg font-semibold text-blue-600 md:text-lg lg:text-2xl">
						Donations Table
					</h3>
					<select bind:value={selectedBranch} class="rounded border p-2 text-sm">
						<option value="current">Current ({currentBranch})</option>

						{#each Object.keys(groupByBranch(filteredDonations)).filter((b) => b !== currentBranch) as pastBranch}
							<option value={pastBranch}>Past: {pastBranch}</option>
						{/each}
					</select>
				</div>

				<!-- Filters -->
				<div class="mb-4 flex flex-wrap gap-4">
					<div>
						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="text-lg font-semibold">Filter by Month:</label>
						<select bind:value={filterMonth} class="w-full rounded border p-2">
							<option value="">All Months</option>
							{#each Array(12) as _, i}
								<option value={i + 1}
									>{new Date(2000, i).toLocaleString('default', { month: 'long' })}</option
								>
							{/each}
						</select>
					</div>
					<div>
						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="text-lg font-semibold">Filter by Year:</label>
						<input
							type="number"
							min="1900"
							max={new Date().getFullYear()}
							bind:value={filterYear}
							placeholder="Year"
							class="w-full rounded border p-2"
						/>
					</div>
				</div>

				<div class="scrollable">
					{#if selectedBranch === 'all'}
						{#each Object.entries(groupByBranch(filteredDonations)) as [branch, donations]}
							<h3 class="mt-6 text-lg font-bold text-blue-600">{branch}</h3>
							<table class="mb-4 w-full border-collapse border">
								<thead>
									<tr class="bg-gray-200">
										<th class="border p-2">Date</th>
										<th class="border p-2">Category</th>
										<th class="border p-2">Amount Donated</th>
									</tr>
								</thead>
								<tbody>
									{#each donations as donation}
										<tr>
											<td class="border p-2">{new Date(donation.date).toLocaleDateString()}</td>
											<td class="border p-2">{donation.category}</td>
											<td class="border p-2">{Number(donation.amount).toFixed(2)} PHP</td>
										</tr>
									{/each}
								</tbody>
							</table>
						{/each}
					{:else}
						<h3 class="mt-6 text-lg font-bold text-blue-600">
							<!-- {selectedBranch === "current" ? currentBranch : selectedBranch} -->
						</h3>
						<table class="mb-4 w-full border-collapse border">
							<thead>
								<tr class="bg-gray-200">
									<th class="border p-2">Date</th>
									<th class="border p-2">Category</th>
									<th class="border p-2">Amount Donated</th>
								</tr>
							</thead>
							<tbody>
								{#if paginatedDonations.length > 0}
									{#each paginatedDonations as donation}
										<tr>
											<td class="border p-2">{new Date(donation.date).toLocaleDateString()}</td>
											<td class="border p-2">{donation.category}</td>
											<td class="border p-2">{Number(donation.amount).toFixed(2)} PHP</td>
										</tr>
									{/each}
								{:else}
									<tr>
										<td colspan="3" class="border p-2 text-center text-gray-500">
											No donations found
										</td>
									</tr>
								{/if}
							</tbody>
						</table>

						<!-- Pagination -->
						{#if totalPages > 1}
							<div class="mt-4 flex items-center justify-center gap-2">
								<button
									class="cursor-pointer rounded bg-gray-300 px-3 py-1 disabled:opacity-50"
									on:click={prevPage}
									disabled={currentPage === 1}>Prev</button
								>
								{#if totalPages > 3}
									{#if currentPage === 1}
										{#each [1, 2, 3] as page}
											<button
												class="rounded px-3 py-1 hover:bg-blue-500 hover:text-white {currentPage ===
												page
													? 'bg-blue-600 text-white'
													: 'bg-gray-200'}"
												on:click={() => goToPage(page)}>{page}</button
											>
										{/each}
									{:else if currentPage === totalPages}
										{#each [totalPages - 2, totalPages - 1, totalPages] as page}
											<button
												class="rounded px-3 py-1 hover:bg-blue-500 hover:text-white {currentPage ===
												page
													? 'bg-blue-600 text-white'
													: 'bg-gray-200'}"
												on:click={() => goToPage(page)}>{page}</button
											>
										{/each}
									{:else}
										{#each [currentPage - 1, currentPage, currentPage + 1] as page}
											<button
												class="rounded px-3 py-1 hover:bg-blue-500 hover:text-white {currentPage ===
												page
													? 'bg-blue-600 text-white'
													: 'bg-gray-200'}"
												on:click={() => goToPage(page)}>{page}</button
											>
										{/each}
									{/if}
								{:else}
									{#each Array(totalPages) as _, index}
										<button
											class="rounded px-3 py-1 hover:bg-blue-500 hover:text-white {currentPage ===
											index + 1
												? 'bg-blue-600 text-white'
												: 'bg-gray-200'}"
											on:click={() => goToPage(index + 1)}>{index + 1}</button
										>
									{/each}
								{/if}
								<button
									class="cursor-pointer rounded bg-gray-300 px-3 py-1 disabled:opacity-50"
									on:click={nextPage}
									disabled={currentPage === totalPages}>Next</button
								>
							</div>
						{/if}
					{/if}
				</div>
			</div>
		</div>

		<!-- Charts Section -->
		<div class="mt-6">
			<h3 class="mb-4 text-lg font-semibold md:text-lg lg:text-2xl">Charts Overview</h3>
			<div class="flex flex-wrap gap-6">
				<!-- Pie Chart -->
				<div class="min-w-[300px] flex-1">
					<h4 class="mb-2 text-lg font-semibold text-gray-700 md:text-lg lg:text-xl">
						Fund Allocation Chart
					</h4>
					<div class="flex items-center justify-center">
						<canvas id="pieChart" style="max-height: 300px; max-width: 100%;"></canvas>
					</div>
				</div>

				<!-- Bar Chart -->
				<div class="min-w-[300px] flex-1">
					<h4 class="mb-2 text-lg font-semibold text-gray-700 md:text-lg lg:text-xl">
						Monthly Donations Chart
					</h4>
					<div class="flex items-center justify-center">
						<canvas id="barChart" style="max-height: 300px; max-width: 100%;"></canvas>
					</div>
				</div>
			</div>
		</div>
	</div>
</Layout>

<style>
	.scrollable {
		max-width: 100%;
		overflow-x: auto;
		white-space: nowrap;
	}

	@media (max-width: 768px) {
		.flex-wrap {
			flex-direction: column;
		}

		.scrollable {
			margin-bottom: 1rem;
		}
	}
</style>
