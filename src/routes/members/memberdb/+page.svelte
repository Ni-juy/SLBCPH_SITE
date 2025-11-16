<script>
	import Layout from '../../layout/layout.svelte';
</script>

<head>
	<title>Shining Light Baptist Church</title>
</head>

<Layout>
	<span slot="header">Dashboard</span>

	<div class="grid grid-cols-1 gap-6 md:grid-cols-2">
		<div
			class="flex items-center justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md transition-transform duration-300 hover:scale-[1.02] hover:shadow-xl"
		>
			<h2 class="text-lg font-bold transition-colors duration-300 hover:text-blue-700">
				UPCOMING EVENTS
			</h2>
			<a
				href="/members/events"
				class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white transition-colors duration-300 hover:bg-gray-600"
			>
				View
			</a>
		</div>

		<div
			class="flex items-center justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md transition-transform duration-300 hover:scale-[1.02] hover:shadow-xl"
		>
			<h2 class="text-lg font-bold transition-colors duration-300 hover:text-blue-700">
				PRAYER REQUESTS / BLESSINGS OF THE DAY
			</h2>
			<a
				href="/members/pr"
				class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white transition-colors duration-300 hover:bg-gray-600"
			>
				Send
			</a>
		</div>

		<div
			class="flex items-center justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md transition-transform duration-300 hover:scale-[1.02] hover:shadow-xl"
		>
			<h2 class="text-lg font-bold transition-colors duration-300 hover:text-blue-700">
				MY ATTENDANCE
			</h2>
			<a
				href="/members/ssm"
				class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white transition-colors duration-300 hover:bg-gray-600"
			>
				Monitor
			</a>
		</div>

		<div
			class="flex items-center justify-between rounded-lg bg-blue-100 p-6 text-black shadow-md transition-transform duration-300 hover:scale-[1.02] hover:shadow-xl"
		>
			<h2 class="text-lg font-bold transition-colors duration-300 hover:text-blue-700">
				TITHES AND OFFERINGS
			</h2>
			<a
				href="/members/financial"
				class="rounded-lg bg-blue-600 px-6 py-2 font-semibold text-white transition-colors duration-300 hover:bg-gray-600"
			>
				View
			</a>
		</div>
	</div>
</Layout>
