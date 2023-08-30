<script lang="ts">
	import PDFViewer from '$lib/components/PDFViewer/PDFViewer.svelte';

	let url =
		'https://raw.githubusercontent.com/huyubing/books-pdf/4628830db115e552b08763bd4fa70398233efa64/slime.pdf';

	let nextPage: () => void;
	let prevPage: () => void;
	let gotoPage: (nbr: number) => void;
	let pageNumber: number;
	let pagesCount: number;

	let seekPage: number;
</script>

<div>
	<PDFViewer bind:nextPage bind:prevPage bind:gotoPage bind:pageNumber bind:pagesCount {url}>
		<div slot="top-actions">
			<button on:click={prevPage}>Prev</button>
			<span>{pageNumber} / {pagesCount}</span>
			<button on:click={nextPage}>Next</button>
			&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
			<input type="number" bind:value={seekPage} />
			<button on:click={() => gotoPage(seekPage)}>go</button>
		</div>

		<div slot="bottom-actions">
			<button on:click={prevPage}>Prev</button>
			<span>{pageNumber} / {pagesCount}</span>
			<button on:click={nextPage}>Next</button>
			&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
			<input type="number" bind:value={seekPage} />
			<button on:click={() => gotoPage(seekPage)}>go</button>
		</div>
	</PDFViewer>
</div>
