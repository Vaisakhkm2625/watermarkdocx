<script>
	import JSZip from 'jszip';

	let fileInput;

	function processDocx() {
		if (fileInput.files.length === 0) {
			alert('Please select a docx file');
			return;
		}

		const file = fileInput.files[0];
		const reader = new FileReader();

		reader.onload = async function (event) {
			const arrayBuffer = event.target.result;

			const zip = new JSZip();
			await zip.loadAsync(arrayBuffer);

			// Unzip and process the content of docx (which is a zip file)
			//const docxContent = await zip.file('word/document.xml').async("string");
			console.log(zip.files);

			// Add watermark to images (logic depends on where images are stored)
			const imageFiles = Object.keys(zip.files).filter((fileName) => fileName.includes('media/'));

			for (let i = 0; i < imageFiles.length; i++) {
				const imageFile = zip.files[imageFiles[i]];
				const imageBlob = await imageFile.async('blob');

				const watermarkedImage = await addWatermarkToImage(imageBlob);
				zip.file(imageFiles[i], watermarkedImage);
			}

			// Rezip the file
			//zip.file('word/document.xml', docxContent);
			const newDocx = await zip.generateAsync({ type: 'blob' });

			// Create download link
			const downloadLink = document.getElementById('downloadLink');
			downloadLink.href = URL.createObjectURL(newDocx);
			downloadLink.download = 'watermarked.docx';
			downloadLink.style.display = 'block';
			downloadLink.innerHTML = 'Download Watermarked Docx';
		};

		reader.readAsArrayBuffer(file);
	}
</script>

<h1>Docx Watermark Tool</h1>
<!-- <input type="file" class="file"  accept=".docx" /> -->

<!-- Navbar -->
<nav class="navbar" role="navigation" aria-label="main navigation">
	<div class="navbar-brand">
		<a class="navbar-item" href="#">
			<img src="https://bulma.io/images/bulma-logo.png" alt="Bulma logo" width="112" height="28" />
		</a>

		<a
			role="button"
			class="navbar-burger"
			aria-label="menu"
			aria-expanded="false"
			data-target="navbarMenu"
		>
			<span aria-hidden="true"></span>
			<span aria-hidden="true"></span>
			<span aria-hidden="true"></span>
		</a>
	</div>

	<div id="navbarMenu" class="navbar-menu">
		<div class="navbar-start">
			<a class="navbar-item" href="#">Home</a>
			<a class="navbar-item" href="#">About</a>
			<a class="navbar-item" href="#">Services</a>
			<a class="navbar-item" href="#">Contact</a>
		</div>

		<div class="navbar-end">
			<div class="navbar-item">
				<div class="buttons">
					<a class="button is-primary"><strong>Sign up</strong></a>
					<a class="button is-light">Log in</a>
				</div>
			</div>
		</div>
	</div>
</nav>

<!-- Hero Section -->
<section class="hero is-info">
	<div class="hero-body">
		<div class="container has-text-centered">
			<h1 class="title">Welcome to Bulma</h1>
			<h2 class="subtitle">A simple and responsive layout with Bulma</h2>
		</div>
	</div>
</section>

<!-- Centered Section -->
<section class="section">
	<div class="container has-text-centered">
		<h3 class="title is-3">Centered Content</h3>
		<p>This section is centered using Bulma's <code>has-text-centered</code> utility class.</p>
	</div>
</section>

<!-- Cards Section -->
<section class="section">
	<div class="container">
		<div class="columns is-multiline">
			<!-- Card 1 -->
			<div class="column is-one-quarter">
				<div class="card">
					<div class="card-image">
						<figure class="image is-4by3">
							<img src="https://via.placeholder.com/300" alt="Placeholder image" />
						</figure>
					</div>
					<div class="card-content">
						<div class="media">
							<div class="media-content">
								<p class="title is-4">Card 1</p>
								<p class="subtitle is-6">@username1</p>
							</div>
						</div>
						<div class="content">Lorem ipsum dolor sit amet, consectetur adipiscing elit.</div>
					</div>
				</div>
			</div>

			<!-- Card 2 -->
			<div class="column is-one-quarter">
				<div class="card">
					<div class="card-image">
						<figure class="image is-4by3">
							<img src="https://via.placeholder.com/300" alt="Placeholder image" />
						</figure>
					</div>
					<div class="card-content">
						<div class="media">
							<div class="media-content">
								<p class="title is-4">Card 2</p>
								<p class="subtitle is-6">@username2</p>
							</div>
						</div>
						<div class="content">Lorem ipsum dolor sit amet, consectetur adipiscing elit.</div>
					</div>
				</div>
			</div>

			<!-- Card 3 -->
			<div class="column is-one-quarter">
				<div class="card">
					<div class="card-image">
						<figure class="image is-4by3">
							<img src="https://via.placeholder.com/300" alt="Placeholder image" />
						</figure>
					</div>
					<div class="card-content">
						<div class="media">
							<div class="media-content">
								<p class="title is-4">Card 3</p>
								<p class="subtitle is-6">@username3</p>
							</div>
						</div>
						<div class="content">Lorem ipsum dolor sit amet, consectetur adipiscing elit.</div>
					</div>
				</div>
			</div>

			<!-- Card 4 -->
			<div class="column is-one-quarter">
				<div class="card">
					<div class="card-image">
						<figure class="image is-4by3">
							<img src="https://via.placeholder.com/300" alt="Placeholder image" />
						</figure>
					</div>
					<div class="card-content">
						<div class="media">
							<div class="media-content">
								<p class="title is-4">Card 4</p>
								<p class="subtitle is-6">@username4</p>
							</div>
						</div>
						<div class="content">Lorem ipsum dolor sit amet, consectetur adipiscing elit.</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</section>

<!-- Footer -->
<footer class="footer">
	<div class="content has-text-centered">
		<p>
			<strong>Bulma</strong> by <a href="https://bulma.io">Bulma.io</a>. The source code is licensed
			MIT.
		</p>
	</div>
</footer>

<div class="file is-boxed">
	<label class="file-label">
		<input class="file-input" type="file" bind:this={fileInput} name="resume" />
		<span class="file-cta">
			<span class="file-icon">
				<i class="fas fa-upload"></i>
			</span>
			<span class="file-label"> Choose a file… </span>
		</span>
	</label>

	<button onclick="processDocx()">Add Watermark</button>
	<a id="downloadLink" style="display: none;">Download Watermarked Docx</a>
</div>
