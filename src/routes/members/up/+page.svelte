<script lang="ts">
	import { url } from '$lib';
	import RequestHandler from '$lib/request';
	import Swal from 'sweetalert2';
	import Layout from '../../layout/layout.svelte';
	import { onMount } from 'svelte';

	const passwordPattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^\w\s]).{8,}$/; // ≥8, 1 upper, 1 lower, 1 digit, 1 special
	let today = new Date().toISOString().split('T')[0]; // e.g. "2025-09-08"

	let userId: number | string = '';
	let username = '';
	let email = '';
	let firstName = '';
	let middleName = '';
	let lastName = '';
	let contact = '';
	let address = '';
	let birthdate = ''; // yyyy-mm-dd
	let gender = ''; // Male | Female | Other
	let baptismDate = ''; // yyyy-mm-dd
	let salvationDate = ''; // yyyy-mm-dd
	let status = ''; // Active | Inactive  etc.

	let currentPassword = '';
	let newPassword = '';
	let confirmPassword = '';

	let profileImage = '';
	let selectedFile: File | null = null;

	/* ── branch-transfer state ─────────────────── */
	let branchOptions: any[] = [];
	let currentBranchId: number | string | null = null;
	let requestedBranchId = '';
	let transferReason = '';

	// individual rule flags
	$: hasUpper = /[A-Z]/.test(newPassword);
	$: hasLower = /[a-z]/.test(newPassword);
	$: hasNumber = /\d/.test(newPassword);
	$: hasSpecial = /[^\w\s]/.test(newPassword);
	$: longEnough = newPassword.length >= 8;

	// $: allGood = hasUpper && hasLower && hasNumber && hasSpecial && longEnough;
	$: mismatch = confirmPassword && newPassword !== confirmPassword;

	onMount(async () => {
		await fetchUserData();
		await fetchBranches();
	});

	// Helper function to format date strings to yyyy-mm-dd
	function formatDateToInput(dateStr: string | null): string {
		if (!dateStr) return '';
		const d = new Date(dateStr);
		if (isNaN(d.getTime())) return '';
		const year = d.getFullYear();
		const month = (d.getMonth() + 1).toString().padStart(2, '0');
		const day = d.getDate().toString().padStart(2, '0');
		return `${year}-${month}-${day}`;
	}

	async function fetchUserData() {
		const token = localStorage.getItem('authToken');
		if (!token) return;

		try {
			const res = await fetch(`${url}/api/user`, {
				headers: { Authorization: `Bearer ${token}` }
			});

			if (!res.ok) throw new Error(await res.text());
			const u = await res.json();

			userId = u.id;
			username = u.username;
			email = u.email;
			firstName = u.first_name ?? '';
			middleName = u.middle_name ?? '';
			lastName = u.last_name ?? '';
			contact = u.contact_number ?? '';
			address = u.address ?? '';
			birthdate = formatDateToInput(u.birthdate ?? '');
			gender = u.gender ?? '';
			baptismDate = formatDateToInput(u.baptism_date ?? '');
			salvationDate = formatDateToInput(u.salvation_date ?? '');
			status = u.status ?? '';
			profileImage = u.profile_image;
			currentBranchId = u.branch_id ?? null;
		} catch (err) {
			console.error('fetchUserData →', err);
		}
	}

	async function fetchBranches() {
		try {
			const res = await fetch(`${url}/api/branches`);
			if (res.ok) {
				const all = await res.json();
				// Filter branches to only include 'Organized' and 'Main' types
				branchOptions = all.filter(
					(b: any) =>
						b.id !== currentBranchId && (b.branch_type === 'Organized' || b.branch_type === 'Main')
				);
			}
		} catch (err) {
			console.error(err);
		}
	}

	async function handleImageUpload(e: Event) {
		const fileInput = document.getElementById('imageUpload') as HTMLInputElement;
		if (!fileInput || !fileInput.files?.length) return;

		const file = fileInput.files[0];
		const formData = new FormData();
		formData.append('profile_image', file);
		formData.append('userId', userId.toString());

		try {
			const res = await fetch('https://slbcph.site/api/upload-profile', {
				method: 'POST',
				body: formData
			});

			const data = await res.json();
			console.log(data);

			if (!res.ok) throw new Error(data.message);

			profileImage = data.url;

			Swal.fire({
				icon: 'success',
				title: 'Uploaded!',
				text: 'Image uploaded successfully.',
				timer: 2000,
				showConfirmButton: false
			});
		} catch (err) {
			console.error('handleImageUpload →', err);
			Swal.fire({
				icon: 'error',
				title: 'Upload Failed',
				text: 'Something went wrong while uploading your photo.'
			});
		}
	}

	function togglePassword(id: string) {
		const input = document.getElementById(id) as HTMLInputElement;
		if (input) {
			input.type = input.type === 'password' ? 'text' : 'password';
		}
	}

	/*  save general info  */
	async function updateProfile() {
		const confirm = await Swal.fire({
			title: 'Save Changes?',
			text: 'Are you sure you want to update your profile information?',
			icon: 'question',
			showCancelButton: true,
			confirmButtonText: 'Yes, update it',
			cancelButtonText: 'Cancel'
		});

		// User clicked Cancel
		if (!confirm.isConfirmed) return;

		const token = localStorage.getItem('authToken');
		if (!token) return;

		const fd = new FormData();
		fd.append('user_id', userId.toString());
		fd.append('username', username);
		fd.append('email', email);
		fd.append('first_name', firstName);
		fd.append('middle_name', middleName);
		fd.append('last_name', lastName);
		fd.append('contact_number', contact);
		fd.append('address', address);
		fd.append('birthdate', birthdate);
		fd.append('gender', gender);
		fd.append('baptism_date', baptismDate);
		fd.append('salvation_date', salvationDate);

		// Laravel treats this as a PUT request
		fd.append('_method', 'PUT');

		try {
			const res = await fetch(`https://slbcph.site/api/user/update`, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${token}`
					// Don't manually set Content-Type here
				},
				body: fd
			});

			if (!res.ok) throw new Error(await res.text());

			Swal.fire({
				icon: 'success',
				title: 'Profile Updated',
				text: 'Your profile information was updated successfully.',
				timer: 2000,
				showConfirmButton: false
			});
		} catch (err) {
			console.error('updateProfile →', err);
			Swal.fire({
				icon: 'error',
				title: 'Update Failed',
				text: 'Something went wrong while updating your profile.'
			});
		}
	}

	/* ── change password ───────────────────────── */
	async function changePassword() {
		/* 1️⃣  Front-end validation -------------------------------------- */
		const mismatch = newPassword !== confirmPassword;
		const allGood = passwordPattern.test(newPassword); // same rule used by the checklist

		if (!allGood) {
			return Swal.fire({
				icon: 'warning',
				title: 'Weak Password',
				html:
					'Password must have at least <b>8 characters</b>, ' +
					'<b>1 uppercase</b>, <b>1 lowercase</b>, <b>1 number</b> ' +
					'and <b>1 special character</b>.'
			});
		}

		if (mismatch) {
			return Swal.fire({
				icon: 'error',
				title: 'Passwords Don’t Match'
			});
		}

		/* 2️⃣  Ask for confirmation -------------------------------------- */
		const { isConfirmed } = await Swal.fire({
			title: 'Change Password?',
			text: 'You’ll need the new password next time you log in.',
			icon: 'question',
			showCancelButton: true,
			confirmButtonText: 'Yes, change it'
		});
		if (!isConfirmed) return;

		/* 3️⃣  Send request to API --------------------------------------- */
		const token = localStorage.getItem('authToken');
		if (!token) return;

		try {
			const res = await fetch(`${url}/api/user/change-password`, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${token}`,
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					current_password: currentPassword,
					new_password: newPassword,
					new_password_confirmation: confirmPassword
				})
			});

			const data = await res.json();
			if (!res.ok) throw new Error(data.message || 'Server error');

			await Swal.fire({
				icon: 'success',
				title: 'Password Updated',
				timer: 1800,
				showConfirmButton: false
			});

			// reset fields
			currentPassword = newPassword = confirmPassword = '';
		} catch (err: any) {
			Swal.fire({
				icon: 'error',
				title: 'Update Failed',
				text: err.message
			});
		}
	}

	/* ── branch transfer ───────────────────────── */
	async function submitTransferRequest() {
		const token = localStorage.getItem('authToken');

		// ✅ basic validation
		if (!token || !requestedBranchId || requestedBranchId == currentBranchId) {
			return Swal.fire({
				icon: 'warning',
				title: 'Invalid Branch Selection',
				text: 'Please select a different branch to request transfer.'
			});
		}

		// ✅ confirmation dialog
		const confirm = await Swal.fire({
			title: 'Submit Transfer Request?',
			text: 'Your request will be reviewed by the admin.',
			icon: 'question',
			showCancelButton: true,
			confirmButtonText: 'Yes, submit it',
			cancelButtonText: 'Cancel'
		});
		if (!confirm.isConfirmed) return;

		// ✅ API call
		try {
			const res = await fetch(`${url}/api/branch-transfer-request`, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${token}`,
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					user_id: userId,
					current_branch_id: currentBranchId,
					requested_branch_id: requestedBranchId,
					reason: transferReason
				})
			});

			if (!res.ok) throw new Error(await res.text());

			await Swal.fire({
				icon: 'success',
				title: 'Request Submitted',
				text: 'Your branch transfer request has been submitted.',
				timer: 2000,
				showConfirmButton: false
			});

			// reset form
			requestedBranchId = '';
			transferReason = '';
		} catch (err) {
			console.error('submitTransferRequest →', err);
			Swal.fire({
				icon: 'error',
				title: 'Submission Failed',
				text: 'Something went wrong while submitting the transfer request.'
			});
		}
	}
</script>

<Layout>
	<div slot="header">Member Profile</div>

	<div class="rounded-lg bg-white p-6 shadow-md">
		<!-- profile photo -->
		<div class="mb-6 flex flex-col items-center">
			<div class="mb-6 flex flex-col items-center">
				<img
					class="h-auto min-h-24 w-24 rounded-full border shadow bg-gray-300"
					src={profileImage || '/default-profile.png'}
					alt=""
				/>
				<input id="imageUpload" type="file" accept="image/*" class="hidden" on:change={handleImageUpload} />
				<button
					type="button"
					on:click={() => document.getElementById('imageUpload')?.click()}
					class="mt-3 rounded bg-blue-600 px-4 py-2 text-sm text-white shadow transition-all hover:bg-blue-700 md:text-base lg:text-lg"
				>
					Change Photo
				</button>
			</div>
		</div>

		<!-- general information -->
		<div
			class="mb-6 rounded-lg border-t-4 border-blue-600 bg-gray-100 p-6 text-base shadow md:text-base lg:text-lg"
		>
			<h3 class="mb-4 text-2xl font-semibold text-blue-600 lg:text-3xl">General Information</h3>
			<form class="grid gap-4 md:grid-cols-2" on:submit|preventDefault={updateProfile}>
				<!-- username / email -->
				<div class="md:col-span-1">
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium" >Username</label>
					<input
						bind:value={username}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>
				<div class="md:col-span-1">
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium" >Email</label>
					<input
						type="email"
						bind:value={email}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500" disabled
					/>
				</div>

				<!-- names -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">First Name</label>
					<input
						bind:value={firstName}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Middle Name</label>
					<input
						bind:value={middleName}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Last Name</label>
					<input
						bind:value={lastName}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<!-- contact / address -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Contact Number</label>
					<input
						bind:value={contact}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Address</label>
					<input
						bind:value={address}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>

				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Birthdate</label>
					<input
						type="date"
						bind:value={birthdate}
						max={today}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Gender</label>
					<select
						bind:value={gender}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					>
						<option value="" disabled selected>Select gender</option>
						<option>Male</option>
						<option>Female</option>
						<option>Other</option>
					</select>
				</div>
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Baptism Date</label>
					<input
						type="date"
						bind:value={baptismDate}
						max={today}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Salvation Date</label>
					<input
						type="date"
						bind:value={salvationDate}
						max={today}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-blue-500"
					/>
				</div>
				<div>
					<div>
						<!-- svelte-ignore a11y_label_has_associated_control -->
						<label class="block font-medium">Status</label>
						<input
							type="text"
							bind:value={status}
							class="w-full cursor-not-allowed rounded border bg-gray-100 p-2 text-gray-600"
							readonly
							disabled
						/>
					</div>
				</div>

				<div class="flex justify-start pt-6">
					<button
						type="submit"
						class="h-auto w-auto rounded-lg bg-blue-600 px-4 py-2 text-base text-white shadow hover:bg-blue-700 md:text-base lg:text-lg"
					>
						Update Information
					</button>
				</div>
			</form>
		</div>
		<!-- Password Change Section -->
		<div
			class="mb-6 rounded-lg border-t-4 border-rose-600 bg-gray-100 p-6 text-base shadow md:text-base lg:text-lg"
		>
			<h3 class="mb-4 text-2xl font-semibold text-rose-600 lg:text-3xl">Change Password</h3>
			<form class="space-y-4" on:submit|preventDefault={changePassword}>
				<!-- current password -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Current Password</label>
					<div class="relative">
						<input
							type="password"
							id="currentPassword"
							class="w-full rounded border p-2"
							bind:value={currentPassword}
							placeholder="Enter current password"
						/>
						<button
							type="button"
							on:click={() => togglePassword('currentPassword')}
							class="absolute inset-y-0 right-3 flex items-center text-gray-600 hover:text-gray-800"
							>👁</button
						>
					</div>
				</div>

				<!-- new password -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">New Password</label>
					<div class="relative">
						<input
							type="password"
							id="newPassword"
							class="w-full rounded border p-2"
							bind:value={newPassword}
							placeholder="Enter new password"
						/>
						<button
							type="button"
							on:click={() => togglePassword('newPassword')}
							class="absolute inset-y-0 right-3 flex items-center text-gray-600 hover:text-gray-800"
							>👁</button
						>
					</div>
					<ul class="mt-1 space-y-0.5 text-base">
						<li class={longEnough ? 'text-green-600' : 'text-red-600'}>· at least 8 characters</li>
						<li class={hasUpper ? 'text-green-600' : 'text-red-600'}>· 1 uppercase letter</li>
						<li class={hasLower ? 'text-green-600' : 'text-red-600'}>· 1 lowercase letter</li>
						<li class={hasNumber ? 'text-green-600' : 'text-red-600'}>· 1 number</li>
						<li class={hasSpecial ? 'text-green-600' : 'text-red-600'}>· 1 special character</li>
					</ul>
				</div>

				<!-- confirm password -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Confirm New Password</label>
					<div class="relative">
						<input
							type="password"
							id="confirmPassword"
							class="w-full rounded border p-2"
							bind:value={confirmPassword}
							placeholder="Confirm new password"
						/>
						<button
							type="button"
							on:click={() => togglePassword('confirmPassword')}
							class="absolute inset-y-0 right-3 flex items-center text-gray-600 hover:text-gray-800"
							>👁</button
						>
					</div>
					{#if mismatch}
						<p class="mt-1 text-xs text-red-600">Passwords don’t match</p>
					{/if}
				</div>

				<!-- submit -->
				<div class="flex justify-start pt-6">
					<button
						type="submit"
						class="h-11 w-48 rounded-lg bg-rose-600 px-6 text-white shadow hover:bg-rose-700 disabled:opacity-50"
						disabled={!currentPassword || !newPassword || !confirmPassword}
					>
						Update Password
					</button>
				</div>
			</form>
		</div>

		<!-- Branch Transfer Section -->
		<div class="mt-8 rounded-lg border-t-4 border-amber-600 bg-gray-100 p-6 shadow">
			<h3 class="mb-4 text-2xl font-semibold text-amber-600 lg:text-3xl">
				Branch Transfer Request
			</h3>

			<div class="space-y-4 text-base md:text-base lg:text-lg">
				<!-- branch selector -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Select New Branch</label>
					<select
						bind:value={requestedBranchId}
						class="w-full rounded border p-2 focus:ring-2 focus:ring-amber-500"
					>
						<option value="" disabled selected>Select a branch</option>
						{#each branchOptions as branch}
							<option value={branch.id}>{branch.name}</option>
						{/each}
					</select>
				</div>

				<!-- reason -->
				<div>
					<!-- svelte-ignore a11y_label_has_associated_control -->
					<label class="block font-medium">Reason (optional)</label>
					<textarea
						bind:value={transferReason}
						rows="3"
						class="w-full rounded border p-2 focus:ring-2 focus:ring-amber-500"
						placeholder="Explain your reason (optional)"
					></textarea>
				</div>

				<!-- button (same size / placement as others) -->
				<div class="flex justify-start pt-6">
					<button
						type="button"
						on:click={submitTransferRequest}
						class="h-auto w-auto rounded-lg bg-amber-600 px-4 py-2 text-center leading-tight text-white shadow hover:bg-amber-700"
					>
						<span>Submit Transfer<br />Request</span>
					</button>
				</div>
			</div>
		</div>
	</div></Layout
>
