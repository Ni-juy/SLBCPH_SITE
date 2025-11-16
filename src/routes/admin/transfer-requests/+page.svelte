<script lang="ts">
  import { onMount } from 'svelte';

  let transferRequests: any[] = [];
  let error: string = '';

  async function fetchTransferRequests() {
    const token = localStorage.getItem('authToken');
    if (!token) {
      error = 'User not authenticated.';
      return;
    }

    try {
      const response = await fetch('https://slbcph.site/api/admin/members/transfer-requests', {
        headers: {
          'Authorization': `Bearer ${token}`,
          'Accept': 'application/json',
        },
      });

      if (!response.ok) {
        error = `Failed to load transfer requests: ${response.status} ${response.statusText}`;
        return;
      }

      transferRequests = await response.json();
    } catch (err: unknown) {
      if (err instanceof Error) {
        error = 'Error fetching transfer requests: ' + err.message;
      } else {
        error = 'Unknown error fetching transfer requests';
      }
    }
  }

  onMount(() => {
    fetchTransferRequests();
  });
</script>

<h1 class="text-2xl font-bold mb-4">Transfer Member Requests</h1>

{#if error}
  <p class="text-red-600">{error}</p>
{:else}
  <table class="min-w-full bg-white rounded shadow">
    <thead>
      <tr>
        <th class="py-2 px-4 border-b">Member Name</th>
        <th class="py-2 px-4 border-b">Reason</th>
        <th class="py-2 px-4 border-b">Status</th>
        <th class="py-2 px-4 border-b">Requested At</th>
      </tr>
    </thead>
    <tbody>
      {#each transferRequests as request}
        <tr class="hover:bg-gray-100">
          <td class="py-2 px-4 border-b">{request.user ? `${request.user.first_name} ${request.user.middle_name} ${request.user.last_name}` : 'Unknown'}</td>
          <td class="py-2 px-4 border-b">{request.reason}</td>
          <td class="py-2 px-4 border-b">{request.status}</td>
          <td class="py-2 px-4 border-b">{new Date(request.created_at).toLocaleString()}</td>
        </tr>
      {/each}
    </tbody>
  </table>
{/if}
