<script lang="ts">
    import Layout from '../../layout/layout.svelte';
    import { onMount, onDestroy } from 'svelte';
    import { Html5QrcodeScanner, Html5Qrcode } from 'html5-qrcode';
	import Swal from 'sweetalert2';

    let events: any[] = [];
    let searchQuery: string = '';
    let loading = false;
    let error = '';
    let totalEventsCount = 0;

    const API_URL = 'https://slbcph.site/api';

    $: filteredEvents = events.filter(e => {
        const lcTitle = e.title?.toLowerCase() ?? '';
        const lcLocation = e.location?.toLowerCase() ?? '';
        const matchesSearch = lcTitle.includes(searchQuery.toLowerCase()) || lcLocation.includes(searchQuery.toLowerCase());
        return matchesSearch;     
    });

    async function fetchEvents() {
        const token = localStorage.getItem('authToken');
        if (!token) {
            error = 'You must be logged in to view events.';
            return;
        }

        loading = true;
        try {
            const res = await fetch(`${API_URL}/sunday-service-attendance`, {
                headers: { Authorization: `Bearer ${token}` }
            });
            if (!res.ok) throw new Error((await res.json()).message ?? 'Failed to load events.');
            const data = await res.json();
            events = Array.isArray(data.events) ? data.events : [];
            totalEventsCount = data.total_events ?? events.length;
        } catch (err: any) {
            error = err.message;
        } finally {
            loading = false;
        }
    }

    let locked = false;

    let qrScanner: Html5QrcodeScanner | undefined;
    function startQrScanner() {
        qrScanner = new Html5QrcodeScanner('qr-reader', { fps: 10, qrbox: 250 }, undefined);
        qrScanner.render(
            async text => {
                if (locked) return;
                locked = true;
                await handleQrCodeScanned(text);
                locked = false;
                stopQrScanner();
            },
            errorMessage => {
            }
        );
    }
    function stopQrScanner() {
        qrScanner?.clear();
    }

    let scanned = false;
async function handleQrCodeScanned(qrText: string) {
    if (scanned ) return; 
    scanned = true;
    const token = localStorage.getItem('authToken');
    if (!token) {
        error = 'You must be logged in to record attendance.';
        return;
    }

    try {
        const res = await fetch(`${API_URL}/sunday-service-attendance`, {
            method: 'POST',
            headers: {
                Authorization: `Bearer ${token}`,
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ event_id: qrText, status: 'Attended' })
        });
        if (!res.ok) throw new Error((await res.json()).message ?? 'Failed to record attendance.');
        
        await fetchEvents();

        Swal.fire({
            icon: 'success',
            title: 'Attendance Recorded',
            text: 'You have successfully attended this event!',
            confirmButtonColor: '#16a34a'
        });
    } catch (err: any) {
        error = err.message;
        Swal.fire({
            icon: 'error',
            title: 'Error',
            text: error || 'Something went wrong while recording attendance.'
        });
    }
}


    // Mount
  onMount(() => {
    fetchEvents();
    fetchFinishedEvents();
});
    onDestroy(stopQrScanner);

    // New function to handle image file upload and scan
    async function scanQrFromFile(event: Event) {
        error = '';
        const input = event.target as HTMLInputElement;
        if (!input.files || input.files.length === 0) {
            error = 'No file selected.';
            return;
        }
        const file = input.files[0];
        const html5QrCode = new Html5Qrcode("qr-reader");
        try {
            // Use scanFile with 3 arguments: file, showImage, scanRegion
            // @ts-ignore - TypeScript definitions may be incorrect
            const decodedText = await html5QrCode.scanFile(file, true, null);
            await handleQrCodeScanned(decodedText);
        } catch (err: any) {
            error = err?.message || 'Failed to decode QR code from image.';
        } finally {
            html5QrCode.clear();
            input.value = ''; // reset input
        }
    }


    let finishedEvents: any[] = [];
let loadingFinished = false;
let finishedError = '';

async function fetchFinishedEvents() {
    const token = localStorage.getItem('authToken');
    if (!token) {
        finishedError = 'You must be logged in to view finished events.';
        return;
    }

    loadingFinished = true;
    try {
        const res = await fetch(`${API_URL}/sunday-service-attendance/finished-events`, {
            headers: { Authorization: `Bearer ${token}` }
        });
        if (!res.ok) throw new Error((await res.json()).message ?? 'Failed to load finished events.');
        const data = await res.json();
        finishedEvents = Array.isArray(data.events) ? data.events : [];
    } catch (err: any) {
        finishedError = err.message;
    } finally {
        loadingFinished = false;
    }
}

let selectedEvent: any = null;
let selectedField: string = ''; // 'title' or 'location'
let showModal: boolean = false;

function openEventModal(event: any, field: string) {
    selectedEvent = event;
    selectedField = field;
    showModal = true;
}

let currentPage = 1;
const perPage = 5;

$: totalPages = Math.ceil(finishedEvents.length / perPage);
$: if (finishedEvents) currentPage = 1;
$: paginatedFinishedEvents = finishedEvents.slice(
  (currentPage - 1) * perPage,
  currentPage * perPage
);

function nextPage() {
  if (currentPage < totalPages) currentPage++;
}

function prevPage() {
  if (currentPage > 1) currentPage--;
}


</script>

<Layout>
    <div slot="header">Sunday Service Monitoring</div>
    <div class="bg-white p-6 rounded-lg shadow-md">

        <!-- Table -->
        <div class="scrollable"> <h2 class="font-semibold text-xl md:text-xl lg:text-2xl">Ongoing Service/Event</h2>
            {#if loading}
                <p class="text-center text-base md:text-base lg:text-lg">Loading services/events…</p>
            {:else if error}
                <p class="text-red-500">{error}</p>
            {:else if filteredEvents.length === 0}
                <p class="text-gray-600 text-center text-base md:text-base lg:text-lg">No ongoing service/event found for today.</p>
            {:else}
                <table class="w-full border-collapse border text-base md:text-base lg:text-lg">
                    <thead>
                        <tr class="bg-gray-200">
                            <th class="border p-2 text-center">Date</th>
                            <th class="border p-2 text-center">Title</th>
                            <th class="border p-2 text-center">Location</th>
                            <th class="border p-2 text-center">Status</th>
                            <th class="border p-2 text-center">Action</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each filteredEvents as ev}
                            <tr>
                                <td class="border p-2 text-center">
                                    {new Date(ev.event_date).toLocaleDateString('en-US')}
                               <td
  class="border p-2 text-center max-w-[150px] truncate cursor-pointer"
  title={ev.title}
  on:click={() => openEventModal(ev, 'title')}
>
  {ev.title}
</td>

<td
  class="border p-2 text-center max-w-[150px] truncate cursor-pointer"
  title={ev.location}
  on:click={() => openEventModal(ev, 'location')}
>
  {ev.location}
</td>
                                <td class="border p-2 text-center font-semibold
                                    {ev.attendance_status === 'Attended' ? 'text-green-600' :
                                     ev.attendance_status === 'Missed' ? 'text-red-600' :
                                     'text-gray-500'}">
                                    {ev.attendance_status ?? 'Not Recorded'}
                                </td>
                                <td class="border p-2 text-center">
                                    {#if ev.attendance_status !== 'Attended'}
                                        <button
                                            on:click={startQrScanner}
                                            class="px-3 py-1 rounded text-white bg-green-600 hover:bg-green-700 transition">
                                            Scan QR
                                        </button>
                                    {/if}
                                </td>
                            </tr>
                        {/each}
                    </tbody>
                </table>
            {/if}

            
        </div>


    <div id="qr-reader" class="w-full h-auto mt-6"></div>

    

<!-- Attendance Summary -->
<div class="mt-4 p-4 border rounded bg-gray-100 text-base md:text-base lg:text-lg">
  <strong>Attendance Summary:</strong>

  {#if loading || loadingFinished}
    <p>Calculating attendance…</p>
  {:else}
    <p>
      You attended
      <span class="font-semibold">
        {
          [...events, ...finishedEvents].filter(
            e => e.attendance_status === 'Attended'
          ).length
        }
      </span>
      of
      <span class="font-semibold">
        {[...events, ...finishedEvents].length}
      </span>
      services/events
    </p>
  {/if}
</div>



        <!-- FINISHED EVENTS TABLE -->
<h2 class="mt-3 text-xl md:text-xl lg:text-2xl font-semibold ">Past Services/Events</h2>



<div class="scrollable mt-4 text-base md:text-base lg:text-lg">
    {#if loadingFinished}
        <p class="text-center">Loading finished services/events…</p>
    {:else if finishedError}
        <p class="text-red-500">{finishedError}</p>
    {:else if finishedEvents.length === 0}
        <p class="text-gray-600 text-center">No finished services/events found.</p>
    {:else}
        <table class="w-full border-collapse border text-base md:text-base lg:text-lg table-auto">
            <thead>
                <tr class="bg-gray-100 text-gray-700 text-lg md:text-lg lg:text-xl">
                    <th class="border p-2 text-center">Date</th>
                    <th class="border p-2 text-left">Title</th>
                    <th class="border p-2 text-left">Location</th>
                    <th class="border p-2 text-center">Status</th>
                </tr>
            </thead>
            <tbody class="text-gray-800">
               {#each paginatedFinishedEvents as ev}
                    <tr class="hover:bg-gray-50 transition">
                        <td class="border p-2 text-center">
                            {new Date(ev.event_date).toLocaleDateString('en-US')}
                        </td>

                        <!-- 📌 Title: truncate and clickable -->
                        <td class="border p-2 max-w-[180px] truncate  cursor-pointer hover:underline"
                            title={ev.title}
                            on:click={() => openEventModal(ev, 'title')}
>
                            {ev.title}
                        </td>

                        <!-- 📌 Location: truncate and clickable -->
                        <td class="border p-2 max-w-[180px] truncate cursor-pointer hover:underline"
                            title={ev.location}
                            on:click={() => openEventModal(ev, 'location')}>
                            {ev.location}
                        </td>

                        <!-- ✅ Attendance Status -->
                        <td class="border p-2 text-center font-semibold
                            {ev.attendance_status === 'Attended' ? 'text-green-600' :
                             ev.attendance_status === 'Missed' ? 'text-red-600' :
                             'text-gray-500'}">
                            {ev.attendance_status ?? 'Not Recorded'}
                        </td>
                    </tr>
                {/each}
            </tbody>
        </table>
        
    {/if}
</div>
<div class="flex justify-between items-center mt-4">
  <button
    class="px-3 py-1 border rounded disabled:opacity-50"
    on:click={prevPage}
    disabled={currentPage === 1}
  >
    Prev
  </button>

  <span class="text-sm text-gray-700">
    Page {currentPage} of {totalPages}
  </span>

<button
  class="px-3 py-1 border rounded disabled:opacity-50 cursor-pointer"
  on:click={nextPage}
  disabled={currentPage === totalPages}
>
  Next
</button>

  
</div>

    </div>

  
</Layout>
   {#if showModal}
 <div class="fixed inset-0 bg-white/30  z-[9999] flex items-center justify-center px-4">

    <div
      class="bg-white p-6 rounded-lg shadow-lg w-full max-w-md max-h-[80vh] overflow-y-auto"
    >
      <h2 class="text-xl font-bold mb-2">
        {selectedField === 'title' ? 'Event Title' : 'Event Location'}
      </h2>
      <p class="break-words text-gray-700">
        {selectedField === 'title' ? selectedEvent.title : selectedEvent.location}
      </p>

      <div class="mt-6 text-right">
        <button
          class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600"
          on:click={() => showModal = false}
        >
          Close
        </button>
      </div>
    </div>
  </div>
{/if}

<style>
    .scrollable {
        max-width: 100%;
        overflow-x: auto;
        white-space: nowrap;
    }
</style>
