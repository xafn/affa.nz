<script>
	import Social from '../atoms/Social.svelte';
	import { user } from '../../util/discord';

	import { GitHubIcon } from '@indaco/svelte-iconoir/github';
	import { DiscordIcon } from '@indaco/svelte-iconoir/discord';
	import { MailIcon } from '@indaco/svelte-iconoir/mail';
	import { CoffeeCupIcon } from '@indaco/svelte-iconoir/coffee-cup';
	import { LinkedInIcon } from '@indaco/svelte-iconoir/linkedin';
</script>

<div class="socials-container">
	<Social tip="xafn" link="https://github.com/xafn">
		<GitHubIcon color="var(--accent)" />
	</Social>

	<Social tip="affn" link="https://linkedin.com/in/affn">
		<LinkedInIcon color="var(--accent)" />
	</Social>

	<Social tip={user.username} link={`https://discord.com/users/${user.id}`}>
		<DiscordIcon color="var(--accent)" />
	</Social>

	<Social tip="hi@affa.nz" link="mailto:hi@affa.nz">
		<MailIcon color="var(--accent)" />
	</Social>

	<Social tip="affan" link="https://ko-fi.com/affan">
		<CoffeeCupIcon color="var(--accent)" />
	</Social>
</div>

<style>
	.socials-container {
		display: flex;
		gap: 0.75rem;
	}
</style>
