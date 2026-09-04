<script lang="ts">
	import Nav from '../atoms/Nav.svelte';
	let y: number;
</script>

<nav>
	<div class:scrolled={y > 20}>
		<ul>
			<Nav href="#home" section="/" isSelected={y < 350}>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="24"
					height="24"
					fill="var(--accent)"
					viewBox="0 0 24 24"
					><path fill="none" d="M0 0h24v24H0z" /><path
						d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"
					/></svg
				>
			</Nav>
			<Nav href="#projects" section="work" isSelected={y > 350 && y < 675}>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					height="24px"
					viewBox="0 -960 960 960"
					width="24px"
					fill="var(--accent)"
					><path
						d="M440-91 160-252q-19-11-29.5-29T120-321v-318q0-22 10.5-40t29.5-29l280-161q19-11 40-11t40 11l280 161q19 11 29.5 29t10.5 40v318q0 22-10.5 40T800-252L520-91q-19 11-40 11t-40-11Zm0-366v274l40 23 40-23v-274l240-139v-42l-43-25-237 137-237-137-43 25v42l240 139Z"
					/></svg
				>
			</Nav>
			<Nav href="#aw" section="museum" isSelected={y > 675}>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					height="24px"
					viewBox="0 -960 960 960"
					width="24px"
					fill="var(--accent)"
					><path
						d="M160-80q-33 0-56.5-23.5T80-160v-480q0-33 23.5-56.5T160-720h160l160-160 160 160h160q33 0 56.5 23.5T880-640v480q0 33-23.5 56.5T800-80H160Zm80-160h480L570-440 450-280l-90-120-120 160Zm502.5-217.5Q760-475 760-500t-17.5-42.5Q725-560 700-560t-42.5 17.5Q640-525 640-500t17.5 42.5Q675-440 700-440t42.5-17.5ZM404-720h152l-76-76-76 76Z"
					/></svg
				>
			</Nav>
		</ul>
	</div>
</nav>

<!-- this is the coolest thing ever wtf?? -->
<svelte:window bind:scrollY={y} />

<style lang="scss">
	nav {
		display: flex;
		align-items: center;
		justify-content: center;
	}
	div {
		width: 100%;
		text-align: center;
		align-items: center;
		position: fixed;
		top: 0;
		padding: 1.25rem 0;
		z-index: 15;
		background-color: var(--bg-color);
		backdrop-filter: blur(15px);
		-webkit-backdrop-filter: blur(15px);
		background-blend-mode: overlay;
		transition: all 0.5s ease;

		ul {
			display: flex;
			gap: 4rem;
			justify-content: center;
		}
	}
	@media (min-width: 868px) {
		div {
			width: 40rem;
			border-radius: 12px;
		}
		.scrolled {
			border-bottom: 1.5px solid var(--accent-opacity);
			padding: 0.5rem 0;
			margin-top: 0.5rem;
			width: 34rem;
			background-color: var(--elevation-five);
			box-shadow: 0px 15px 8px -10px rgba(0, 0, 0, 0.4);
		}
	}

	@media (max-width: 868px) {
		div {
			bottom: 0;
			top: auto;
			padding: 0rem;
			background-color: var(--elevation-one);
		}

		div ul {
			display: flex;
			justify-content: space-evenly;
			gap: 2vw;
		}
	}
</style>
