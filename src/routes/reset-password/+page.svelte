<script>
	import RequestHandler from '$lib/request';
  import { onMount } from 'svelte';

  let token = '';
  let email = '';
  let password = '';
  let passwordConfirmation = '';
  let message = '';
  let error = '';

  onMount(() => {
    const urlParams = new URLSearchParams(window.location.search);
    token = urlParams.get('token') || '';
    email = urlParams.get('email') || '';
  });

  async function resetPassword() {
    try {
      const data = await RequestHandler.fetchData('POST', 'reset-password', {
        token,
        email,
        password,
        password_confirmation: passwordConfirmation
      }, {
        'Content-Type': 'application/json'
      });

      if (data.success) {
        message = '✅ Password has been reset. You can now log in.';
        error = '';
      } else {
        error = data.message || 'Password reset failed.';
      }

    } catch (e) {
      error = 'Something went wrong.';
    }
  }
</script>

<h1 class="text-2xl font-bold text-center my-4">Reset Your Password</h1>

{#if message}
  <div class="text-green-600 text-center my-2">{message}</div>
{:else}
  {#if error}
    <div class="text-red-600 text-center my-2">{error}</div>
  {/if}

  <form on:submit|preventDefault={resetPassword} class="max-w-md mx-auto space-y-4">
    <div>
      <!-- svelte-ignore a11y_label_has_associated_control -->
      <label>Email</label>
      <input type="email" bind:value={email} class="w-full border p-2 rounded" readonly />
    </div>

    <div>
      <!-- svelte-ignore a11y_label_has_associated_control -->
      <label>New Password</label>
      <input type="password" bind:value={password} class="w-full border p-2 rounded" required />
    </div>

    <div>
      <!-- svelte-ignore a11y_label_has_associated_control -->
      <label>Confirm Password</label>
      <input type="password" bind:value={passwordConfirmation} class="w-full border p-2 rounded" required />
    </div>

    <button type="submit" class="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700">
      Reset Password
    </button>
  </form>
{/if}
