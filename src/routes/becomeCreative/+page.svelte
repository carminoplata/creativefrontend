<script>
    let files;

	function onSubmitClick() {

		if (files) {
			const formData = new FormData();
			console.log(files);
			console.log(files[0]);
			formData.append('resume', files[0]);
			console.log(formData);
			const upload = fetch('https://carminoplata.eu.pythonanywhere.com/extractInfo', {
				method: 'POST',
				crossorigin: true,
				mode: 'no-cors',
				body: formData
			}).then((response) => response.json()).then((result) => {
            	console.log('Success:', result);
        	}).catch((error) => {
				console.error('Error:', error);
			});
		}
		
	}
</script>

<svelte:head>
	<title>Become a Creative</title>
</svelte:head>

<div class="text-column">
	<h1 class="page-title">Let the world know about you</h1>

	<label for="many">Upload your resume</label>
	<input
		type="file"
		bind:files={files}
	/>

	<button on:click={onSubmitClick} aria-label="Increase the counter by one">
		Submit
	</button>
</div>


