<svelte:head>
  <title>
    Financial Tracking - SLBCPH Members
  </title>
</svelte:head>

<script lang="ts">
	import type { Donation } from "$lib/type";
	import { Chart, registerables  } from "chart.js";
  import Layout from "../../layout/layout.svelte";
  import { onMount } from "svelte";
  import Swal from "sweetalert2";
  Chart.register(...registerables);

  let donations: Donation[] = [];
  let filteredDonations: Donation[] = [];
  let filterStartDate = "";
  let filterEndDate = "";
  let categoryFilter = "all";
  let searchQuery = "";
  let filterMonth: number | null = null;
  let filterYear: number | null = null;
  let pieChartInstance: any = null;
  let barChartInstance: any = null;
  let errorMessage = "";
  let today = new Date().toISOString().split("T")[0];
  let currentBranch = "";
  let selectedBranch = "current";

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
      params.append("from", filterStartDate);
      params.append("to", filterEndDate);
    }

    if (categoryFilter && categoryFilter !== "all") {
      params.append("event", categoryFilter);
    }

    try {
      const res = await fetch(
        `http://localhost:8000/api/financial-tracking-data?${params.toString()}`,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("authToken")}`,
          },
        }
      );

      if (!res.ok) {
        const errorText = await res.text();
        throw new Error(
          `HTTP error! status: ${res.status}, message: ${errorText}`
        );
      }

      const data = await res.json();
      donations = data.donations || [];
      filteredDonations = donations;
      currentBranch = data.currentBranch || "";
    } catch (error) {
      console.error("Failed to fetch donations", error);
    }
  }

  /* ─────────── Group by Branch ─────────── */
  function groupByBranch(donations: Donation[]): Record<string, Donation[]> {
    return donations.reduce((acc: Record<string, Donation[]>, d) => {
      const branch = d.branch_name || "N/A";
      if (!acc[branch]) acc[branch] = [];
      acc[branch].push(d);
      return acc;
    }, {});
  }

  /* ─────────── Displayed Donations ─────────── */
  $: displayedDonations = (() => {
    const grouped = groupByBranch(filteredDonations);

    if (selectedBranch === "all") {
      return filteredDonations;
    } else if (selectedBranch === "current") {
      return grouped[currentBranch] || [];
    } else {
      return grouped[selectedBranch] || [];
    }
  })();

  /* ─────────── Charts ─────────── */
  function createCharts() {
    const pieCanvas = document.getElementById("pieChart") as HTMLCanvasElement;
    const barCanvas = document.getElementById("barChart") as HTMLCanvasElement;

    if (!pieCanvas || !barCanvas) return;

    const pieCtx = pieCanvas.getContext("2d");
    const barCtx = barCanvas.getContext("2d");

    // Category totals
    const categoryTotals = displayedDonations.reduce((acc, donation) => {
      acc[donation.category] =
        (acc[donation.category] || 0) + Number(donation.amount);
      return acc;
    }, {} as Record<string, number>);

    const categories = Object.keys(categoryTotals);
    const pieData = Object.values(categoryTotals);

    // Pie chart
    if (pieCtx) {
      if (pieChartInstance) pieChartInstance.destroy();
      pieChartInstance = new Chart(pieCtx, {
        type: "pie",
        data: {
          labels: categories,
          datasets: [
            {
              label: "Fund Allocation",
              data: pieData,
              backgroundColor: [
                "rgba(255, 99, 132, 0.2)",
                "rgba(54, 162, 235, 0.2)",
                "rgba(255, 206, 86, 0.2)",
                "rgba(75, 192, 192, 0.2)",
                "rgba(153, 102, 255, 0.2)",
              ],
              borderColor: [
                "rgba(255, 99, 132, 1)",
                "rgba(54, 162, 235, 1)",
                "rgba(255, 206, 86, 1)",
                "rgba(75, 192, 192, 1)",
                "rgba(153, 102, 255, 1)",
              ],
              borderWidth: 1,
            },
          ],
        },
        options: {
          responsive: true,
          plugins: {
            legend: { position: "top" },
            title: { display: true, text: "Fund Allocation Overview" },
          },
        },
      });
    }

    // Bar chart
    const monthlyData: Record<string, number> = {};
    displayedDonations.forEach((donation) => {
      const month = new Date(donation.date).toLocaleString("default", {
        month: "long",
        year: "numeric",
      });
      monthlyData[month] =
        (monthlyData[month] || 0) + Number(donation.amount);
    });

    const labels = Object.keys(monthlyData);
    const amounts = Object.values(monthlyData);

    if (barCtx) {
      if (barChartInstance) barChartInstance.destroy();
      barChartInstance = new Chart(barCtx, {
        type: "bar",
        data: {
          labels,
          datasets: [
            {
              label: "Total Amount Donated per Month",
              data: amounts,
              backgroundColor: "rgba(75, 192, 192, 0.2)",
              borderColor: "rgba(75, 192, 192, 1)",
              borderWidth: 1,
            },
          ],
        },
        options: {
          responsive: true,
          scales: {
            y: {
              beginAtZero: true,
              ticks: {
                stepSize: Math.ceil(Math.max(...amounts) / 5) || 1,
              },
            },
          },
          plugins: {
            legend: { position: "top" },
            title: { display: true, text: "Monthly Donations Overview" },
          },
        },
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
      errorMessage = "Please select both start and end dates.";
      return false;
    }
    const startDate = new Date(filterStartDate);
    const endDate = new Date(filterEndDate);
    if (startDate > endDate) {
      errorMessage = "Start date cannot be later than end date.";
      return false;
    }
    errorMessage = "";
    return true;
  }

  /* ─────────── Download Report ─────────── */
  async function downloadReport() {
    const params = new URLSearchParams();
    if (filterStartDate && filterEndDate) {
      params.append("from", filterStartDate);
      params.append("to", filterEndDate);
    }

    try {
      const res = await fetch(
        `http://localhost:8000/api/download-report?${params.toString()}`,
        { headers: { Authorization: `Bearer ${localStorage.getItem("authToken")}` } }
      );

      if (!res.ok) {
        const data = await res.json().catch(() => ({}));
        throw new Error(data.error || `HTTP ${res.status}`);
      }

      const blob = await res.blob();
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "donations_report.pdf";
      document.body.appendChild(a);
      a.click();
      a.remove();

      Swal.fire("Success", "PDF downloaded.", "success");
    } catch (err: any) {
      console.error(err);
      Swal.fire("Error", err.message || "Download failed.", "error");
    }
  }

  async function confirmDownload() {
    if (!validateDates()) {
      Swal.fire("Invalid dates", errorMessage, "warning");
      return;
    }
    const { isConfirmed } = await Swal.fire({
      title: "Download report?",
      text: "Generate and download the PDF for this date range.",
      icon: "question",
      showCancelButton: true,
      confirmButtonText: "Yes, download",
      cancelButtonText: "Cancel",
    });
    if (isConfirmed) await downloadReport();
  }

  /* ─────────── Load Chart.js then fetch ─────────── */
  onMount(() => {
    const script = document.createElement("script");
    script.src = "https://cdn.jsdelivr.net/npm/chart.js";
    script.onload = () => fetchDonations();
    script.onerror = () => console.error("Failed to load Chart.js");
    document.head.appendChild(script);
  });

  /* ─────────── Filters ─────────── */
  $: {
    filteredDonations = donations.filter((donation) => {
      const donationDate = new Date(donation.date);
      const matchesMonth = filterMonth
        ? donationDate.getMonth() + 1 === +filterMonth
        : true;
      const matchesYear = filterYear
        ? donationDate.getFullYear() === +filterYear
        : true;
      const matchesSearch =
        donation.category.toLowerCase().includes(searchQuery.toLowerCase()) ||
        donation.amount.toString().includes(searchQuery);
      return matchesMonth && matchesYear && matchesSearch;
    });
  }
</script>

<Layout>
  <div slot="header">Financial Tracking</div>

  <div class="bg-white p-6 rounded-lg shadow-md">
    <!-- Report Download Filters -->
    <!-- svelte-ignore a11y_label_has_associated_control -->
    <label class="font-semibold text-lg md:text-lg lg:text-xl">
      Download Report Date Range:
    </label>
    <div class="flex flex-col md:flex-row gap-4 mb-4">
      <input type="date" bind:value={filterStartDate} class="border p-2 rounded w-full md:w-auto" max={today} />
      <input type="date" bind:value={filterEndDate} class="border p-2 rounded w-full md:w-auto" max={today} />
      <button
        on:click={confirmDownload}
        class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded w-full md:w-auto text-base md:text-base lg:text-lg transition-transform duration-300 transform hover:scale-[1.02] cursor-pointer"
      >
        Download Report
      </button>
    </div>

    <!-- Donations Table -->
    <div class="mt-4">
      <strong class="block text-lg md:text-lg lg:text-xl mb-2 font-semibold">
        Financial Tracking Overview:
      </strong>

      <div class="p-3 border rounded bg-gray-100">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg md:text-lg lg:text-2xl font-semibold text-blue-600">
            Donations Table
          </h3>
          <select bind:value={selectedBranch} class="border p-2 rounded text-sm">
            <option value="current">Current ({currentBranch})</option>
            
            {#each Object.keys(groupByBranch(filteredDonations)).filter(b => b !== currentBranch) as pastBranch}
              <option value={pastBranch}>Past: {pastBranch}</option>
            {/each}
          </select>
        </div>

        <!-- Filters -->
        <div class="flex flex-wrap gap-4 mb-4">
          <div>
            <!-- svelte-ignore a11y_label_has_associated_control -->
            <label class="font-semibold text-lg">Filter by Month:</label>
            <select bind:value={filterMonth} class="border p-2 rounded w-full">
              <option value="">All Months</option>
              {#each Array(12) as _, i}
                <option value={i+1}>{new Date(2000, i).toLocaleString("default", { month: "long" })}</option>
              {/each}
            </select>
          </div>
          <div>
            <!-- svelte-ignore a11y_label_has_associated_control -->
            <label class="font-semibold text-lg">Filter by Year:</label>
            <input type="number" min="1900" max={new Date().getFullYear()} bind:value={filterYear} placeholder="Year" class="border p-2 rounded w-full" />
          </div>
        </div>

        <div class="scrollable">
          {#if selectedBranch === "all"}
            {#each Object.entries(groupByBranch(filteredDonations)) as [branch, donations]}
              <h3 class="mt-6 font-bold text-lg text-blue-600">{branch}</h3>
              <table class="w-full border-collapse border mb-4">
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
            <h3 class="mt-6 font-bold text-lg text-blue-600">
              <!-- {selectedBranch === "current" ? currentBranch : selectedBranch} -->
            </h3>
            <table class="w-full border-collapse border mb-4">
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
              <div class="flex justify-center items-center mt-4 gap-2">
                <button class="px-3 py-1 rounded bg-gray-300 cursor-pointer disabled:opacity-50" on:click={prevPage} disabled={currentPage === 1}>Prev</button>
                {#if totalPages > 3}
                  {#if currentPage === 1}
                    {#each [1,2,3] as page}
                      <button class="px-3 py-1 rounded hover:bg-blue-500 hover:text-white {currentPage===page ? 'bg-blue-600 text-white' : 'bg-gray-200'}" on:click={() => goToPage(page)}>{page}</button>
                    {/each}
                  {:else if currentPage === totalPages}
                    {#each [totalPages-2, totalPages-1, totalPages] as page}
                      <button class="px-3 py-1 rounded hover:bg-blue-500 hover:text-white {currentPage===page ? 'bg-blue-600 text-white' : 'bg-gray-200'}" on:click={() => goToPage(page)}>{page}</button>
                    {/each}
                  {:else}
                    {#each [currentPage-1,currentPage,currentPage+1] as page}
                      <button class="px-3 py-1 rounded hover:bg-blue-500 hover:text-white {currentPage===page ? 'bg-blue-600 text-white' : 'bg-gray-200'}" on:click={() => goToPage(page)}>{page}</button>
                    {/each}
                  {/if}
                {:else}
                  {#each Array(totalPages) as _, index}
                    <button class="px-3 py-1 rounded hover:bg-blue-500 hover:text-white {currentPage===index+1 ? 'bg-blue-600 text-white' : 'bg-gray-200'}" on:click={() => goToPage(index+1)}>{index+1}</button>
                  {/each}
                {/if}
                <button class="px-3 py-1 rounded bg-gray-300 cursor-pointer disabled:opacity-50" on:click={nextPage} disabled={currentPage === totalPages}>Next</button>
              </div>
            {/if}
          {/if}
        </div>
      </div>
    </div>

  

        <!-- Charts Section -->
        <div class="mt-6">
            <h3 class="text-lg md:text-lg lg:text-2xl font-semibold mb-4 ">Charts Overview</h3>
            <div class="flex flex-wrap gap-6">
                <!-- Pie Chart -->
                <div class="flex-1 min-w-[300px]">
                    <h4 class="text-lg md:text-lg lg:text-xl font-semibold mb-2 text-gray-700">Fund Allocation Chart</h4>
                    <div class="flex items-center justify-center">
                        <canvas id="pieChart" style="max-height: 300px; max-width: 100%;"></canvas>
                    </div>
                </div>

                <!-- Bar Chart -->
                <div class="flex-1 min-w-[300px]">
                    <h4 class="text-lg md:text-lg lg:text-xl font-semibold mb-2 text-gray-700">Monthly Donations Chart</h4>
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
