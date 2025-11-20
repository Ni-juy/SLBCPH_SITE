<script lang="ts">
import Layout from '../../layout/layout.svelte';
import { onMount } from 'svelte';

interface Metrics {
    upcomingEvents: number;
    prayerRequests: number;
    attended: number;
    missed: number;
    financialSummary: string;
}

let metrics: Metrics = {
    upcomingEvents: 0,
    prayerRequests: 0,
    attended: 0,
    missed: 0,
    financialSummary: '₱ 0.00',
};

let loading = false;
let error = '';



const API_BASE = 'https://slbcph.site';
async function fetchMetrics() {
    loading = true;
    error = '';
    try {
        const token = localStorage.getItem('authToken');
        if (!token) throw new Error('User is not logged in');

        const res = await fetch(`${API_BASE}/api/dashboard-metrics`, {
            method: 'GET',
            headers: {
                'Accept': 'application/json',
                'Authorization': `Bearer ${token}`,
            },
        });

        if (!res.ok) {
            const text = await res.text();
            throw new Error(`HTTP ${res.status}: ${text}`);
        }

        const data = await res.json();
        metrics = {
            upcomingEvents: data.upcomingEvents ?? 0,
            attended: data.attended ?? 0,
            missed: data.missed ?? 0,
            prayerRequests: data.todayPrayerRequests ?? 0,
            financialSummary: data.financialSummary ?? '₱ 0.00',
        };
    } catch (err: any) {
        error = err.message ?? 'Failed to fetch metrics';
    } finally {
        loading = false;
    }
}

onMount(fetchMetrics);
</script>

<Layout>
    <span slot="header">Dashboard</span>

    {#if loading}
        <p class="text-center text-gray-500">Loading metrics...</p>
    {:else if error}
        <p class="text-center text-red-500">{error}</p>
    {:else}
        <div class="grid grid-cols-1 gap-6 md:grid-cols-2">

            <!-- Upcoming Events -->
            <div class="flex flex-col justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md hover:scale-[1.02] hover:shadow-xl">
                <div>
                    <h2 class="text-lg font-bold hover:text-blue-700">UPCOMING EVENTS</h2>
                    <p class="text-2xl font-semibold mt-2">{metrics.upcomingEvents}</p>
                </div>
                <a href="/members/events" class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white hover:bg-gray-600">
                    View
                </a>
            </div>

            <!-- Prayer Requests -->
            <div class="flex flex-col justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md hover:scale-[1.02] hover:shadow-xl">
                <div>
                    <h2 class="text-lg font-bold hover:text-blue-700">PRAYER REQUESTS / BLESSINGS / REFLECTION OF THE DAY</h2>
                    <p class="text-2xl font-semibold mt-2">{metrics.prayerRequests}</p>
                </div>
                <a href="/members/pr" class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white hover:bg-gray-600">
                    Send
                </a>
            </div>

            <!-- Attendance -->
            <div class="flex flex-col justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md hover:scale-[1.02] hover:shadow-xl">
                <div>
                    <h2 class="text-lg font-bold hover:text-blue-700">MY ATTENDANCE</h2>
                    <p class="text-2xl font-semibold mt-2">
                        Attended: {metrics.attended} | Missed: {metrics.missed}
                    </p>
                </div>
                <a href="/members/ssm" class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white hover:bg-gray-600">
                    Monitor
                </a>
            </div>

            <!-- Financial Summary -->
            <div class="flex flex-col justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md hover:scale-[1.02] hover:shadow-xl">
                <div>
                    <h2 class="text-lg font-bold hover:text-blue-700">TITHES AND OFFERINGS</h2>
                    <p class="text-2xl font-semibold mt-2">{metrics.financialSummary}</p>
                </div>
                <a href="/members/financial" class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white hover:bg-gray-600">
                    View
                </a>
            </div>

        </div>
    {/if}
</Layout>
