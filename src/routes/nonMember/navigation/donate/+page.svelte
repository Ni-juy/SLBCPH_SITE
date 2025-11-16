<script lang="ts">
	import Swal from 'sweetalert2';
	import Navbar from '../../../components/Navbar.svelte';

	let amount = '';
	let message = '';
	let currentImage: string = '/default-image.jpg';
	let newImage: File | null = null;
	let referenceNumber = '';
	let email = '';
	let otp = '';
	let otpRequested = false;
	let otpVerified = false;
	let showDonationFields = false;
	let branches: any = [];
	let selectedBranchId = '';
	let name = '';

	function handleImageChange(e: Event) {
		const file = (e.target as HTMLInputElement).files?.[0];
		if (!file) return;
		newImage = file;
		const reader = new FileReader();
		reader.onload = () => (currentImage = reader.result as string);
		reader.readAsDataURL(file);
	}

	const changeImage = () => (document.getElementById('imageInput') as HTMLInputElement)?.click();

	function showLoader(text = 'Please wait…') {
		Swal.fire({
			title: text,
			allowOutsideClick: false,
			allowEscapeKey: false,
			didOpen() {
				Swal.showLoading();
			}
		});
	}

	async function handleBadResponse(res: Response) {
		const ct = res.headers.get('content-type') || '';
		let msg = `HTTP ${res.status}`;
		if (ct.includes('application/json')) {
			const j = await res.json();
			if (j.errors) {
				const firstField = Object.keys(j.errors)[0];
				const firstMessage = j.errors[firstField][0];
				msg = firstMessage;
			} else if (j.message) {
				msg = j.message;
			} else {
				msg = JSON.stringify(j);
			}
		} else {
			msg = await res.text();
		}
		throw new Error(msg);
	}

	/* ──────────────────────── validation ───────────────────────── */
	function validateForm(): boolean {
		if (!/^\d{13}$/.test(referenceNumber.trim())) {
			Swal.fire('Invalid Reference', 'GCash reference number must be exactly 13 digits.', 'error');
			return false;
		}
		const amountValue = parseFloat(amount);
		if (isNaN(amountValue) || amountValue <= 0) {
			Swal.fire('Invalid Amount', 'Donation amount must be greater than ₱0.00.', 'error');
			return false;
		}
		return true;
	}

	/* ──────────────────────── API actions ───────────────────────── */
	async function requestOtp() {
		try {
			showLoader('Sending OTP…');
			const res = await fetch('https://slbcph.site/api/request-otp', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email })
			});
			if (!res.ok) await handleBadResponse(res);
			otpRequested = true;
			Swal.fire('Success', 'OTP sent to your email.', 'success');
		} catch (err: any) {
			Swal.fire('Error', err.message || 'Network error.', 'error');
		}
	}

	async function fetchBranches() {
		try {
			const res = await fetch('https://slbcph.site/api/branches-for-donation');
			if (!res.ok) throw new Error('Failed to fetch branches');
			branches = await res.json();
		} catch (err: any) {
			Swal.fire('Error', err.message || 'Failed to load branches.', 'error');
		}
	}

	fetchBranches();
	async function verifyOtp() {
		try {
			showLoader('Verifying OTP…');
			const res = await fetch('https://slbcph.site/api/verify-otp', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email, otp })
			});
			if (!res.ok) await handleBadResponse(res);
			const data = await res.json();
			if (data.success) {
				otpVerified = true;
				showDonationFields = true; 
				Swal.fire('Verified', 'OTP accepted. You may now donate.', 'success');
			} else {
				Swal.fire('Invalid OTP', data.message || 'Please try again.', 'error');
			}
		} catch (err: any) {
			Swal.fire('Error', err.message || 'Network error.', 'error');
		}
	}

	async function submitDonation(e: Event) {
		e.preventDefault();
		if (!validateForm()) return;
		if (!otpVerified) {
			Swal.fire('OTP required', 'Verify OTP before submitting.', 'warning');
			return;
		}
		if (!selectedBranchId) {
			Swal.fire('Branch Required', 'Please select a branch to donate to.', 'warning');
			return;
		}
		const formData = new FormData();
		formData.append('name', name);
		formData.append('referenceNumber', referenceNumber);
		formData.append('amount', amount);
		formData.append('message', message);
		formData.append('branch_id', selectedBranchId);
		if (newImage) formData.append('image', newImage);
		try {
			showLoader('Submitting donation…');
			const res = await fetch('https://slbcph.site/api/submit-donation', {
				method: 'POST',
				headers: {
					Accept: 'application/json',
					Authorization: `Bearer ${localStorage.getItem('authToken') || ''}`
				},
				body: formData
			});
			if (!res.ok) await handleBadResponse(res);
			Swal.fire('Thank you!', 'Donation submitted successfully.', 'success');
			resetForm();
		} catch (err: any) {
			Swal.fire('Error', err.message || 'Donation failed.', 'error');
		}
	}

	function resetForm() {
		name = '';
		referenceNumber = '';
		email = '';
		otp = '';
		otpRequested = false;
		otpVerified = false;
		selectedBranchId = '';
		showDonationFields = false;
		currentImage = '/default-image.jpg';
		newImage = null;
	}
</script>

<svelte:head>
	<title>Shining Light Baptist Church</title>
	<!-- SweetAlert2 CDN -->
	<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
</svelte:head>

<div class="flex min-h-screen w-full flex-col bg-gray-200">
	<Navbar />
	<main class="flex flex-grow flex-col">
		<!-- -------------- hero / form -------------- -->
		<section
			class="flex flex-grow flex-col items-center justify-center bg-cover bg-center p-8 md:flex-row"
			style="background-image:url('/bg2.jpg')"
		>
			<!-- why donate -->
			<div
				class="rounded-lg border border-white/70 bg-white/60 p-6 text-center shadow-md md:w-1/2 md:text-left"
			>
				<h2 class="mb-4 text-3xl font-bold text-[#003366]">Why Donate?</h2>
				<p class="text-base text-black md:text-lg">
					Your generosity helps us continue spreading the Word and serving the community. Through
					your pledges, we bring light where it’s most needed.
				</p>
			</div>
			<!-- donation form -->
			<div
				class="mt-8 rounded-lg border border-white/70 bg-white/60 p-6 shadow-lg md:mt-0 md:ml-6 md:w-1/2"
			>
				<h2 class="mb-4 text-2xl font-semibold text-[#003366]">Donation Form (GCASH)</h2>
				<form on:submit={submitDonation}>
					{#if !showDonationFields}
						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="mb-2 block text-lg">Name (Optional)</label>
						<input
							id="name"
							bind:value={name}
							class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
						/>

						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="mb-2 block text-lg">Select Branch to Donate (Required)</label>
						<select
							bind:value={selectedBranchId}
							class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
							required
						>
							<option value="" disabled selected>Select a branch</option>
							{#each branches as branch}
								<option value={branch.id}>{branch.name} ({branch.branch_type})</option>
							{/each}
						</select>

						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="mb-2 block text-lg">Email (Required for OTP)</label>
						<input
							id="email"
							type="email"
							bind:value={email}
							required
							class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
						/>

						{#if !otpRequested}
							<button
								type="button"
								on:click={requestOtp}
								class="mb-4 rounded-lg bg-[#003366] px-6 py-2 text-white"
							>
								Request OTP
							</button>
						{:else if !otpVerified}
							<!-- svelte-ignore a11y_label_has_associated_control -->
							<label class="mb-2 block text-lg">Enter OTP</label>
							<input
								id="otp"
								maxlength="6"
								bind:value={otp}
								class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
							/>
							<button
								type="button"
								on:click={verifyOtp}
								class="mb-4 rounded-lg bg-[#003366] px-6 py-2 text-white"
							>
								Verify OTP
							</button>
						{/if}
					<!-- svelte-ignore a11y_label_has_associated_control -->
					{:else}
						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="mb-2 block text-lg">Reference Number (Required)</label>
						<input
							id="referenceNumber"
							bind:value={referenceNumber}
							required
							class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
						/>

						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="mb-2 block text-lg">Donation Amount (Required)</label>
						<input
							id="amount"
							type="number"
							step="0.01"
							min="0.01"
							bind:value={amount}
							required
							class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
						/>

						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="mb-2 block text-lg">Message (Optional)</label>
						<textarea
							id="message"
							bind:value={message}
							class="mb-4 w-full rounded-lg border bg-white px-4 py-2"
						></textarea>

						<!-- QR -->
						<div class="mt-8 text-center">
							<h3 class="mb-2 text-lg font-bold text-[#003366]">Or Scan to Donate via GCash</h3>
							<img
								src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=GCASH-1234567890"
								class="mx-auto rounded-lg border bg-white p-2" alt="TEST"
							/>
							<p class="mt-2 text-sm text-gray-700">Sample QR — replace with your own</p>
						</div>

						<!-- Image upload -->
						<div class="mt-8 text-center">
							<h3 class="mb-2 text-lg font-bold text-[#003366]">Upload Proof of Donation</h3>
							<img
								src={currentImage}
								class="mx-auto mb-4 h-40 w-40 rounded-lg border bg-white object-cover p-2"
                                alt="Current Proof of Donation"
							/>
							<input
								id="imageInput"
								type="file"
								accept="image/*"
								hidden
								on:change={handleImageChange}
							/>
							<button
								type="button"
								on:click={changeImage}
								class="rounded-lg bg-[#CC0000] px-6 py-2 text-white"
							>
								Add Image
							</button>
						</div>

						<div class="mt-10 text-center">
							<button type="submit" class="rounded-lg bg-[#003366] px-6 py-3 text-white">
								Submit Donation
							</button>
						</div>
					{/if}
				</form>
			</div>
		</section>
	</main>
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
	<footer class="overflow-auto bg-[#003366] p-4 text-center text-white">
		© Shining Light Baptist Church (Est. 2014)
	</footer>
</div>
