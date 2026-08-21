<script lang="ts">
	import { onMount } from 'svelte';
	import type { Repo } from '../../util/types';

	import { StarIcon } from '@indaco/svelte-iconoir/star';
	import { GitForkIcon } from '@indaco/svelte-iconoir/git-fork';
	import { OpenNewWindowIcon } from '@indaco/svelte-iconoir/open-new-window';

	type RepoCard = Repo & { link: string };

	let repos: RepoCard[] | null = null;

	onMount(async () => {
		try {
			const response = await fetch('https://pinned.berrysauce.dev/get/xafn');
			if (!response.ok) {
				repos = [];
				return;
			}

			const pinnedRepos: Repo[] = await response.json();
			repos = pinnedRepos.map((repo) => {
				const author = repo.author.replace(/\/$/, '');

				return {
					...repo,
					author,
					link: `https://github.com/${author}/${repo.name}`
				};
			});
		} catch {
			repos = [];
		}
	});
</script>

<section class="wrapper" id="code">
	<div class="title">
		<h2>oss</h2>
	</div>
	<div class="grid">
		{#if repos}
			{#each repos as { link, author, name, description, languageColor, language, stars, forks }}
				<a href={link} target="_blank" rel="noreferrer">
					<div class="repo-card">
						<div id="top-part">
							<div class="info">
								<img
									src="https://github.com/{author}.png"
									alt="{author}'s profile picture"
									id="pfp"
								/>
								<h6>{author}</h6>
							</div>
							<div id="open">
								<OpenNewWindowIcon color="var(--text-secondary)" size="20px" />
							</div>
						</div>
						<div>
							<h3>{name}</h3>
							<h6>{description}</h6>
						</div>
						<div class="info-container">
							<div class="info">
								<span class="dot" style="background-color: {languageColor}" />
								<h6>{language}</h6>
							</div>
							<div class="info">
								<StarIcon color="var(--text-secondary)" size="16px" />
								<h6>{stars}</h6>
							</div>
							<div class="info">
								<GitForkIcon color="var(--text-secondary)" size="16px" />
								<h6>{forks}</h6>
							</div>
						</div>
					</div>
				</a>
			{/each}
		{:else}
			{#each Array(4) as _}
				<div class="repo-card shimmer" />
			{/each}
		{/if}
	</div>
</section>

<style lang="scss">
	@import '../../styles/mixins.scss';

	.title {
		display: flex;
		justify-content: center;
		margin-top: 0;

		@media (max-width: 868px) {
			justify-content: left;
		}
	}
	.repo-card {
		padding: 1rem 1.25rem;
		background-color: var(--elevation-two);
		border-radius: 8px;
		min-height: 140px;
		height: 100%;
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		transition:
			transform 0.3s var(--bezier-one),
			box-shadow 0.3s var(--bezier-one);
		justify-content: space-between;
		backdrop-filter: blur(5px);
		-webkit-backdrop-filter: blur(5px);
		background-blend-mode: overlay;
		border: 1px solid var(--elevation-four);

		&:hover {
			transform: translateY(-2px);
			box-shadow: 0px 15px 25px -10px rgba(0, 0, 0, 0.25);

			#open {
				filter: brightness(1.3);
			}
		}
	}

	.shimmer {
		animation-duration: 2.2s;
		animation-fill-mode: forwards;
		animation-iteration-count: infinite;
		animation-name: shimmer;
		animation-timing-function: linear;
		background: #ddd;
		background: linear-gradient(
			to right,
			var(--elevation-two) 8%,
			var(--elevation-one) 18%,
			var(--elevation-two) 33%
		);
		background-size: 1200px 100%;
	}

	@keyframes shimmer {
		0% {
			background-position: -1200px 0;
		}
		100% {
			background-position: 1200px 0;
		}
	}

	a {
		text-decoration: none;
		color: var(--text-primary);
		height: 100%;
		border-radius: 8px;
	}

	h2 {
		display: inline-block;
		margin-bottom: 1rem;
	}

	#pfp {
		border-radius: 50%;
		height: 16px;
	}

	#top-part {
		display: flex;
		justify-content: space-between;
	}

	#open {
		height: 20px;
		transition: filter 0.3s var(--bezier-one);
	}

	span {
		color: var(--accent);
	}

	.grid {
		gap: 0.8rem;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		display: grid;
		grid-template-columns: 1fr 1fr;
		margin-bottom: 5rem;
		position: relative;

		&:before {
			@include outlineText($content: 'λ', $translateX: 1150%, $translateY: -30%);
		}

		@media (max-width: 600px) {
			grid-template-columns: 1fr;
		}

		@media (max-width: 868px) {
			margin-bottom: 2rem;
		}
	}

	.dot {
		height: 11px;
		width: 11px;
		border-radius: 50%;
		display: inline-block;
	}

	.info {
		display: flex;
		gap: 0.2rem;
		align-items: center;

		&-container {
			display: flex;
			gap: 0.9rem;
		}
	}
</style>
