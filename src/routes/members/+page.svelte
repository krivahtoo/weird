<script lang="ts">
	import Avatar from '$lib/components/Avatar.svelte';
	import type { WorkCapacity, WorkCompensation } from '../auth/v1/account/proxy+page.server';
	import type { PageData } from './$types';

	const { data }: { data: PageData } = $props();
	const profiles = data.profiles;
	let search = $state('');

	const printWorkCapacity = (c?: WorkCapacity): string => {
		if (c == 'full_time') {
			return 'Full Time';
		} else if (c == 'part_time') {
			return 'Part Time';
		} else {
			return 'Not Specified';
		}
	};
	const printWorkCompensation = (c?: WorkCompensation): string => {
		if (c == 'paid') {
			return 'Paid';
		} else if (c == 'volunteer') {
			return 'Volunteer';
		} else {
			return 'Not Specified';
		}
	};

	let filtered_profiles = $derived.by(() => {
		const words = search.split(' ');
		return profiles.filter((x) => {
			if (search == '') return true;
			for (const word of words) {
				for (const field of [
					x.bio,
					x.username,
					printWorkCapacity(x.work_capacity),
					printWorkCompensation(x.work_compensation),
					...x.tags
				]) {
					if (word && field && field.includes(word)) {
						return true;
					}
				}
			}
			return false;
		});
	});
</script>

<main class="flex max-w-full flex-col items-center">
	<h1 class="mt-8 text-4xl font-bold">Weird Ones</h1>

	<input class="input mt-8 max-w-80" placeholder="Search..." bind:value={search} />

	<div class="mt-10 flex max-w-full flex-row flex-wrap justify-center gap-5 px-5">
		{#each filtered_profiles as profile}
			<a href={`/u/${profile.username}`} class="w-120 card flex flex-col items-center p-5">
				<div class="flex w-[15em] flex-col items-center text-center">
					<div class="mb-3 flex flex-col flex-wrap items-center gap-4">
						<Avatar width="w-[5em]" seed={profile.avatar_seed || profile.username || ''} />
						<h2 class="text-2xl font-semibold">{profile.display_name || profile.username}</h2>
					</div>

					<div class="flex max-w-full flex-col gap-4">
						{#if profile.location}
							<div>
								{profile.location}
							</div>
						{/if}
						{#if profile.tags.length > 0}
							<div class="flex max-w-full flex-wrap items-center justify-center gap-2">
								{#each profile.tags as tag}<span class="rounded-md bg-surface-900 p-1">{tag}</span
									>{/each}
							</div>
						{/if}
						{#if profile.contact_info}
							<div>
								{profile.contact_info}
							</div>
						{/if}
						<div>
							{#if profile.work_capacity}
								{printWorkCapacity(profile.work_capacity)}
							{/if}
							{#if profile.work_capacity && profile.work_compensation}
								&nbsp;/&nbsp;
							{/if}
							{#if profile.work_compensation}
								{printWorkCompensation(profile.work_compensation)}
							{/if}
						</div>
					</div>
				</div>
			</a>
		{/each}
	</div>
</main>
