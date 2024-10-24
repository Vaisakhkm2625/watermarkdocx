<script>
	export let title = 'Docx Image Batch Editor';
</script>

<nav class="navbar is-primary" role="navigation" aria-label="main navigation">
	<div class="navbar-brand">
		<a class="navbar-item" href="/">
			<strong>{title}</strong>
		</a>
	</div>

	<div class="navbar-menu">
		<div class="navbar-start">
			<a class="navbar-item" href="https://linuxwith.in">Home</a>
			<a class="navbar-item" href="https://me.linuxwith.in">About</a>
		</div>
		<div class="navbar-end">
			<div class="navbar-item">
				<div class="buttons">
					<a class="button is-light" href="https://github.com/Vaisakhkm2625/watermarkdocx">
						Source Code
					</a>
				</div>
			</div>
		</div>
	</div>
</nav>

<slot />

<footer class="footer">
	<div class="content has-text-centered">
		<p>
			<strong>Docx Image Batch Editor</strong> by <a href="https://me.linuxwith.in">Vaisakh K M</a>.
			Built with
			<a href="https://bulma.io/">Bulma</a>, JSDoc and Svelte.
		</p>
	</div>
</footer>
