<script lang="ts">
	import Dots from '$lib/Dots.svelte';
	import autoAnimate from '@formkit/auto-animate';
	import { MultiSelect } from 'svelte-multiselect';

	let authed = 2;
	let page = 2;

	// Smithed stuff
	import { initializeApp } from 'firebase/app';
	import { getAuth, signInWithEmailAndPassword, getIdToken, type User } from 'firebase/auth';

	initializeApp({
		databaseURL: 'https://mc-smithed-default-rtdb.firebaseio.com',
		apiKey: 'AIzaSyDX-vLCBhO8StKAxnpvQ2EW8lz3kzYn4Qk',
		authDomain: 'mc-smithed.firebaseapp.com',
		projectId: 'mc-smithed',
		storageBucket: 'mc-smithed.appspot.com',
		messagingSenderId: '574184244682',
		appId: '1:574184244682:web:498d168c09b39e4f0d7b33',
		measurementId: 'G-40SRKC35Z0'
	});

	async function logIntoSmithed(email: string, password: string) {
		const auth = getAuth();
		let smithedToken: string | null = null;
		let smithedUser: User | null = null;

		try {
			const userCredential = await signInWithEmailAndPassword(auth, email, password);
			const user = userCredential.user;
			const tok = await getIdToken(user);
			const smithedApiTokenReq = await fetch(
				`https://api.smithed.dev/v2/token?token=${tok}&expires=1d`
			);
			const smithedApiToken = await smithedApiTokenReq.text();
			smithedToken = smithedApiToken;
			smithedUser = userCredential.user;
		} catch (error: any) {
			alert('There was an error: ' + error.message);
		}

		return { token: smithedToken, user: smithedUser };
	}

	// Auth
	let smithedEmail: string;
	let smithedPassword: string;

	let authedSmithed: null | {
		displayName: string;
		cleanName: string;
		creationTime: number;
		uid: string;
		pfp: string | undefined;
	} = null;
	let smithedPacks: {
		id: string;
		display: {
			name: string;
			description: string;
			icon: string;
			hidden: boolean;
			webPage: string | undefined;
		};
		versions: string[];
		categories: string[];
	}[] = [];

	async function authSmithed() {
		let smithed = await logIntoSmithed(smithedEmail, smithedPassword);
		if (!smithed.user) {
			return;
		}

		let smithedUserData = await fetch('https://api.smithed.dev/v2/users/' + smithed.user.uid);

		if (!smithedUserData.ok) {
			alert('Error loading your Smithed user data.');
			return;
		}

		let smithedUserParsed = await smithedUserData.json();

		let smithedUserPacks = await fetch(
			`https://api.smithed.dev/v2/users/${smithed.user.uid}/packs`
		);
		if (!smithedUserPacks.ok) {
			alert('Error fetching your Smithed packs.');
		}
		let tempPacks = smithedPacks;
		let smithedUserPacksParsed = (await smithedUserPacks.json()) as string[];

		for (const pack of smithedUserPacksParsed) {
			let smithedPack = await fetch(`https://api.smithed.dev/v2/packs/${pack}`);
			if (smithedPack.ok) {
				let smithedPackParsed = await smithedPack.json();
				console.log(smithedPackParsed);
				tempPacks.push(smithedPackParsed);
			}
		}

		smithedPacks = tempPacks;
		console.log('Smithed Packs: ' + smithedPacks);

		authedSmithed = smithedUserParsed;
		authed++;
	}

	let dphToken: string;

	let authedDph: null | {
		id: number;
		bio: string;
		profile_icon: string;
		role: string;
		username: string;
		badges: string[];
	} = null;

	let dphPacks: {
		title: string;
		id: string;
	}[] = [];

	async function authDph() {
		let dphUser = await fetch(`https://api.datapackhub.net/user/me`, {
			headers: {
				Authorization: 'Basic ' + dphToken
			}
		});

		if (dphUser.ok) {
			let dphUserParsed = await dphUser.json();
			authedDph = dphUserParsed;

			let dphProjects = await fetch(
				`https://api.datapackhub.net/user/${dphUserParsed.username}/projects`
			);
			if (dphProjects.ok) {
				let dphProjectsParsed = await dphProjects.json();
				dphPacks = dphProjectsParsed.result;
			}
			authed++;
		}
	}

	let modToken: string;

	let authedModrinth: null | {
		username: string;
		id: string;
	} = null;

	let modPacks: {
		title: string;
		id: string;
	}[] = [];

	async function authModrinth() {
		let modUser = await fetch(`https://api.modrinth.com/v2/user`, {
			headers: {
				Authorization: modToken
			}
		});
		if (modUser.ok) {
			let modUserParsed = (await modUser.json()) as { username: string; id: string };
			authedModrinth = modUserParsed;

			let modProjects = await fetch(
				`https://api.modrinth.com/v2/user/${modUserParsed.id}/projects`,
				{
					headers: {
						Authorization: modToken
					}
				}
			);
			if (modProjects.ok) {
				let modProjectsParsed = (await modProjects.json()) as { title: string; id: string }[];
				modPacks = modProjectsParsed;
			}
			authed++;
		}
	}

	const minecraftVersions = [
		'1.13',
		'1.15',
		'1.16.2',
		'1.17.x',
		'1.18',
		'1.18.2',
		'1.19',
		'1.19.4',
		'1.20',
		'1.20.2'
	];

	// MODRINTH DEPENDENCIES
	let modDeps: {
		title: string;
		id: string;
		icon_url: string;
		dep_type: 'required' | 'optional' | 'incompatible' | 'embedded';
	}[] = [];

	let addModDep = false;
	let modDepSource: 'project' | 'version' = 'project';
	let modDepType: 'required' | 'optional' | 'incompatible' | 'embedded' = 'required';
	let modDepValue: string = '';

	async function createModDep() {
		if (modDepValue == '') return alert('Make sure to set a value!');
		if (modDepSource == 'project') {
			let projReq = await fetch('https://api.modrinth.com/v2/project/' + modDepValue);
			if (!projReq.ok) {
				alert('Could not get the Modrinth project. Does it exist?');
			}

			let projReqParsed = (await projReq.json()) as {
				title: string;
				id: string;
				icon_url: string;
				dep_type: 'required' | 'optional' | 'incompatible' | 'embedded';
			};
			projReqParsed.dep_type = modDepType;
			modDeps.push(projReqParsed);
			console.log(modDeps);
			addModDep = false;
		}
	}

	// SMITHED DEPENDENCIES
	let addSmiDep = false;
	let smiDepValue: string = '';
	let smiDepVerValue: string = '';
	let smiDeps: {
		id: string;
		version: string;
		name: string;
		icon: string;
	}[] = [];

	async function createSmiDep() {
		if (smiDepValue == '' || smiDepVerValue == '')
			return alert('Make sure to set a value and version ID!');
		let projReq = await fetch('https://api.smithed.dev/v2/packs/' + smiDepValue + '/versions');
		let projDataReq = await fetch('https://api.smithed.dev/v2/packs/' + smiDepValue);
		if (!projReq.ok || !projDataReq.ok) {
			return alert('Could not get the Smithed project. Does it exist?');
		}

		let projReqParsed = (await projReq.json()) as { name: string }[];
		let projDataParsed = (await projDataReq.json()) as {
			id: string;
			display: { name: string; icon: string };
		};

		const versionExists = projReqParsed.some((versionObj) => versionObj.name === smiDepVerValue);
		if (!versionExists) return alert('Are you sure the version exists?');

		smiDeps.push({
			id: projDataParsed.id,
			version: smiDepVerValue,
			name: projDataParsed.display.name,
			icon: projDataParsed.display.icon
		});
		addSmiDep = false;
	}

	let rpLabel: HTMLLabelElement;
	function uploadRP() {
		rpLabel.innerHTML = `<span class="text-slate-200">${
			rpInput.files?.item(0)?.name
		} (click to change)</span>`;
	}

	// --------------- VERSION DETAILS ---------------
	let fileInput: HTMLInputElement;

	let title: string = '';
	let versionCode: string = '';
	let changelog: string = '';
	let mcVersions: string[] = [];
	// dependencies are separate
	let releaseChannel: 'release' | 'beta' | 'alpha' = 'release';
	let rpInput: HTMLInputElement;

	// ------------ THE RELEVANT FUNCTION ------------
	let selectedDphPack: string;

	const toBase64 = (file: Blob) =>
		new Promise<string>((resolve, reject) => {
			const reader = new FileReader();
			reader.readAsDataURL(file);
			reader.addEventListener('load', () => resolve(reader.result as string));
			reader.addEventListener('error', () => reject);
		});

	async function postFunction() {
		if (authedDph) {
			const dpFileInput = document.getElementById('upload') as HTMLInputElement;
			const dpFile = dpFileInput.files?.item(0);
			if (!dpFile)
				return alert(
					"Something went wrong. Please re-upload your datapack file, I can't seem to find it."
				);
		}
	}
</script>

<div class="flex justify-around items-center min-h-screen h-full py-3">
	<div class="flex flex-col items-center space-y-10">
		<div class="flex flex-col items-center space-y-2">
			<div class="flex items-center">
				<img src="/mailman.png" class="h-20 mr-2" alt="logo" />
				<h1>
					<b class="text-5xl md:text-7xl">Mailman</b>
					<span class="text-base hidden md:inline">by Datapack Hub</span>
				</h1>
			</div>
			<p>
				Upload your datapack versions to
				<a class="text-orange-600 hover:underline" href="https://datapackhub.net" target="_blank"
					>Datapack Hub</a
				>,
				<a class="text-green-600 hover:underline" href="https://modrinth.com" target="_blank"
					>Modrinth</a
				>, and
				<a class="text-blue-500 hover:underline" href="https://beta.smithed.dev" target="_blank"
					>Smithed</a
				> at the same time!
			</p>
		</div>
		<div>
			<div
				class="bg-zinc-950 rounded-xl border-zinc-800 border-2 p-3 max-h-[50%] overflow-y-auto styled-scrollbar max-w-screen-lg"
				use:autoAnimate
			>
				<input name="upload" id="upload" type="file" class="hidden" on:change={() => (page = 1)} />
				{#if page == 0}
					<label for="upload" class="cursor-pointer">
						<p class="p-5 px-10 hover:px-12 transition-all"><b>Click</b> to upload a datapack.</p>
					</label>
				{:else if page == 1}
					<p class="font-light text-sm mb-2">
						<b class="font-bold text-base">Authenticate and choose your projects. </b>If you leave
						one blank, the version won't be posted to that site.
					</p>
					<div
						class="flex flex-col md:flex-row justify-around space-y-2 md:space-y-0 md:space-x-2 w-full"
					>
						<!-- DATAPACK HUB -->
						<div class="bg-zinc-950 w-full rounded-xl border-zinc-800 border-2 p-3 space-y-2">
							<div class="flex items-center space-x-2">
								<img src="/dph.png" class="h-8" alt="logo" />
								<b>Datapack Hub</b>
							</div>
							{#if !authedDph}
								<input
									class="focus:outline-orange-600"
									placeholder="datapackhub.net API token"
									bind:value={dphToken}
								/>
								<button
									class="items-center flex w-fit bg-zinc-900 rounded-md h-8 p-2 px-2 cursor-pointer text-zinc-400 hover:text-zinc-300 mb-1"
									on:click={authDph}>Authenticate</button
								>
							{:else}
								<p><b>User:</b> {authedDph?.username}</p>
								<select class="focus:outline-blue-500" bind:value={selectedDphPack}>
									<option>-- Select a pack --</option>
									{#each dphPacks as pack}
										<option value={pack.id}>{pack.title}</option>
									{/each}
								</select>
							{/if}
						</div>

						<!-- MODRINTH -->
						<div class="bg-zinc-950 w-full rounded-xl border-zinc-800 border-2 p-3 space-y-2">
							<div class="flex items-center space-x-2">
								<img src="/modrinth.svg" class="h-8" alt="logo" />
								<b>Modrinth</b>
							</div>
							{#if !authedModrinth}
								<input
									class="focus:outline-green-600"
									placeholder="modrinth.com API token"
									bind:value={modToken}
								/>
								<button
									class="items-center flex w-fit bg-zinc-900 rounded-md h-8 p-2 px-2 cursor-pointer text-zinc-400 hover:text-zinc-300 mb-1"
									on:click={authModrinth}>Authenticate</button
								>
							{:else}
								<p><b>User:</b> {authedModrinth.username}</p>
								<select class="focus:outline-blue-500">
									<option>-- Select a pack --</option>
									{#each modPacks as pack}
										<option>{pack.title}</option>
									{/each}
								</select>
							{/if}
						</div>

						<!-- SMITHED -->
						<div class="bg-zinc-950 w-full rounded-xl border-zinc-800 border-2 p-3 space-y-2">
							<div class="flex items-center space-x-2">
								<img src="/smithed.svg" class="h-8 rounded-full bg-[#1b48c4]" alt="logo" />
								<b>Smithed</b>
							</div>
							{#if !authedSmithed}
								<input
									class="focus:outline-blue-500"
									placeholder="smithed.dev email"
									bind:value={smithedEmail}
								/>
								<input
									type="password"
									class="focus:outline-blue-500"
									placeholder="smithed.dev password"
									bind:value={smithedPassword}
								/>
								<button
									class="items-center flex w-fit bg-zinc-900 rounded-md h-8 p-2 px-2 cursor-pointer text-zinc-400 hover:text-zinc-300 mb-1"
									on:click={authSmithed}>Log In</button
								>
							{:else}
								<p><b>User:</b> {authedSmithed.displayName}</p>
								<select class="focus:outline-blue-500">
									<option>-- Select a pack --</option>
									{#each smithedPacks as pack}
										<option>{pack.display.name}</option>
									{/each}
								</select>
							{/if}
						</div>
					</div>
					<p class="text-sm text-zinc-500 mt-2">
						Nothing entered here is sent to us. All code is client-side, which means that we can't
						view anything you enter here, including API tokens.
					</p>
				{:else if page == 2}
					<h1 class="font-bold">Enter version details.</h1>

					<h2 class="text-sm mt-3">Title <Dots dots={['mod', 'dph']} /></h2>
					<input placeholder="The Snails Update" bind:value={title} />

					<h2 class="text-sm mt-3">Version Code <Dots dots={['mod', 'smi', 'dph']} /></h2>
					<input placeholder="v1.2.3" bind:value={versionCode} />

					<h2 class="text-sm mt-3">Changelog <Dots dots={['mod', 'dph']} /></h2>
					<textarea
						placeholder="This version adds a snail you can worship, among other bug fixes"
						bind:value={changelog}
					/>

					<h2 class="text-sm mt-3">
						Supported Minecraft Versions <Dots dots={['mod', 'dph', 'smi']} />
					</h2>
					<MultiSelect
						options={minecraftVersions}
						placeholder="1.17, 1.18"
						bind:value={mcVersions}
					/>

					<div class="flex">
						<div class="w-1/3">
							<h2 class="text-sm mt-3">Modrinth Dependencies <Dots dots={['mod']} /></h2>
							{#if addModDep == false}
								<button
									class="bg-zinc-900 rounded-md h-8 mb-2 p-1 px-2 text-zinc-400 hover:text-zinc-300"
									on:click={() => (addModDep = true)}>Add Dependency</button
								>
								{#key modDeps}
									{#each modDeps as dep}
										<div class="flex items-center space-x-2 mb-1">
											<img src={dep.icon_url} alt="Icon" class="h-8 rounded-md" />
											<h1>{dep.title}</h1>
											{#if dep.dep_type == 'required'}
												<h1 class="font-light text-sm text-green-500">• Required</h1>
											{:else if dep.dep_type == 'optional'}
												<h1 class="font-light text-sm text-yellow-500">• Optional</h1>
											{:else if dep.dep_type == 'incompatible'}
												<h1 class="font-light text-sm text-red-500">• Incompatible</h1>
											{:else if dep.dep_type == 'embedded'}
												<h1 class="font-light text-sm text-pink-500">• Embedded</h1>
											{/if}
										</div>
									{/each}
								{/key}
							{:else}
								<div class="space-y-1 w-2/3">
									<select bind:value={modDepSource}>
										<option value="project">Project</option>
										<option value="version">Version</option>
									</select>
									<input
										placeholder={modDepSource == 'project' ? 'Project ID or slug' : 'Version ID'}
										bind:value={modDepValue}
									/>
									<select bind:value={modDepType}>
										<option value="required">Required Dependency</option>
										<option value="optional">Optional Dependency</option>
										<option value="incompatible">Incompatible</option>
										<option value="embedded">Embedded</option>
									</select>
									<div class="flex space-x-1">
										<button
											class="bg-orange-600 rounded-md p-1 px-2 text-md"
											on:click={createModDep}>Add</button
										>
										<button
											class="bg-zinc-800 rounded-md p-1 px-2 text-md"
											on:click={() => (addModDep = false)}>Cancel</button
										>
									</div>
								</div>
							{/if}
						</div>
						<div class="w-1/3">
							<h2 class="text-sm mt-3">Smithed Dependencies <Dots dots={['smi']} /></h2>
							{#each [] ?? [] as dep}
								<h1>{dep}</h1>
							{/each}
							{#if addSmiDep == false}
								<button
									class="bg-zinc-900 rounded-md h-8 mb-2 p-1 px-2 text-zinc-400 hover:text-zinc-300"
									on:click={() => (addSmiDep = true)}>Add Dependency</button
								>
								{#key smiDeps}
									{#each smiDeps as dep}
										<div class="flex items-center space-x-2 mb-1">
											<img src={dep.icon} alt="Dependency icon" class="h-8 rounded-md" />
											<h1>{dep.name}</h1>
											<h1 class="font-light text-sm">(v{dep.version})</h1>
										</div>
									{/each}
								{/key}
							{:else}
								<div class="space-y-1 w-2/3">
									<input placeholder="Project ID" bind:value={smiDepValue} />
									<input placeholder="Version code" bind:value={smiDepVerValue} />
									<div class="flex space-x-1">
										<button
											class="bg-orange-600 rounded-md p-1 px-2 text-md"
											on:click={createSmiDep}>Add</button
										>
										<button
											class="bg-zinc-800 rounded-md p-1 px-2 text-md"
											on:click={() => (addSmiDep = false)}>Cancel</button
										>
									</div>
								</div>
							{/if}
						</div>
					</div>

					<h2 class="text-sm mt-3">Release Channel <Dots dots={['mod']} /></h2>
					<select bind:value={releaseChannel}>
						<option value="release">Stable Release</option>
						<option value="beta">Beta</option>
						<option value="alpha">Alpha</option>
					</select>

					<h2 class="text-sm mt-3">Resource Pack <Dots dots={['mod', 'smi', 'dph']} /></h2>
					<input
						type="file"
						class="hidden"
						name="rp"
						id="rp"
						bind:this={rpInput}
						on:change={uploadRP}
					/>
					<label
						for="rp"
						bind:this={rpLabel}
						class="items-center flex w-fit bg-zinc-900 rounded-md h-8 p-2 px-2 cursor-pointer text-zinc-400 hover:text-zinc-300 mb-1"
						>Add Resource Pack file</label
					>

					<p class="text-sm text-zinc-500 mt-2">
						Nothing entered here is sent to us. All code is client-side, which means that we can't
						view anything you enter here, including API tokens.
					</p>
				{:else if page == 3}
					<p class="font-bold text-md mb-2">Preview your new version:</p>
					<div class="p-3 bg-zinc-900 rounded-xl max-w-full">
						<p class="text-zinc-400"><b class="text-zinc-200">Title: </b> {title}</p>
						<p class="text-zinc-400"><b class="text-zinc-200">Version Code: </b> {versionCode}</p>
						{#if rpInput.files && rpInput.files?.length > 0}
							<p class="text-zinc-400">
								<b class="text-zinc-200">Resource Pack: </b>
								{rpInput.files[0].name} (Required)
							</p>
						{/if}
						<p class="text-zinc-400 mb-3">
							<b class="text-zinc-200">Supports Minecraft Versions: </b>
							{mcVersions.join(', ')}
						</p>

						<p class="text-zinc-400 mb-3"><b class="text-zinc-200">Changelog: </b> {changelog}</p>

						{#if modDeps.length > 0}
							<p class="text-zinc-400">
								<b class="text-zinc-200">Modrinth Dependencies: </b>
								{modDeps.map((obj) => obj.title).join(', ')}
							</p>
						{/if}
						{#if smiDeps.length > 0}
							<p class="text-zinc-400 mb-3">
								<b class="text-zinc-200">Smithed Dependencies: </b>
								{smiDeps.map((obj) => obj.name).join(', ')}
							</p>
						{/if}

						<p class="text-zinc-400">
							<b class="text-zinc-200">Release Channel: </b>
							{releaseChannel}
						</p>
					</div>
					<p class="text-sm text-zinc-500 mt-2">
						Once you're sure that the information above is correct, press Upload Version to upload
						your datpack to the specified websites. (Some details, such as Release Channel and
						Dependencies, are only on some websites. For example, Release Channel only applies to
						Modrinth.)
					</p>
				{/if}
			</div>
			{#if page != 0}
				<div class="mt-1 flex">
					<div class="w-1/3">
						<button
							class="bg-zinc-700 rounded-md p-1 px-2 text-lg"
							on:click={() => {
								if (page > 0) page--;
								if (page == 0) fileInput.files = null;
							}}>Back</button
						>
					</div>
					<div class="w-1/3 flex items-center justify-around text-blue-400">
						<div class="flex space-x-1">
							<a href="https://discord.datapackhub.net">Need help?</a>
						</div>
					</div>
					<div class="w-1/3">
						<button
							class="transition-all {authed > 1
								? 'bg-orange-600'
								: 'bg-orange-600/40 cursor-default'} rounded-md p-1 px-2 float-right text-lg font-semibold"
							on:click={() => {
								if (authed > 1) page++;
							}}>{page == 3 ? 'Upload Version' : 'Next'}</button
						>
					</div>
				</div>
			{/if}
		</div>
	</div>
</div>

<style>
	:root {
		--sms-bg: theme(colors.zinc.900);
		--sms-border: 0px solid theme(colors.zinc.200);
		--sms-selected-bg: theme(colors.zinc.700);
		--sms-remove-btn-hover-bg: theme(colors.orange.500);
		--sms-options-bg: theme(colors.zinc.800);
		--sms-text-color: theme(colors.slate.100);
		--sms-min-height: height: 3rem;
		--sms-border-radius: 0.375rem;
		--sms-placeholder-color: theme(colors.zinc.600);
	}
</style>
