# App

<html lang="en">

<head>
    <meta charset="UTF-8" />
    <title>Paste Screenshot and Extract Text for Excel (Tabular)</title>
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4/dist/tesseract.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }

        #result {
            margin-top: 20px;
            white-space: pre-wrap;
            border: 1px solid #ccc;
            padding: 10px;
            min-height: 100px;
        }

        #imagePreview {
            margin-top: 20px;
            max-width: 300px;
            display: none;
            border: 1px solid #aaa;
        }

        #copyBtn {
            margin-top: 10px;
            display: none;
        }
    </style>
</head>

<body>
    <h2>Paste Screenshot from Clipboard</h2>
    <button id="pasteBtn">Paste from Clipboard</button>
    <div id="result">Extracted text will appear here...</div>
    <button id="copyBtn">Copy for Excel</button>
    <img id="imagePreview" />

    <script>
        const pasteBtn = document.getElementById('pasteBtn');
        const copyBtn = document.getElementById('copyBtn');
        const resultDiv = document.getElementById('result');
        const imagePreview = document.getElementById('imagePreview');

        pasteBtn.addEventListener('click', async () => {
            resultDiv.textContent = 'Waiting for clipboard...';
            copyBtn.style.display = 'none';
            imagePreview.style.display = 'none';

            try {
                if (!navigator.clipboard.read) {
                    alert('Your browser does not support reading images from clipboard!');
                    return;
                }

                const items = await navigator.clipboard.read();
                let foundImage = false;

                for (const item of items) {
                    for (const type of item.types) {
                        if (type.startsWith('image/')) {
                            foundImage = true;
                            const blob = await item.getType(type);
                            const imageURL = URL.createObjectURL(blob);
                            imagePreview.src = imageURL;
                            imagePreview.style.display = 'block';

                            resultDiv.textContent = 'Running OCR... please wait.';

                            const { data: { text } } = await Tesseract.recognize(blob, 'eng', {
                                logger: m => console.log(m)
                            });

                            const cleaned = convertToTabSeparated(text);
                            resultDiv.textContent = cleaned || 'No text detected!';
                            if (cleaned.trim()) {
                                copyBtn.style.display = 'inline-block';
                            }
                            return;
                        }
                    }
                }

                if (!foundImage) {
                    resultDiv.textContent = 'No image found in clipboard!';
                }
            } catch (err) {
                console.error(err);
                resultDiv.textContent = 'Error reading clipboard or running OCR.';
            }
        });

        function convertToTabSeparated(text) {
            return text
                .split('\n')
                .map(line =>
                    line
                        .trim()
                        .split(/\s+/)     // split by spaces (multiple spaces count)
                        .join('\t')       // join with tab
                )
                .filter(line => line.length > 0)
                .join('\n');
        }

        copyBtn.addEventListener('click', async () => {
            const text = resultDiv.textContent;
            try {
                await navigator.clipboard.writeText(text);
                alert('Copied! Now paste directly into Excel and it will go into columns.');
            } catch (err) {
                console.error(err);
                alert('Failed to copy text.');
            }
        });
    </script>
</body>

</html>
