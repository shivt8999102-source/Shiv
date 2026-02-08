<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bulk Image to PDF - Multi-Select Enabled</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&display=swap');
        body { 
            font-family: 'Poppins', sans-serif;
            background-color: #f0fdf4; 
        }
        .green-gradient {
            background: linear-gradient(135deg, #166534 0%, #15803d 100%);
        }
        .custom-shadow {
            box-shadow: 0 10px 25px -5px rgba(22, 101, 52, 0.1);
        }
        .loader {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #22c55e;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="min-h-screen pb-10">

    <div class="green-gradient text-white py-10 px-4 shadow-lg mb-8 text-center">
        <h1 class="text-3xl md:text-5xl font-bold mb-2">Multi-Photo PDF Converter</h1>
        <p class="text-green-100 italic">Select 50+ images at once and convert instantly</p>
    </div>

    <div class="max-w-6xl mx-auto px-4">
        <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">
            
            <!-- Sidebar Controls -->
            <div class="lg:col-span-1">
                <div class="bg-white p-6 rounded-2xl custom-shadow sticky top-5 border border-green-100">
                    <h2 class="text-lg font-bold text-green-800 mb-4 uppercase tracking-wider">PDF Options</h2>
                    
                    <div class="space-y-4">
                        <div>
                            <label class="block text-xs font-bold text-gray-400 mb-1">File Name</label>
                            <input type="text" id="pdfName" value="My_Images_PDF" 
                                class="w-full px-4 py-2 bg-green-50 border border-green-200 rounded-lg outline-none focus:ring-2 focus:ring-green-500">
                        </div>
                        
                        <div>
                            <label class="block text-xs font-bold text-gray-400 mb-1">Page Format</label>
                            <select id="orientation" class="w-full px-4 py-2 bg-green-50 border border-green-200 rounded-lg outline-none">
                                <option value="p">Portrait (Vertical)</option>
                                <option value="l">Landscape (Horizontal)</option>
                            </select>
                        </div>

                        <div class="bg-green-50 p-4 rounded-xl text-center">
                            <span class="text-sm text-green-700 font-medium">Total Images</span>
                            <div id="totalCount" class="text-3xl font-bold text-green-900">0</div>
                        </div>

                        <button id="downloadBtn" disabled class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-4 rounded-xl shadow-lg transition-all disabled:opacity-30 flex items-center justify-center gap-2">
                            Download PDF
                        </button>
                        
                        <button id="clearBtn" class="w-full text-gray-400 hover:text-red-500 text-sm font-medium transition-colors">Clear All Images</button>
                    </div>
                </div>
            </div>

            <!-- Main Upload & Preview Area -->
            <div class="lg:col-span-3">
                <!-- Dropzone with Multi-Select Fix -->
                <div id="dropZone" onclick="document.getElementById('fileInput').click()" class="relative bg-white border-2 border-dashed border-green-300 rounded-3xl p-16 text-center transition-all hover:border-green-600 hover:bg-green-50/50 cursor-pointer">
                    <!-- CRITICAL: multiple attribute ensures you can select many files -->
                    <input type="file" id="fileInput" multiple accept="image/*" class="hidden">
                    
                    <div class="flex flex-col items-center">
                        <div class="w-20 h-20 bg-green-100 text-green-600 rounded-full flex items-center justify-center mb-4">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-10 w-10" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                            </svg>
                        </div>
                        <h3 class="text-xl font-bold text-gray-800">Click to Select Bulk Images</h3>
                        <p class="text-gray-500 mt-2">Mobile: Long-press to select multiple. PC: Ctrl + Click.</p>
                        <span class="mt-4 px-6 py-2 bg-green-600 text-white rounded-full text-sm font-bold shadow-md">Browse Gallery</span>
                    </div>
                </div>

                <!-- Preview Grid -->
                <div id="previewContainer" class="mt-8 bg-white p-6 rounded-3xl custom-shadow hidden border border-gray-100">
                    <h3 class="text-sm font-bold text-gray-400 uppercase mb-4 tracking-widest">Image Sequence</h3>
                    <div id="imageGrid" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-4">
                        <!-- Items injected here -->
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Full Screen Loader Overlay -->
    <div id="overlay" class="fixed inset-0 bg-white/90 z-50 flex-col items-center justify-center hidden flex">
        <div class="loader mb-4"></div>
        <p class="text-green-800 font-bold text-xl" id="statusText">Processing Images...</p>
        <p class="text-gray-500 text-sm mt-1">Converting your bulk files into a single PDF</p>
    </div>

    <script>
        const { jsPDF } = window.jspdf;
        let images = [];

        const fileInput = document.getElementById('fileInput');
        const imageGrid = document.getElementById('imageGrid');
        const previewContainer = document.getElementById('previewContainer');
        const totalCount = document.getElementById('totalCount');
        const downloadBtn = document.getElementById('downloadBtn');
        const clearBtn = document.getElementById('clearBtn');
        const overlay = document.getElementById('overlay');
        const statusText = document.getElementById('statusText');

        // Handle File Selection
        fileInput.addEventListener('change', async (e) => {
            const files = Array.from(e.target.files).filter(f => f.type.startsWith('image/'));
            if(files.length === 0) return;

            overlay.classList.remove('hidden');
            statusText.innerText = `Adding ${files.length} images...`;

            for (const file of files) {
                const data = await new Promise(resolve => {
                    const reader = new FileReader();
                    reader.onload = (ev) => resolve(ev.target.result);
                    reader.readAsDataURL(file);
                });
                images.push({ data });
            }

            render();
            overlay.classList.add('hidden');
            fileInput.value = ''; // Reset for next selection
        });

        function render() {
            imageGrid.innerHTML = '';
            totalCount.innerText = images.length;
            
            if(images.length > 0) {
                previewContainer.classList.remove('hidden');
                downloadBtn.disabled = false;
            } else {
                previewContainer.classList.add('hidden');
                downloadBtn.disabled = true;
            }

            images.forEach((img, idx) => {
                const div = document.createElement('div');
                div.className = "relative aspect-square rounded-xl overflow-hidden border-2 border-green-50 group";
                div.innerHTML = `
                    <img src="${img.data}" class="w-full h-full object-cover">
                    <div class="absolute inset-0 bg-black/40 opacity-0 group-hover:opacity-100 flex items-center justify-center transition-all">
                        <button onclick="removeImg(${idx})" class="bg-red-500 text-white p-2 rounded-full shadow-lg">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12" />
                            </svg>
                        </button>
                    </div>
                    <div class="absolute top-1 left-1 bg-green-600 text-[10px] text-white font-bold px-1.5 py-0.5 rounded shadow">P${idx+1}</div>
                `;
                imageGrid.appendChild(div);
            });
        }

        window.removeImg = (i) => {
            images.splice(i, 1);
            render();
        }

        clearBtn.onclick = () => {
            images = [];
            render();
        }

        downloadBtn.onclick = async () => {
            overlay.classList.remove('hidden');
            statusText.innerText = "Generating PDF Document...";

            const orientation = document.getElementById('orientation').value;
            const pdfName = document.getElementById('pdfName').value || 'My_Export';

            try {
                const doc = new jsPDF(orientation, 'mm', 'a4');
                const pageWidth = doc.internal.pageSize.getWidth();
                const pageHeight = doc.internal.pageSize.getHeight();

                for (let i = 0; i < images.length; i++) {
                    if (i > 0) doc.addPage();
                    
                    const img = new Image();
                    img.src = images[i].data;
                    await new Promise(r => img.onload = r);

                    const imgRatio = img.width / img.height;
                    const pageRatio = pageWidth / pageHeight;

                    let w, h;
                    if (imgRatio > pageRatio) {
                        w = pageWidth;
                        h = pageWidth / imgRatio;
                    } else {
                        h = pageHeight;
                        w = pageHeight * imgRatio;
                    }

                    doc.addImage(images[i].data, 'JPEG', (pageWidth - w)/2, (pageHeight - h)/2, w, h, undefined, 'FAST');
                }

                doc.save(`${pdfName}.pdf`);
            } catch (err) {
                console.error(err);
            } finally {
                overlay.classList.add('hidden');
            }
        };
    </script>
</body>
</html>

