<script lang="ts">
  let activeMenu: string | null = null;
  let sidebarOpen = false; // Controls sidebar visibility (for mobile)

  


  // Function to open/close sidebar (for mobile view)
  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }

  // Function to close the sidebar (when clicking outside or selecting a menu item)
  function closeSidebar() {
    sidebarOpen = false;
  }

  // Function to toggle dropdown menus (Manage Members & Manage Events)
  function toggleDropdown(menu: any) {
    activeMenu = activeMenu === menu ? null : menu;
  }
</script>

<div class="flex h-screen bg-gray-100">
  <!-- Sidebar -->
  <aside class={`w-64 bg-gradient-to-b from-yellow-400 to-red-500 text-white p-4 transition-transform 
                ${sidebarOpen ? "translate-x-0" : "-translate-x-64"} md:translate-x-0 fixed md:relative h-full z-50`}>
    <div class="flex flex-col items-center">
      <div class="w-20 h-20 rounded-full bg-gray-700 flex items-center justify-center">
        <span class="text-3xl">👤</span>
      </div>
    </div>
    <nav class="mt-6">
      <button class="block p-2 rounded bg-red-600" on:click={closeSidebar}>📊 Dashboard</button>

      <div class="mt-2">
        <button class="w-full text-left p-2 hover:bg-red-500 rounded flex items-center" on:click={() => toggleDropdown('members')}>
          👥 Manage Members {activeMenu === 'members' ? '▲' : '▼'}
        </button>
        {#if activeMenu === 'members'}
          <div class="ml-4 border-l-2 border-gray-300 pl-2 mt-1">
            <button class="p-2 hover:bg-red-500 rounded flex items-center" on:click={closeSidebar}>
              👤 Member Details
            </button>
            <button class="p-2 hover:bg-red-500 rounded flex items-center" on:click={closeSidebar}>
              ➕ Add Member
            </button>
          </div>
        {/if}
      </div>

      <button class="block p-2 mt-2 hover:bg-red-500 rounded" on:click={closeSidebar}>🛐 Manage Sunday Service Monitoring</button>
      <button class="block p-2 mt-2 hover:bg-red-500 rounded" on:click={closeSidebar}>🙏 Manage Prayer Requests</button>
      <button class="block p-2 mt-2 hover:bg-red-500 rounded" on:click={closeSidebar}>💰 Financial Manager</button>

      <div class="mt-2">
        <button class="w-full text-left p-2 hover:bg-red-500 rounded flex items-center" on:click={() => toggleDropdown('events')}>
          📅 Manage Event {activeMenu === 'events' ? '▲' : '▼'}
        </button>
        {#if activeMenu === 'events'}
          <div class="ml-4 border-l-2 border-gray-300 pl-2 mt-1">
            <button class="p-2 hover:bg-red-500 rounded flex items-center" on:click={closeSidebar}>
              ➕ Add Event
            </button>
            <button class="p-2 hover:bg-red-500 rounded flex items-center" on:click={closeSidebar}>
              📆 Upcoming Events
            </button>
          </div>
        {/if}
      </div>
    </nav>
  </aside>

  {#if sidebarOpen}
    <!-- svelte-ignore a11y_consider_explicit_label -->
    <button class="fixed inset-0 bg-black opacity-50 md:hidden z-40" on:click={closeSidebar}></button>
  {/if}

  <div class="flex-1">
    <header class="bg-white shadow p-4 rounded-lg w-full border-b-4 border-yellow-400 flex justify-between items-center">
      <button class="md:hidden text-2xl" on:click={toggleSidebar}>☰</button>
      <h1 class="text-xl font-bold">Dashboard</h1>
      <div class="flex items-center space-x-2">
        <span class="text-gray-700 font-semibold">Admin Riyu</span>
        <span class="text-2xl">👤</span>
      </div>
    </header>
  </div>
</div>