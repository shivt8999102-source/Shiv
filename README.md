fist website               const pageWidth = doc.internal.pageSize.getWidth();
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

