<script>
  import { onMount } from 'svelte';

  let isOpen = false;
  let currentPath = '';

  function toggleMenu() {
    isOpen = !isOpen;
  }

  onMount(() => {
    currentPath = window.location.pathname;
  });
</script>

<!-- NAVIGATION -->
<nav class="w-full flex items-center justify-between px-6 py-4 bg-[#003366] shadow-md font-sans relative z-50">
  <!-- Logo -->
  <div class="flex items-center">
    <img src="/logo.png" alt="Logo" class="h-12 w-12 mr-3" />
    <span class="text-lg md:text-xl font-semibold text-white tracking-wide">
      Shining Light Baptist Church
    </span>
  </div>

  <!-- svelte-ignore a11y_consider_explicit_label -->
  <button class="md:hidden focus:outline-none" on:click={toggleMenu}>
    <svg class="w-7 h-7 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
        d="M4 6h16M4 12h16M4 18h16" />
    </svg>
  </button>

  <div class="hidden md:flex gap-8 ml-20 text-base text-white font-medium">
    <a href="/" class="hover:text-[#FFD700] transition border-b-2 border-transparent hover:border-[#FFD700] {currentPath === '/' ? 'text-[#FFD700] font-semibold' : ''}">Home</a>
    <a href="/nonMember/navigation/aboutUs" class="hover:text-[#FFD700] transition border-b-2 border-transparent hover:border-[#FFD700] {currentPath === '/nonMember/navigation/aboutUs' ? 'text-[#FFD700] font-semibold' : ''}">About</a>
    <a href="/nonMember/navigation/online" class="hover:text-[#FFD700] transition border-b-2 border-transparent hover:border-[#FFD700] {currentPath === '/nonMember/navigation/online' ? 'text-[#FFD700] font-semibold' : ''}">Give</a>
    <a href="/nonMember/navigation/faqs" class="hover:text-[#FFD700] transition border-b-2 border-transparent hover:border-[#FFD700] {currentPath === '/nonMember/navigation/faqs' ? 'text-[#FFD700] font-semibold' : ''}">FAQs</a>
  </div>

  <div class="hidden md:block">
    <a href="../../../login" class="bg-[#CC0000] text-white px-5 py-2 rounded-lg text-base font-medium hover:brightness-110 transition">
      Login
    </a>
  </div>

  {#if isOpen}
    <div class="absolute top-full left-0 w-full bg-white flex flex-col items-center shadow-md md:hidden text-gray-700 font-medium">
      <a href="/" class="py-3 w-full text-center hover:bg-gray-100 {currentPath === '/' ? 'bg-[#FFD700] text-white font-semibold' : ''}">Home</a>
      <a href="/nonMember/navigation/aboutUs" class="py-3 w-full text-center hover:bg-gray-100 {currentPath === '/nonMember/navigation/aboutUs' ? 'bg-[#FFD700] text-white font-semibold' : ''}">About</a>
      <a href="/nonMember/navigation/online" class="py-3 w-full text-center hover:bg-gray-100 {currentPath === '/nonMember/navigation/online' ? 'bg-[#FFD700] text-white font-semibold' : ''}">Give</a>
      <a href="/nonMember/navigation/faqs" class="py-3 w-full text-center hover:bg-gray-100 {currentPath === '/nonMember/navigation/faqs' ? 'bg-[#FFD700] text-white font-semibold' : ''}">FAQs</a>
      <a href="../../../login" class="py-3 w-full text-center bg-[#CC0000] text-white hover:brightness-110">Login</a>
    </div>
  {/if}
</nav>
