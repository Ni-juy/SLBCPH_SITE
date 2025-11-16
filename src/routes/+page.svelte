<svelte:head>
	<title>Shining Light Baptist Church</title>
</svelte:head>

<script lang="ts">
	import Navbar from './components/Navbar.svelte';
	import { onMount } from 'svelte';


	let slide1: HTMLElement;
	let slide2: HTMLElement;
	let slide3: HTMLElement;
	let slides: HTMLElement[];
	let currentIndex  = 0;
	let showScrollTop = false;

	onMount(() => {
		slides = [slide1, slide2, slide3];
		const interval = setInterval(() => {
			slides[currentIndex].classList.replace('opacity-100', 'opacity-0');
			currentIndex = (currentIndex + 1) % slides.length;
			slides[currentIndex].classList.replace('opacity-0', 'opacity-100');
		}, 4000);

		window.addEventListener('scroll', checkScroll);
		checkScroll();

		return () => {
			clearInterval(interval);
			window.removeEventListener('scroll', checkScroll);
		};
	});

	function checkScroll() {
		const bottomReached = window.innerHeight + window.scrollY >= document.body.scrollHeight - 10;
		showScrollTop = bottomReached;
	}

	function scrollToTop() {
		window.scrollTo({ top: 0, behavior: 'smooth' });
	}
</script>

<div class="min-h-screen w-full bg-gray-200">
	<Navbar />
	<section class="relative aspect-[16/9] w-full overflow-hidden bg-white md:min-h-[700px]">
		<video class="absolute inset-0 h-full w-full object-cover" autoplay loop muted playsinline>
			<source
				src="https://res.cloudinary.com/dkknfnzxb/video/upload/v1756053237/3D_LOG_WHITE_mspifq.mp4"
				type="video/mp4"
			/>
		</video>
		<div class="absolute inset-0"></div>
	</section>

	<section class="relative aspect-[16/9] w-full overflow-hidden bg-white md:min-h-[700px]">
		<video class="absolute inset-0 h-full w-full object-cover" autoplay loop muted playsinline>
			<source
				src="https://res.cloudinary.com/dkknfnzxb/video/upload/v1756054164/slbc-mission_e9ppye.mp4"
				type="video/mp4"
			/>
		</video>
		<div class="absolute inset-0 bg-black/30"></div>
	</section>

	<section class="relative h-[600px] w-full overflow-hidden bg-[#003366] md:h-[700px] lg:h-[800px]">
		<div class="relative h-full w-full">
			<div
				class="absolute inset-0 opacity-100 transition-opacity duration-1000 ease-in-out"
				bind:this={slide1}
			>
				<img src="/bgpic.jpg" alt="Slide 1" class="h-full w-full object-cover opacity-60" />
				<div
					class="absolute inset-0 mx-auto flex max-w-[90%] flex-col items-center justify-center px-4 text-center text-white"
				>
					<h2 class="text-3xl font-bold md:text-4xl">Church Anniversary Celebration</h2>
					<p class="text-sm md:text-base">
						A memorable day filled with worship and fellowship across all branches.
					</p>
				</div>
			</div>

			<div
				class="absolute inset-0 opacity-0 transition-opacity duration-1000 ease-in-out"
				bind:this={slide2}
			>
				<img src="/samplepic1.jpg" alt="Slide 2" class="h-full w-full object-cover opacity-60" />
				<div
					class="absolute inset-0 mx-auto flex max-w-[90%] flex-col items-center justify-center px-4 text-center text-white"
				>
					<h2 class="text-3xl font-bold md:text-4xl">Youth Camp 2024</h2>
					<p class="text-sm md:text-base">
						Empowering the next generation with Christ-centered values.
					</p>
				</div>
			</div>

			<div
				class="absolute inset-0 opacity-0 transition-opacity duration-1000 ease-in-out"
				bind:this={slide3}
			>
				<img src="/samplepic3.jpg" alt="Slide 3" class="h-full w-full object-cover opacity-60" />
				<div
					class="absolute inset-0 mx-auto flex max-w-[90%] flex-col items-center justify-center px-4 text-center text-white"
				>
					<h2 class="text-3xl font-bold md:text-4xl">Community Outreach</h2>
					<p class="text-sm md:text-base">
						Sharing God’s love through service in local communities.
					</p>
				</div>
			</div>
		</div>
	</section>

	<section
		class="flex flex-col justify-between overflow-auto border-t-4 border-b-4 border-black bg-gray-100 p-6 text-center md:flex-row md:text-left"
	>
		<div class="mb-4 md:mb-0 md:w-1/3">
			<h3 class="text-lg font-bold text-[#003366]">Socmed Accounts (e.g., FB)</h3>
			<div class="mt-8 flex justify-center space-x-8 md:justify-start">
				<div class="flex flex-col items-center">
					<a href="https://www.facebook.com/ShiningLightCalamba" target="_blank">
						<img
							src="https://upload.wikimedia.org/wikipedia/commons/5/51/Facebook_f_logo_%282019%29.svg"
							class="h-10 w-10 cursor-pointer rounded-full object-cover"
							alt="Facebook Logo"
						/>
					</a>
					<p class="mt-1 text-sm font-semibold text-black">Calamba Branch</p>
				</div>
				<div class="flex flex-col items-center">
					<a href="https://www.facebook.com/SLBCSanPedroLaguna" target="_blank">
						<img
							src="https://upload.wikimedia.org/wikipedia/commons/5/51/Facebook_f_logo_%282019%29.svg"
							class="h-10 w-10 cursor-pointer rounded-full object-cover"
							alt="Facebook Logo"
						/>
					</a>
					<p class="mt-1 text-sm font-semibold text-black">San Pedro Branch</p>
				</div>
			</div>
		</div>

		<div class="flex flex-col justify-between gap-8 md:w-1/3">
			<div>
				<h3 class="text-lg font-bold text-[#003366]">Contact us:</h3>
				<p><strong>San Pedro:</strong> (0921) 372 5922</p>
				<p><strong>Calamba:</strong> (0995) 460 2711</p>
				<p><strong>Calauan:</strong> (0935) 566 8843 | (0994) 722 9795</p>
				<p><strong>Mauro:</strong> (0995) 380 5525</p>
			</div>
		</div>

		<div class="flex flex-col justify-between md:w-1/3">
			<h3 class="text-lg font-bold text-[#003366]">Email Address:</h3>
			<p><strong>San Pedro:</strong> slbc_304@yahoo.com</p>
			<p><strong>Calamba:</strong> calamba.shininglight@gmail.com</p>
			<p><strong>Calauan:</strong> velascoarnel201@gmail.com</p>
			<p><strong>Mauro:</strong> inesjeffrey3@gmail.com</p>
		</div>
	</section>

	<footer
		class="overflow-auto bg-[#003366] p-4 text-center text-white"
		style="font-family: 'Times New Roman', serif;"
	>
		© Shining Light Baptist Church (Est. 2014)
	</footer>

	{#if showScrollTop}
		<button
			on:click={scrollToTop}
			class="fixed right-6 bottom-6 z-50 flex h-12 w-12 items-center justify-center rounded-full bg-red-500 shadow-2xl text-white hover:bg-red-600"
			aria-label="Scroll to top"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				class="h-6 w-6"
				fill="none"
				viewBox="0 0 24 24"
				stroke="currentColor"
				stroke-width="2"
			>
				<path stroke-linecap="round" stroke-linejoin="round" d="M5 15l7-7 7 7" />
			</svg>
		</button>
	{/if}
</div>
