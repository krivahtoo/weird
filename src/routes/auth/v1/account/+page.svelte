<script lang="ts">
	import getPkce from 'oauth-pkce';
	import { getUserInfo } from '$lib/rauthy';
	import { onMount } from 'svelte';
	import type { PageData } from './$types';
	import { page } from '$app/stores';
	import Avatar from '$lib/components/Avatar.svelte';
	import { InputChip } from '@skeletonlabs/skeleton';

	const { data }: { data: PageData } = $props();

	const userInfo = getUserInfo();
	const PKCE_VERIFIER = 'pkce_verifier';

	let username = $state(data.profile?.username || '');
	let display_name = $state(data.profile?.display_name || '');
	let avatar_seed = $state(data.profile?.avatar_seed || data.profile?.username || '');
	let location = $state(data.profile?.location || '');
	let contact_info = $state(data.profile?.contact_info || '');
	let tags = $state(data.profile?.tags || []);
	let work_capacity = $state(data.profile?.work_capacity);
	let work_compensation = $state(data.profile?.work_compensation);
	let bio = $state(data.profile?.bio || '');

	const baseUrl = new URL($page.url);
	baseUrl.pathname = '';

	const getKey = (i: number) => {
		let res = '';

		const target = i || 8;
		for (let i = 0; i < target; i += 1) {
			let nextNumber = 60;
			while ((nextNumber > 57 && nextNumber < 65) || (nextNumber > 90 && nextNumber < 97)) {
				nextNumber = Math.floor(Math.random() * 74) + 48;
			}
			res = res.concat(String.fromCharCode(nextNumber));
		}

		return res;
	};

	onMount(() => {
		if (!userInfo) {
			getPkce(64, (error, { challenge, verifier }) => {
				if (!error) {
					localStorage.setItem(PKCE_VERIFIER, verifier);
					const nonce = getKey(24);
					const s = 'account';
					const redirect_uri = `${window.location.origin}/auth/v1/oidc/callback`
						.replaceAll(':', '%3A')
						.replaceAll('/', '%2F');
					window.location.href = `/auth/v1/oidc/authorize?client_id=rauthy&redirect_uri=${redirect_uri}&response_type=code&code_challenge=${challenge}&code_challenge_method=S256&scope=openid+profile+email&nonce=${nonce}&state=${s}`;
				}
			});
		}
	});
</script>

{#if userInfo}
	<main class="flex flex-col items-center">
		<form
			method="post"
			action="/account/update"
			class="card mt-12 flex w-[600px] max-w-[90%] flex-col gap-4 p-8 text-xl"
		>
			<div class="mb-4 flex items-center gap-3">
				<Avatar seed={avatar_seed} />
				<div>
					<h1 class="my-3 text-4xl">Edit Profile</h1>
					<div class="text-surface-300">{userInfo.email}</div>
				</div>
			</div>

			<div class="mb-4">
				<div>
					<strong class="pr-2">Public Profile:</strong>
					<span class="text-base">
						{#if data.profile?.username}
							<a class="underline" href={`${baseUrl}u/${data.profile?.username}`}>
								{`${baseUrl}u/${data.profile?.username}`}
							</a>
						{:else}<span class="text-surface-400">Username Not Set</span>{/if}
					</span>
				</div>
			</div>

			<label class="label">
				<span>Username</span>
				<input name="username" class="input" placeholder="Username" bind:value={username} />
				{#if !data.profile?.username}<div class="pl-3 text-sm">
						Set a username to claim your profile page.
					</div>{/if}
			</label>
			<label class="label">
				<span>Display Name</span>
				<input name="display_name" class="input" placeholder={username} bind:value={display_name} />
			</label>
			<label class="label">
				<span>Avatar Seed</span>
				<input
					name="avatar_seed"
					class="input"
					placeholder="I am a robot..."
					bind:value={avatar_seed}
				/>
				<div class="pl-3 text-sm">Your avatar is randomly generated from this text.</div>
			</label>
			<label class="label">
				<span>Location</span>
				<input name="location" class="input" bind:value={location} />
			</label>
			<label class="label">
				<span>Contact Info</span>
				<input
					name="contact_info"
					class="input"
					placeholder="Email, phone, etc."
					bind:value={contact_info}
				/>

				<div class="pl-3 text-sm">Contact info will only be shown to logged-in users.</div>
			</label>
			<!-- svelte-ignore a11y_label_has_associated_control -->
			<label class="label">
				<span>Tags</span>
				<InputChip bind:value={tags} name="tags" placeholder="Interest, skill, etc." />
			</label>
			<label class="label">
				<span>Work Capacity</span>
				<select class="select" name="work_capacity" bind:value={work_capacity}>
					<option value={null}>Not Specified</option>
					<option value="part_time">Part Time</option>
					<option value="full_time">Full Time</option>
				</select>
			</label>
			<label class="label">
				<span>Work Compensation</span>
				<select class="select" name="work_compensation" bind:value={work_compensation}>
					<option value={null}>Not Specified</option>
					<option value="paid">Paid</option>
					<option value="volunteer">Volunteer</option>
				</select>
			</label>
			<label class="label">
				<span>Bio</span>
				<textarea
					name="bio"
					class="textarea"
					placeholder="Tell people more about you..."
					rows="5"
					bind:value={bio}
				>
				</textarea>
			</label>

			<button class="variant-filled btn mt-4"> Save </button>
		</form>
	</main>
{/if}
