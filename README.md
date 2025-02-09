# AVIF-to-JPG
AVIF to JPG converter
Here's a detailed explanation of how to convert AVIF to JPG using JavaScript and HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AVIF to JPG Converter</title>
</head>
<body>
    <input type="file" id="avifInput" accept=".avif">
    <a id="downloadLink" style="display: none;">Download JPG</a>

    <script>
        // Check for AVIF support
        async function isAVIFSupported() {
            try {
                await createImageBitmap(new Blob([''], { type: 'image/avif' }));
                return true;
            } catch (error) {
                return false;
            }
        }

        async function convertAVIFtoJPG(avifFile) {
            const avifSupport = await isAVIFSupported();
            let imageBitmap;

            if (avifSupport) {
                // Native decoding
                imageBitmap = await createImageBitmap(avifFile);
            } else {
                // Fallback using library (jsquash)
                const { AVIF } = await import('https://cdn.jsdelivr.net/npm/@jsquash/avif@1.0.0/+esm');
                const avifData = new Uint8Array(await avifFile.arrayBuffer());
                const imageData = await AVIF.decode(avifData);
                
                // Create ImageBitmap from ImageData
                imageBitmap = await createImageBitmap(imageData);
            }

            // Create canvas for conversion
            const canvas = document.createElement('canvas');
            canvas.width = imageBitmap.width;
            canvas.height = imageBitmap.height;
            
            const ctx = canvas.getContext('2d');
            ctx.drawImage(imageBitmap, 0, 0);

            // Convert to JPG
            const jpgUrl = canvas.toDataURL('image/jpeg', 0.85); // 0.85 = quality

            // Create download link
            const downloadLink = document.getElementById('downloadLink');
            downloadLink.href = jpgUrl;
            downloadLink.download = 'converted.jpg';
            downloadLink.style.display = 'block';
        }

        // Handle file input
        document.getElementById('avifInput').addEventListener('change', async (e) => {
            const file = e.target.files[0];
            if (file && file.type === 'image/avif') {
                await convertAVIFtoJPG(file);
            }
        });
    </script>
</body>
</html>
```

Key components explained:

1. **File Input**:
   - Accepts AVIF files through an input element
   - Triggers conversion when file is selected

2. **AVIF Support Check**:
   - Uses `createImageBitmap` to test browser support
   - Helps determine if polyfill is needed

3. **Conversion Process**:
   - **Native Decoding** (if supported):
     - Uses browser's built-in AVIF decoder
   - **Polyfill (jsquash)**:
     - Uses [jsquash library](https://www.npmjs.com/package/@jsquash/avif) for decoding
     - Handles browsers without native AVIF support

4. **Canvas Conversion**:
   - Creates off-screen canvas
   - Draws decoded image to canvas
   - Converts canvas content to JPG using `toDataURL()`

5. **Download Link**:
   - Creates downloadable JPG version
   - Uses data URL for immediate download

Important Considerations:

1. **Browser Support**:
   - Modern browsers (Chrome 85+, Firefox 93+) support AVIF
   - Older browsers need the jsquash polyfill
   - Safari requires macOS Big Sur or newer

2. **Quality Settings**:
   - The second parameter in `toDataURL()` (0.85) controls JPG quality
   - Range: 0 (low quality) to 1 (high quality)

3. **Performance**:
   - Large files may cause memory issues
   - Consider adding progress indicators for big files

4. **Limitations**:
   - Lossy conversion (AVIF → JPG)
   - No EXIF data preservation in this example
   - Alpha channel (transparency) will be lost in JPG

Alternative Libraries:
1. **libavif.js** - More advanced decoding options
2. **Squoosh** - Google's image conversion library
3. **Sharp** (Node.js) - For server-side conversion

Server-Side Alternative (Node.js example):
```javascript
const sharp = require('sharp');

async function convertAvifToJpg(inputPath, outputPath) {
    await sharp(inputPath)
        .jpeg({ quality: 85 })
        .toFile(outputPath);
}
```

Security Considerations:
- Validate file types on both client and server
- Limit maximum file size
- Sanitize file names before saving

This implementation provides a client-side solution that works in most modern browsers. For production use, consider:
- Adding error handling
- Implementing file size limits
- Adding loading indicators
- Handling multiple files
- Adding quality controls for users
