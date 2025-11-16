<script>
	import RequestHandler from '$lib/request';
  import { onMount } from 'svelte';
	import Swal from 'sweetalert2';

  let isLoading = false;
  let errorMessage = '';

  onMount(() => {
    const authToken = localStorage.getItem('authToken');
    const user = JSON.parse(localStorage.getItem('user') || '{}');
    
    if (!authToken || !user.id) {
      window.location.href = '/login';
      return;
    }
    if (user.terms_accepted_at) {
      window.location.href = '/members/memberdb';
    }
  });

  async function acceptTerms() {
    isLoading = true;
    errorMessage = '';

    try {
      const authToken = localStorage.getItem('authToken');
      const data = await RequestHandler.fetchData('POST', 'accept-terms', {}, {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${authToken}`
      });

      if (!data.success) {
        Swal.fire('Error', 'Failed to accept terms.');
        return;
      }

      const user = JSON.parse(localStorage.getItem('user') || '{}');
      user.terms_accepted_at = new Date().toISOString();
      user.requires_terms_acceptance = false;

      localStorage.setItem('user', JSON.stringify(user));
      window.location.href = '/members/memberdb';

    } catch (err) {
      errorMessage = err instanceof Error ? err.message : 'An error occurred';
    } finally {
      isLoading = false;
    }
  }

  function logout() {
    localStorage.removeItem('authToken');
    localStorage.removeItem('user');
    localStorage.removeItem('branchId');
    window.location.href = '/login';
  }
</script>

<svelte:head>
  <title>Terms and Conditions - Shining Light Baptist Church</title>
</svelte:head>

<div class="min-h-screen bg-gray-50 py-8">
  <div class="max-w-4xl mx-auto px-4">
    <!-- Header -->
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold text-[#002B5C]">Terms and Conditions</h1>
      <button
        on:click={logout}
        class="bg-gray-300 text-gray-700 px-4 py-2 rounded hover:bg-gray-400 transition"
      >
        Logout
      </button>
    </div>

    <!-- Error Message -->
    {#if errorMessage}
      <div class="mb-6 p-4 bg-red-100 border border-red-300 text-red-700 rounded">
        {errorMessage}
      </div>
    {/if}

    <!-- Terms Content -->
    <div class="bg-white rounded-lg shadow-md p-6 mb-6">
      <div class="prose max-w-none">
        <h2 class="text-2xl font-semibold text-[#002B5C] mb-4">Welcome to Shining Light Baptist Church Member Portal</h2>
        
        <p class="mb-4">
          By accessing and using this member portal, you agree to the following terms and conditions. Please read them carefully before proceeding.
        </p>

        <h3 class="text-xl font-semibold text-[#002B5C] mt-6 mb-3">1. Data Privacy and Confidentiality</h3>
        <ul class="list-disc list-inside mb-4 space-y-2">
          <li>All personal information provided to the church will be kept confidential and used only for church-related purposes</li>
          <li>Your contact information may be used for church communications, event notifications, and spiritual guidance</li>
          <li>Financial information is handled with the utmost security and will not be shared with third parties</li>
          <li>You have the right to access and update your personal information at any time</li>
        </ul>

        <h3 class="text-xl font-semibold text-[#002B5C] mt-6 mb-3">2. System Usage Guidelines</h3>
        <ul class="list-disc list-inside mb-4 space-y-2">
          <li>This portal is for personal use by registered members of Shining Light Baptist Church</li>
          <li>You are responsible for maintaining the confidentiality of your login credentials</li>
          <li>Any suspicious activity should be reported to church administration immediately</li>
          <li>The system should not be used for commercial or unauthorized purposes</li>
        </ul>

        <h3 class="text-xl font-semibold text-[#002B5C] mt-6 mb-3">3. Member Responsibilities</h3>
        <ul class="list-disc list-inside mb-4 space-y-2">
          <li>Keep your profile information up to date</li>
          <li>Use the system respectfully and in accordance with Christian values</li>
          <li>Report any technical issues to the system administrators</li>
          <li>Respect the privacy of other members' information</li>
        </ul>

        <h3 class="text-xl font-semibold text-[#002B5C] mt-6 mb-3">4. Church Policies</h3>
        <ul class="list-disc list-inside mb-4 space-y-2">
          <li>The church reserves the right to modify these terms as needed</li>
          <li>System access may be suspended for violation of these terms</li>
          <li>All activities are subject to church discipline and biblical principles</li>
          <li>Decisions regarding system usage are at the discretion of church leadership</li>
        </ul>

        <h3 class="text-xl font-semibold text-[#002B5C] mt-6 mb-3">5. Acceptance</h3>
        <p class="mb-4">
          By clicking "I Accept", you acknowledge that you have read, understood, and agree to be bound by these terms and conditions.
        </p>

        <p class="text-sm text-gray-600 italic">
          Last updated: {new Date().toLocaleDateString()}
        </p>
      </div>
    </div>

    <div class="text-center">
      <button
        on:click={acceptTerms}
        disabled={isLoading}
        class="bg-[#0054A4] text-white px-8 py-3 rounded-lg font-semibold text-lg
               hover:bg-[#003f91] transition disabled:opacity-60 disabled:cursor-not-allowed"
      >
        {#if isLoading}
          Processing...
        {:else}
          I Accept the Terms and Conditions
        {/if}
      </button>
    </div>
  </div>
</div>
