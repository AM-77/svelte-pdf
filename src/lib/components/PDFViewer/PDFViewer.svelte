<script lang="ts">
	import * as pdfjs from 'pdfjs-dist';

	export let url: string;

	export let pageNumber: number = 1;
	export let pagesCount: number = 0;

	let pdfDoc: pdfjs.PDFDocumentProxy;
	let pageRendering: boolean = false;
	let pageNumberPending: number | null = null;
	let ctx: CanvasRenderingContext2D;
	let canvas: HTMLCanvasElement;
	let scale: number = 1.5;

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

	export const zoomIn = () => {
		if (scale >= 2.5) return;
		scale += 0.1;
		queueRenderPage(pageNumber);
	};

	export const zoomOut = () => {
		if (scale <= 0.8) return;
		scale -= 0.1;
		queueRenderPage(pageNumber);
	};

	$: if (pdfjs.GlobalWorkerOptions) {
		const workerSrc = new URL('pdfjs-dist/build/pdf.worker.js', import.meta.url).toString();
		pdfjs.GlobalWorkerOptions.workerSrc = workerSrc;
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

<canvas id="the-canvas" class={$$props.class} />

<slot name="bottom-actions" />
