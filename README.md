<!DOCTYPE html>
<html lang="hu">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cisco Segédlet - Lapozó</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #2c3e50; color: white; display: flex; flex-direction: column; align-items: center; margin: 0; padding: 20px; }
        #controls { margin-bottom: 20px; position: sticky; top: 10px; background: rgba(0,0,0,0.8); padding: 15px; border-radius: 10px; z-index: 100; box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        #pdf-viewer { border: 1px solid #7f8c8d; background: #ecf0f1; box-shadow: 0 0 30px rgba(0,0,0,0.5); line-height: 0; }
        canvas { display: block; max-width: 100%; height: auto; }
        button { padding: 10px 25px; cursor: pointer; font-size: 16px; border: none; border-radius: 5px; background: #3498db; color: white; margin: 0 10px; transition: background 0.3s; }
        button:hover { background: #2980b9; }
        button:disabled { background: #7f8c8d; cursor: not-allowed; }
        .page-info { font-weight: bold; font-size: 18px; }
        #loading-msg { margin-top: 20px; font-style: italic; }
    </style>
</head>
<body>

    <h2 id="title">Cisco eszközök beállításai</h2>

    <div id="controls">
        <button id="prev"> < Előző </button>
        <span class="page-info">Oldal: <span id="page-num">0</span> / <span id="page-count">0</span></span>
        <button id="next"> Következő > </button>
    </div>

    <div id="loading-msg">PDF betöltése...</div>

    <div id="pdf-viewer">
        <canvas id="pdf-canvas"></canvas>
    </div>

    <script>
        const pdfjsLib = window['pdfjs-dist/build/pdf'];
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';

        // ITT ADTAM MEG A FÁJLNEVET:
        const url = 'Cisco_eszkozok_beallitasai.pdf'; 

        let pdfDoc = null,
            pageNum = 1,
            canvas = document.getElementById('pdf-canvas'),
            ctx = canvas.getContext('2d'),
            pageIsRendering = false,
            pageNumPending = null;

        // PDF Betöltése automatikusan
        pdfjsLib.getDocument(url).promise.then(pdf => {
            pdfDoc = pdf;
            document.getElementById('page-count').textContent = pdf.numPages;
            document.getElementById('loading-msg').style.display = 'none';
            renderPage(pageNum);
        }).catch(err => {
            document.getElementById('loading-msg').textContent = "Hiba: A PDF fájl nem található a szerveren. Ellenőrizd a fájlnevet!";
            console.error(err);
        });

        function renderPage(num) {
            pageIsRendering = true;
            pdfDoc.getPage(num).then(page => {
                const viewport = page.getViewport({ scale: 1.5 });
                canvas.height = viewport.height;
                canvas.width = viewport.width;

                const renderContext = {
                    canvasContext: ctx,
                    viewport: viewport
                };

                page.render(renderContext).promise.then(() => {
                    pageIsRendering = false;
                    if (pageNumPending !== null) {
                        renderPage(pageNumPending);
                        pageNumPending = null;
                    }
                });

                document.getElementById('page-num').textContent = num;
            });
        }

        function queueRenderPage(num) {
            if (pageIsRendering) {
                pageNumPending = num;
            } else {
                renderPage(num);
            }
        }

        document.getElementById('prev').addEventListener('click', () => {
            if (pageNum <= 1) return;
            pageNum--;
            queueRenderPage(pageNum);
        });

        document.getElementById('next').addEventListener('click', () => {
            if (pageNum >= pdfDoc.numPages) return;
            pageNum++;
            queueRenderPage(pageNum);
        });
    </script>
</body>
</html>