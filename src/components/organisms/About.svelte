<script>
	import RichPresence from '../molecules/RichPresence.svelte';
	import Tooltip from '../atoms/Tooltip.svelte';

	// i didnt write this idk
	let getAge = () => {
		let birthDate = new Date('2007/03/24');
		const ageMs = Date.now() - birthDate.getTime();
		const preciseAge = (ageMs / 31536000000).toFixed(10);
		return preciseAge;
	};

	let age = getAge();
	setInterval(() => {
		age = getAge();
	}, 1000);
</script>

<section id="about" class="wrapper">
	<div>
		<RichPresence />
	</div>
	<div class="text">
		<h2>bio</h2>
		<p>
			I'm <Tooltip tip="aka afn online"><span>Affan Zaman</span></Tooltip>, a <Tooltip tip={age}
				><span>{Math.floor(Number(age))}</span></Tooltip
			> year old studying CS at the <Tooltip tip="i am balding here"
				><a href="https://cs.uwaterloo.ca/" target="_blank"><span>University of Waterloo</span></a
				></Tooltip
			>. I previously worked at <Tooltip tip="i made the site btw"
				><a href="https://tempo.new" target="_blank"><span>Tempo (YC S23)</span></a></Tooltip
			> as a design engineer. While my background is now in software, I have a strong love for product, <Tooltip
				tip="like this!"><span>interaction</span></Tooltip
			>, and design. I like to contribute to
			<Tooltip tip="nerd"
				><a href="https://github.com/xafn" target="_blank"><span>open source</span></a></Tooltip
			> in my free time, and most notably worked frontend for <Tooltip tip="might be DMCA'd"
				><a href="https://revanced.app" target="_blank"><span>ReVanced</span></a></Tooltip
			>.
		</p> 
	</div>
</section>

<style lang="scss">
	@import '../../styles/mixins.scss';

	section {
		margin-bottom: 4rem;
		display: grid;
		gap: 4.5rem;
		grid-template-columns: 1fr 1fr;
		align-items: center;
	}

	.text {
		position: relative;
		line-height: 1.75rem;
	}

	span {
		font-weight: 400;
		font-family: var(--font-two);
		font-size: 0.9rem;
		background-color: var(--elevation-one);
		border-radius: 7px;
		color: var(--accent);
		padding: 0.2rem 0.5rem 0.2rem;
		width: fit-content;
	}

	a {
		text-decoration: none;
	}

	.text::before {
		@include outlineText(
			$content: 'afn',
			$translateX: 90%,
			$translateY: -5%,
			$fontSize: 300px,
			$opacity: 0.22
		);
	}

	h2 {
		display: none;
		margin-top: 1rem;
	}

	@media (max-width: 868px) {
		section {
			display: flex;
			flex-direction: column;
			align-items: normal;
		}

		h2 {
			display: block;
			margin-bottom: 1rem;
		}
	}
</style>
