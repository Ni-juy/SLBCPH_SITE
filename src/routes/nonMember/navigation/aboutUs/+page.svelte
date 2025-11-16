<script>
	import Navbar from '../../../components/Navbar.svelte';
	import { onMount } from 'svelte';

	let showScrollTop = false;

	function checkScroll() {
		showScrollTop = window.innerHeight + window.scrollY >= document.body.scrollHeight - 10;
	}

	function scrollToTop() {
		window.scrollTo({ top: 0, behavior: 'smooth' });
	}

	onMount(() => {
		window.addEventListener('scroll', checkScroll);
		return () => window.removeEventListener('scroll', checkScroll);
	});
</script>

<head>
	<title>Shining Light Baptist Church</title>
</head>

<div class="min-h-screen w-full bg-gray-200">
	<link
		href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&family=Merriweather:wght@700&display=swap"
		rel="stylesheet"
	/>
	<!-- Navigation Bar -->
	<Navbar />

	<section class="relative mt-0 bg-white">
		<div
			class="pointer-events-none fixed inset-0 z-10 bg-contain bg-center bg-no-repeat"
			style="background-image: url('/logo.png'); opacity: 0.08;"
		></div>

		<div class="w-full p-6 text-center">
			<h2 class="font-church text-4xl font-bold tracking-wide text-[#003366] md:text-5xl">
				Not I, but Christ
			</h2>
		</div>

		<style>
			@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap');

			.font-church {
				font-family: 'Playfair Display', serif;
			}
		</style>

		<!-- Verse Section with Video Background -->
		<section class="relative flex w-full items-center justify-center overflow-hidden text-center">
			<div class="relative h-[400px] w-full max-w-7xl overflow-hidden rounded-2xl">
				<video autoplay loop muted playsinline class="h-full w-full object-cover">
					<source src="/videobg.mp4" type="video/mp4" />
					Your browser does not support the video tag.
				</video>
			</div>
			<div class="absolute inset-0 flex items-center justify-center px-12">
				<p class="max-w-4xl text-xl text-white italic md:text-xl">
					As Christians, we are the light of this world. The name <strong>"Shining Light"</strong> derives
					from Proverbs 4:18: “But the path of the just is as the shining light, that shineth more and
					more unto the perfect day.”
				</p>
			</div>
		</section>

		<!-- <link rel="preload" href="/fonts/BankGothic.ttf" as="font" type="font/ttf" crossorigin=true> -->

		<style>
			@font-face {
				font-family: 'BankGothic';
				src: url('/fonts/BankGothic.ttf') format('truetype');
				font-weight: normal;
				font-style: normal;
			}
		</style>

		<section class="relative mt-16 bg-white p-6">
			<div class="relative z-10 mx-auto min-h-[500px] w-full max-w-6xl p-6 text-left">
				<!-- Main Title -->
				<h3
					class="font-church mb-10 text-center text-4xl font-bold tracking-wide text-[#003366] uppercase md:text-5xl"
				>
					Constitution and By-Laws of Shining Light Baptist Church of Calamba
				</h3>

				<style>
					@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap');

					.font-church {
						font-family: 'Playfair Display', serif;
					}
				</style>

				<article class="space-y-12">
					<!-- Article I -->
					<div class="mb-8">
						<h4
							class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
						>
							ARTICLE I – NAME
						</h4>
						<p class="text-lg text-gray-900">
							The name of the church shall be Shining Light Baptist Church of Calamba. The church
							was organized January 11, 2014.
						</p>
					</div>

					<!-- Article II -->
					<div class="mb-8">
						<h4
							class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
						>
							ARTICLE II – PURPOSE & AIM
						</h4>
						<p class="mb-2 text-lg text-gray-900">
							The purpose of this church is to carry out the Great Commission as given by our Lord
							Jesus Christ in Matthew 28:19-20.
						</p>
						<blockquote class="mb-2 border-l-4 border-[#003366] pl-4 text-gray-800 italic">
							"Go ye therefore, and teach all nations, baptizing them in the name of the Father, and
							of the Son, and of the Holy Ghost: Teaching them to observe all things whatsoever I
							have commanded you: and, lo, I am with you always, even unto the end of the world.
							Amen."
						</blockquote>
						<ul class="mb-4 list-inside list-disc text-lg text-gray-900">
							<li>The winning of the lost.</li>
							<li>The baptizing of the converts.</li>
							<li>The teaching and building of the disciples in the faith.</li>
						</ul>
						<p class="text-lg text-gray-900">
							It will be the aim of this church to get the Gospel out into the world in every way
							possible.
						</p>
					</div>

					<!-- Article III -->
					<div class="mb-8">
						<h4
							class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
						>
							ARTICLE III – COVENANT
						</h4>
						<p class="mb-4 text-lg text-gray-900">
							Having been led, as we believe, by the Holy Spirit, we solemnly enter into this
							covenant with the Lord and with one another to walk together in His ways, to love one
							another, and to build up the body of Christ in truth and holiness.
						</p>
					</div>

					<!-- Article IV -->
					<div class="mb-8">
						<h4
							class="font-church mb-4 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
						>
							ARTICLE IV – STATEMENT OF FAITH
						</h4>

						<div class="mb-6">
							<h5 class="font-church mb-1 text-lg font-bold tracking-wide text-gray-900 uppercase">
								A. The Scriptures
							</h5>
							<p class="text-lg text-gray-900">
								We believe in the verbal plenary inspiration of the Scriptures and that they are the
								only authoritative rule of faith and practice.<br />
								<span class="text-gray-700 italic"
									>Scripture: 2 Timothy 3:16–17; 2 Peter 1:20–21; Hebrews 1:1–2; Jude 3; Proverbs
									30:5–6</span
								>
							</p>
						</div>

						<div class="mb-6">
							<h5 class="font-church mb-1 text-lg font-bold tracking-wide text-gray-900 uppercase">
								B. God
							</h5>
							<p class="text-lg text-gray-900">
								We believe that there is only one true and living God, eternally existing in three
								persons: Father, Son, and Holy Spirit.<br />
								<span class="text-gray-700 italic"
									>Scripture: Deuteronomy 6:4; Matthew 28:19–20; 2 Corinthians 13:14; John 4:24</span
								>
							</p>
						</div>

						<div class="mb-6">
							<h5 class="font-church mb-1 text-lg font-bold tracking-wide text-gray-900 uppercase">
								C. Jesus Christ
							</h5>
							<p class="text-lg text-gray-900">
								We believe in the deity, virgin birth, sinless life, miracles, substitutionary
								atonement, bodily resurrection, ascension, and personal return of our Lord Jesus
								Christ.
							</p>
						</div>

						<div class="mb-6">
							<h5 class="font-church mb-1 text-lg font-bold tracking-wide text-gray-900 uppercase">
								D. Holy Spirit
							</h5>
							<p class="text-lg text-gray-900">
								We believe in the present ministry of the Holy Spirit, by whose indwelling the
								believer is enabled to live a godly life.
							</p>
						</div>
					</div>

					<!-- Article V -->
					<div class="mb-8">
						<h4
							class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
						>
							ARTICLE V – MEMBERSHIP
						</h4>
						<p class="mb-4 text-lg text-gray-900">
							The membership of this church shall be composed of individuals who have publicly
							confessed Jesus Christ as their Lord and Savior and have been baptized by immersion.
						</p>
					</div>

					<!-- Remaining Articles (VI – XII) -->
					<div class="space-y-8">
						<!-- Repeat the same pattern, just replace headings with font-church -->
						<!-- Article VI -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE VI – DUTIES AND PRIVILEGES OF MEMBERS
							</h4>
							<p class="mb-4 text-lg text-gray-900">
								All members shall attend the regular meetings of the church, participate in church
								activities, and uphold the teachings and practices of the church.
							</p>
						</div>

						<!-- Article VII -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE VII – OFFICERS AND THEIR DUTIES
							</h4>
							<p class="mb-4 text-lg text-gray-900">
								The officers of the church shall include the Pastor, Deacons, and other appointed
								positions. Their duties and responsibilities shall be outlined by the church
								leadership.
							</p>
						</div>

						<!-- Article VIII -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE VIII – MEETINGS
							</h4>
							<p class="mb-4 text-lg text-gray-900">
								The church shall meet regularly for worship, fellowship, and business meetings as
								necessary, and these shall be called by the church leadership.
							</p>
						</div>

						<!-- Article IX -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE IX – CHURCH PROPERTY
							</h4>
							<p class="mb-4 text-lg text-gray-900">
								All properties acquired by the church shall be held in trust for the furtherance of
								the church’s ministry.
							</p>
						</div>

						<!-- Article X -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE X – DISCIPLINE
							</h4>
							<p class="mb-4 text-lg text-gray-900">
								The church shall exercise discipline according to biblical principles when a member
								persistently violates the teachings of the Bible or the church covenant.
							</p>
						</div>

						<!-- Article XI -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE XI – DISSOLUTION
							</h4>
							<p class="mb-4 text-lg text-gray-900">
								In the event that this church ceases to function, its assets shall be distributed in
								accordance with the purpose of the church, as determined by the church leadership.
							</p>
						</div>

						<!-- Article XII -->
						<div>
							<h4
								class="font-church mb-2 text-2xl font-bold tracking-wide text-[#003366] uppercase md:text-3xl"
							>
								ARTICLE XII – AMENDMENTS
							</h4>
							<p class="mb-2 text-lg text-gray-900">
								This constitution, except the Statement of Faith (Article IV), may be amended or
								revised at any business meeting of the church, called or annual, provided a
								four-week notice in writing of the proposed amendment has been given to the church
								membership.
							</p>
							<p class="mb-4 text-lg text-gray-900">
								All amendments must be passed by a three-fourths majority of those voting members.
							</p>
						</div>
					</div>
				</article>
			</div>
		</section>

		<!-- Church Branches & Pastors -->
		<section
			class="mx-auto mt-5 max-w-screen-xl rounded-2xl border-4 border-white bg-gradient-to-br from-[#003366] to-[#002244] px-10 py-16 text-center text-white shadow-lg md:px-20 lg:px-32"
		>
			<style>
				/* Headings - Playfair Display */
				@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap');

				.font-church {
					font-family: 'Playfair Display', serif;
				}

				/* Body text - clean sans-serif */
				.font-modern {
					font-family: 'Inter', 'Montserrat', sans-serif;
				}
			</style>

			<!-- Section Title -->
			<h3 class="font-church mb-12 text-4xl font-bold tracking-wider uppercase md:text-5xl">
				Church Branches & Pastors
			</h3>

			<div class="grid grid-cols-1 gap-10 md:grid-cols-2 lg:grid-cols-4">
				<!-- San Pedro -->
				<div
					class="mx-auto max-w-xs transform rounded-2xl border-4 border-white bg-[#00509E] p-8 shadow-xl transition duration-300 hover:scale-105 hover:ring-4 hover:ring-blue-300"
				>
					<img
						src="/PastorMichael.jpg"
						alt="Pastor Michael Cal"
						class="mx-auto mb-6 h-40 w-40 rounded-full border-4 border-white object-cover transition-transform duration-300"
					/>
					<h4 class="font-church mb-2 text-3xl tracking-wide">San Pedro</h4>
					<p class="font-modern text-lg"><span class="font-bold">Pastor:</span> Michael Cal</p>
				</div>

				<!-- Calamba -->
				<div
					class="mx-auto max-w-xs transform rounded-2xl border-4 border-white bg-[#00509E] p-8 shadow-xl transition duration-300 hover:scale-105 hover:ring-4 hover:ring-blue-300"
				>
					<img
						src="/PastorJoel.jpg"
						alt="Pastor Joel Almazan"
						class="mx-auto mb-6 h-40 w-40 rounded-full border-4 border-white object-cover transition-transform duration-300"
					/>
					<h4 class="font-church mb-2 text-3xl tracking-wide">Calamba</h4>
					<p class="font-modern text-lg"><span class="font-bold">Pastor:</span> Joel Almazan</p>
				</div>

				<!-- Calauan -->
				<div
					class="mx-auto max-w-xs transform rounded-2xl border-4 border-white bg-[#00509E] p-8 shadow-xl transition duration-300 hover:scale-105 hover:ring-4 hover:ring-blue-300"
				>
					<img
						src="/PastorVelasco.jpg"
						alt="Pastor Arnel Velasco"
						class="mx-auto mb-6 h-40 w-40 rounded-full border-4 border-white object-cover transition-transform duration-300"
					/>
					<h4 class="font-church mb-2 text-3xl tracking-wide">Calauan</h4>
					<p class="font-modern text-lg"><span class="font-bold">Pastor:</span> Arnel Velasco</p>
				</div>

				<!-- Mauro -->
				<div
					class="mx-auto max-w-xs transform rounded-2xl border-4 border-white bg-[#00509E] p-8 shadow-xl transition duration-300 hover:scale-105 hover:ring-4 hover:ring-blue-300"
				>
					<img
						src="/PastorInes.jpg"
						alt="Pastor Jeffrey Ines"
						class="mx-auto mb-6 h-40 w-40 rounded-full border-4 border-white object-cover transition-transform duration-300"
					/>
					<h4 class="font-church mb-2 text-3xl tracking-wide">Mauro</h4>
					<p class="font-modern text-lg"><span class="font-bold">Pastor:</span> Jeffrey Ines</p>
				</div>
			</div>
		</section>

		<!-- Address Section -->
		<section class="mt-5 p-6 text-center">
			<style>
				/* Headings - Playfair Display */
				@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&display=swap');

				.font-heading {
					font-family: 'Playfair Display', serif;
				}

				/* Body - clean sans-serif */
				.font-body {
					font-family: 'Inter', 'Montserrat', sans-serif;
				}
			</style>

			<h3 class="font-heading mb-10 text-4xl font-bold tracking-wider text-[#003366] md:text-5xl">
				Address of Church Per Branch
			</h3>

			<div class="mx-auto max-w-6xl space-y-16">
				<!-- San Pedro -->
				<div class="flex flex-col items-center gap-6 md:flex-row">
					<div class="flex justify-center md:w-1/2">
						<img
							src="/SanPedro.jpg"
							alt="San Pedro"
							class="h-90 w-90 rounded-full border-4 border-[#003366] bg-white object-cover p-2 shadow-lg"
						/>
					</div>
					<div class="flex flex-col justify-center text-center md:w-1/2 md:text-left">
						<h4
							class="font-heading mb-2 text-2xl font-bold tracking-wide text-[#003366] md:text-3xl"
						>
							San Pedro
						</h4>
						<p class="font-body text-[1.2rem] leading-relaxed text-gray-800">
							B21 L4 Galatians St. Adelina1, San Antonio, San Pedro Laguna.
						</p>
					</div>
				</div>

				<!-- Calamba -->
				<div class="flex flex-col items-center gap-6 md:flex-row-reverse">
					<div class="flex justify-center md:w-1/2">
						<img
							src="/Banadero.jpg"
							alt="Calamba"
							class="h-90 w-90 rounded-full border-4 border-[#003366] bg-white object-cover p-2 shadow-lg"
						/>
					</div>
					<div class="flex flex-col justify-center text-center md:w-1/2 md:text-left">
						<h4
							class="font-heading mb-2 text-2xl font-bold tracking-wide text-[#003366] md:text-3xl"
						>
							Calamba
						</h4>
						<p class="font-body text-[1.2rem] leading-relaxed text-gray-800">
							Blk 13 Lot 30 BRIA Homes, Brgy Bañadero, Calamba City, Laguna.
						</p>
					</div>
				</div>

				<!-- Calauan -->
				<div class="flex flex-col items-center gap-6 md:flex-row">
					<div class="flex justify-center md:w-1/2">
						<img
							src="/logo2.png"
							alt="Calauan"
							class="h-90 w-90 rounded-full border-4 border-[#003366] bg-white object-cover p-2 shadow-lg"
						/>
					</div>
					<div class="flex flex-col justify-center text-center md:w-1/2 md:text-left">
						<h4
							class="font-heading mb-2 text-2xl font-bold tracking-wide text-[#003366] md:text-3xl"
						>
							Calauan
						</h4>
						<p class="font-body text-[1.2rem] leading-relaxed text-gray-800">
							FCBA Silangan Subd, Brgy. Dayap, Calauan, Laguna.
						</p>
					</div>
				</div>

				<!-- Mauro -->
				<div class="flex flex-col items-center gap-6 md:flex-row-reverse">
					<div class="flex justify-center md:w-1/2">
						<img
							src="/Mauro.jpg"
							alt="Mauro"
							class="h-90 w-90 rounded-full border-4 border-[#003366] bg-white object-cover p-2 shadow-lg"
						/>
					</div>
					<div class="flex flex-col justify-center text-center md:w-1/2 md:text-left">
						<h4
							class="font-heading mb-2 text-2xl font-bold tracking-wide text-[#003366] md:text-3xl"
						>
							Mauro
						</h4>
						<p class="font-body text-[1.2rem] leading-relaxed text-gray-800">
							Block 1 Lot 4, Brgy. Bañadero, Calamba City, Laguna.
						</p>
					</div>
				</div>
			</div>
		</section>
	</section>

	<!-- Contact and Social Media -->
	<section
		class="z-20 flex flex-col justify-between overflow-auto border-t-4 border-b-4 border-black bg-gray-100 p-6 text-center md:flex-row md:text-left"
	>
		<!-- Left Side -->
		<div class="mb-4 md:mb-0 md:w-1/3">
			<h3 class="text-lg font-bold text-[#003366]">Socmed Accounts (e.g., FB)</h3>
			<div class="mt-8 flex justify-center space-x-8 md:justify-start">
				<!-- Facebook - Calamba Branch -->
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
				<!-- Facebook - San Pedro Branch -->
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

		<!-- Middle Section -->
		<div class="flex flex-col justify-between gap-8 md:w-1/3">
			<div>
				<h3 class="text-lg font-bold text-[#003366]">Contact us:</h3>
				<p><strong>San Pedro:</strong> (0921) 372 5922</p>
				<p><strong>Calamba:</strong> (0995) 460 2711</p>
				<p><strong>Calauan:</strong> (0935) 566 8843 | (0994) 722 9795</p>
				<p><strong>Mauro:</strong> (0995) 380 5525</p>
			</div>
		</div>

		<!-- Right Side (Email Section) -->
		<div class="flex flex-col justify-between md:w-1/3">
			<h3 class="text-lg font-bold text-[#003366]">Email Address:</h3>
			<p><strong>San Pedro:</strong> slbc_304@yahoo.com</p>
			<p><strong>Calamba:</strong> calamba.shininglight@gmail.com</p>
			<p><strong>Calauan:</strong> velascoarnel201@gmail.com</p>
			<p><strong>Mauro:</strong> inesjeffrey3@gmail.com</p>
		</div>
	</section>

	<footer class="overflow-auto bg-[#003366] p-4 text-center text-white">
		© Shining Light Baptist Church (Est. 2014)
	</footer>

	{#if showScrollTop}
		<button
			on:click={scrollToTop}
			class="fixed right-6 bottom-6 z-50 flex h-12 w-12 items-center justify-center rounded-full bg-black text-white shadow-lg hover:bg-[#00264d]"
			aria-label="Scroll to top"
		>
			<!-- SVG up arrow icon -->
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
