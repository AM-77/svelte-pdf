<script lang="ts">
	import * as pdfjs from 'pdfjs-dist';
	import workerURL from '../../utils/pdf.worker.min.txt';

	export let url: string;

	export let pageNumber: number = 1;
	export let pagesCount: number = 0;
	let scale: number = 5;

	let pdfDoc: pdfjs.PDFDocumentProxy;
	let pageRendering: boolean = false;
	let pageNumberPending: number | null = null;
	let ctx: CanvasRenderingContext2D;
	let canvas: HTMLCanvasElement;

	const renderPage = (num: number) => {
		pageRendering = true;
		// Using promise to fetch the page
		pdfDoc.getPage(num).then((page: pdfjs.PDFPageProxy) => {
			var viewport = page.getViewport({ scale: scale });
			canvas.height = viewport.height;
			canvas.width = viewport.width;

			// Render PDF page into canvas context
			var renderContext = {
				canvasContext: ctx,
				viewport: viewport
			};
			var renderTask = page.render(renderContext);

			// Wait for rendering to finish
			renderTask.promise.then(() => {
				pageRendering = false;
				if (pageNumberPending !== null) {
					// New page rendering is pending
					renderPage(pageNumberPending);
					pageNumberPending = null;
				}
			});
		});
	};

	const queueRenderPage = (num: number) => {
		if (pageRendering) {
			pageNumberPending = num;
		} else {
			renderPage(num);
		}
	};

	export const prevPage = () => {
		if (pageNumber <= 1) return;

		pageNumber--;
		queueRenderPage(pageNumber);
	};

	export const nextPage = () => {
		if (pageNumber >= pagesCount) return;

		pageNumber++;
		queueRenderPage(pageNumber);
	};

	export const gotoPage = (seekPage: number) => {
		if (seekPage <= 0 || seekPage >= pagesCount) return;
		pageNumber = seekPage;
		queueRenderPage(seekPage);
	};

	$: if (pdfjs.GlobalWorkerOptions) {
		pdfjs.GlobalWorkerOptions.workerSrc = workerURL;
	}

	$: if (typeof document !== 'undefined' && !pdfDoc) {
		canvas = document.getElementById('the-canvas')! as HTMLCanvasElement;
		if (canvas) {
			ctx = canvas.getContext('2d')!;
			pdfjs.getDocument(url).promise.then((doc: pdfjs.PDFDocumentProxy) => {
				pdfDoc = doc;
				pagesCount = pdfDoc.numPages;

				renderPage(pageNumber);
			});
		}
	}
</script>

<slot name="top-actions" />

<canvas id="the-canvas" class={$$props.class || 'pdf-viewer'} />

<slot name="bottom-actions" />

<style>
	.pdf-viewer {
		width: 100%;
		height: 100%;
	}
</style>
