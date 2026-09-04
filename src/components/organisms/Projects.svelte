<script lang="ts">
	type Project = {
		name: string;
		url: string;
		description: string;
		label?: string;
		image?: string;
		videoSrc?: string;
		videoTitle?: string;
	};

	const projects: Project[] = [
		{
			name: 'Tempo (YC S23)',
			url: 'https://tempo.new',
			description:
				'Designed and implemented the Tempo landing page myself, worked on product features for the Electron app, and helped solidify the Tempo design system.',
			videoSrc:
				'https://player.vimeo.com/video/1223916040?title=0&byline=0&portrait=0&badge=0&autopause=0&player_id=0&app_id=58479&background=1&autoplay=1&loop=1&muted=1&controls=0&dnt=1',
			videoTitle: 'Tempo (YC S23)',
			label: 'Design Engineer S26'
		},
		{
			name: 'Amperon',
			url: 'https://devpost.com/software/circuito',
			description:
				'Built a web app that lets users prompt and generate validated circuits, collaborate with teams, create BOMs, and shop for components. Made it into the top 10% of Y Combinator applicants.',
			label: 'Won $1000 and 1st at NexHacks @ CMU',
			image: '/default.webp',
			videoSrc:
				'https://player.vimeo.com/video/1176038363?background=1&autoplay=1&loop=1&muted=1&autopause=0&controls=0&dnt=1',
			videoTitle: 'Amperon'
		},
		{
			name: 'Communico',
			url: 'https://devpost.com/software/communico-i5opvc',
			description:
				'AI-powered communication app for non-verbal individuals. Includes emotion in voices powered by ElevenLabs and facial recognition, draw-to-speak, and live transcription for easy mental processing.',
			label: 'Won Best Accessibility @ DeltaHacks',
			image: '/default.webp',
			videoSrc:
				'https://player.vimeo.com/video/1176039536?background=1&autoplay=1&loop=1&muted=1&autopause=0&controls=0&dnt=1',
			videoTitle: 'Communico'
		},
		{
			name: 'Instafn',
			url: 'https://github.com/xafn/instafn',
			description:
				'Browser extension that enhances the Instagram website. Allows users to confirm actions, hide story views, intercepted websockets to log messages, and much more.',
			image: '/default.webp',
			label: '1000+ users',
			videoSrc:
				'https://player.vimeo.com/video/1176040133?background=1&autoplay=1&loop=1&muted=1&autopause=0&controls=0&dnt=1',
			videoTitle: 'Instafn'
		},
	];
</script>

<section class="wrapper" id="projects">
	<div class="title">
		<h2>work</h2>
	</div>

	<div class="grid">
		{#each projects as project}
		<article class="project-card">
			<a href={project.url} target="_blank" rel="noreferrer" class="project-link">
			<div class="preview shimmer">
						{#if project.videoSrc}
							<iframe
								src={project.videoSrc}
								title={project.videoTitle ?? `${project.name} preview`}
								allow="autoplay; fullscreen; picture-in-picture; clipboard-write; encrypted-media; web-share"
								referrerpolicy="strict-origin-when-cross-origin"
								loading="lazy"
								class="preview-video"
								tabindex="-1"
								aria-hidden="true"
							/>
						{:else if project.image}
							<img src={project.image} alt="{project.name} preview" loading="lazy" />
						{/if}
						<div class="preview-mask" />
					</div>

					<div class="project-head">
						<h3>{project.name}</h3>
						{#if project.label}<h4>{project.label}</h4>{/if}
					</div>
				</a>
					<p>{project.description}</p>
				</article>
		{/each}
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

	h2 {
		display: inline-block;
		margin-bottom: 1.5rem;
		position: relative;

		&::before {
			@include outlineText($content: '***', $translateX: -220%, $translateY: 0%, $fontSize: 150px);
		}
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: 3rem;
		position: relative;
		margin-bottom: 3rem;

		&::after {
			@include outlineText($content: '##', $translateX: 405%, $translateY: 100%, $fontSize: 200px);
		}

		@media (max-width: 700px) {
			grid-template-columns: 1fr;
		}
	}

	.project-link {
		display: block;
		text-decoration: none;
	}

	.project-card {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		height: 100%;
		background-blend-mode: overlay;
		transition: transform 0.3s var(--bezier-one);

		&:hover .preview {
			transform: translateY(-2px);
		}
	}

	.project-head {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.preview {
		position: relative;
		aspect-ratio: 16 / 9;
		border-radius: 4px;
		overflow: hidden;
		background-color: var(--elevation-two);
		transition: transform 0.3s var(--bezier-one);
		margin-bottom: 1rem;

		img {
			width: 100%;
			height: 100%;
			object-fit: cover;
			filter: saturate(0.85) contrast(1.05);
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

	h4 {
		background-color: var(--elevation-one);
		border-radius: 8px;
		font-size: 0.8rem;
		color: var(--accent);
		padding: 0.3rem 0.75rem;
		font-family: var(--font-two);
	}
	@keyframes shimmer {
		0% {
			background-position: -1200px 0;
		}
		100% {
			background-position: 1200px 0;
		}
	}

	.preview-video {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		display: block;
		border: 0;
		pointer-events: none;
		user-select: none;
		filter: saturate(0.9) contrast(1.05);
	}

	.preview-mask {
		position: absolute;
		inset: 0;
	}

	h3 {
		font-size: 1.5rem;
		font-weight: 400;
		line-height: 1;
	}

	p {
		font-size: 0.9rem;
		line-height: 1.6;
		color: var(--text-secondary);
		letter-spacing: -0.01rem;
	}

	@media (max-width: 868px) {
		.grid {
			margin-bottom: 2rem;
		}

		h3 {
			font-size: 1.25rem;
		}
		h4 {
			font-size: 0.7rem;
		}

		p {
			font-size: 0.9rem;
		}
	}
</style>
