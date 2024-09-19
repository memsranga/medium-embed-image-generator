<script>
	import { onMount } from 'svelte';
	import moment from 'moment';
	import html2canvas from 'html2canvas';
	let url = '';
	let author = '';
	let title = '';
	let description = '';
	let date = '';
	let articleFetched = false;
	let errorMessage = '';
	let imageUrl = '';
	let dateDisplayString = '';

	// Function to fetch article data
	const fetchArticleData = async () => {
		if (!url.startsWith('https://medium.com/')) {
			errorMessage = 'Please enter a valid Medium URL.';
			return;
		}

		errorMessage = '';
		articleFetched = false;

		try {
			// Fetching the article content (Medium's content is behind CORS, we might need a proxy in real-world cases)
			const res = await fetch(`https://api.allorigins.win/get?url=${encodeURIComponent(url)}`);
			const data = await res.json();
			const parser = new DOMParser();
			const doc = parser.parseFromString(data.contents, 'text/html');

			// Extract data from the fetched content
			// og:title
			title = doc.querySelector("meta[property='og:title']").getAttribute('content') || 'Untitled';
			// author
			// article:author
			author = doc.querySelector("meta[name='author']").getAttribute('content') || 'Untitled';
			// og:description
			description =
				doc.querySelector("meta[property='og:description']").getAttribute('content') || 'Untitled';
			date = doc.querySelector("meta[property='article:published_time']").getAttribute('content');

			dateDisplayString = moment(date).format('MMMM Do, YYYY');

			// og:url
			// og:image
			imageUrl = doc.querySelector("meta[property='og:image']").getAttribute('content');

			// twitter:label1: twitter:data1
			// Reading time: 7 mins

			// article:published_time
			// 2024-09-14T07:11:53.226Z

			// profile image regex
			// const regex = /"imageId"\s*:\s*"([^"]+)"/;

			// // Match the regex against the string
			// const dataString = JSON.stringify(data);
			// const match = dataString.match(regex);

			// if (match && match[1]) {
			// 	const value = match[1];
			// 	console.log(value); // This will print the extracted value
			// } else {
			// 	console.log('No match found');
			// }

			articleFetched = true;
		} catch (error) {
			console.error(error);
			errorMessage = 'Failed to fetch article data.';
		}
	};

	// Function to convert the div to an image and download it
	const convertDivToImage = async () => {
		const captureDiv = document.getElementById('captureDiv');

		const canvas = await html2canvas(captureDiv, {
			useCORS: true,
			imageTimeout: 5000 // Optional timeout for large images
		});
		const link = document.createElement('a');
		link.href = canvas.toDataURL('image/png');
		link.download = 'medium-article.png';
		link.click();
	};
</script>

<div>
	<h1>Medium to Image Converter</h1>

	<input type="text" placeholder="Enter Medium article URL" bind:value={url} />
	<button on:click={fetchArticleData}>Fetch Article</button>

	{#if errorMessage}
		<p style="color: red;">{errorMessage}</p>
	{/if}

	{#if articleFetched}
		<div id="captureDiv" class="post-container">
			<div class="post-content">
				<div class="author-name">{author}</div>
				<div class="post-title">{title}</div>
				<div class="post-description">{description}</div>
				{#if dateDisplayString}
					<div class="post-time">{dateDisplayString}</div>
				{/if}
			</div>
			{#if imageUrl}
				<img
					id="articl-image"
					class="image-container"
					src={imageUrl}
					alt={description}
					crossorigin="anonymous"
				/>
			{/if}
		</div>

		<button on:click={convertDivToImage}>Download as Image</button>
	{/if}
</div>

<style>
	.post-container {
		border: 1px solid #e0e0e0;
		border-radius: 8px;
		padding: 16px;
		display: flex;
		align-items: center;
		max-width: 600px;
		margin: 20px auto;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
	}
	.profile-picture {
		width: 40px;
		height: 40px;
		border-radius: 50%;
		background-color: #f0f0f0;
		margin-right: 12px;
	}
	.post-content {
		flex-grow: 1;
	}
	.author-name {
		font-family: sohne, 'Helvetica Neue', Helvetica, Arial, sans-serif;
		font-size: 13px;
		font-weight: 400;
		margin-bottom: 16px;
		color: #242424;
	}
	.post-title {
		font-family: sohne, 'Helvetica Neue', Helvetica, Arial, sans-serif;
		font-size: 24px;
		font-weight: 700;
		margin: 0 0 8px 0;
	}
	.post-description {
		font-family: sohne, 'Helvetica Neue', Helvetica, Arial, sans-serif;
		color: #555;
		font-size: 16px;
		font-weight: 400;
		color: #6b6b6b;
	}
	.post-time {
		font-family: sohne, 'Helvetica Neue', Helvetica, Arial, sans-serif;
		color: #888;
		font-size: 12px;
		margin-top: 16px;
	}
	.image-container {
		width: 160px;
		height: 107px;
		margin-left: 16px;
		border-radius: 2px;
		background-color: lightgray;
	}
</style>
