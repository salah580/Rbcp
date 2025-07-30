<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>إزالة الخلفية وتحويل الصور</title>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.css" rel="stylesheet"/>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css"/>
  <style>
    body {
      font-family: 'Cairo', sans-serif;
      background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
      margin: 0;
      padding: 20px;
      color: #333;
    }
    .container {
      background: white;
      padding: 25px;
      max-width: 850px;
      margin: auto;
      border-radius: 16px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.15);
      text-align: center;
    }
    h2 {
      color: #2c3e50;
      margin-bottom: 10px;
    }
    .upload-area {
      border: 2px dashed #aaa;
      border-radius: 12px;
      padding: 30px;
      margin: 20px 0;
      background: #fdfdfd;
      cursor: pointer;
      transition: 0.3s;
    }
    .upload-area:hover { border-color: #007bff; background: #f0f8ff; }
    .preview-container {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-around;
      margin-top: 20px;
      gap: 20px;
    }
    .preview-container div {
      flex: 1;
      min-width: 250px;
    }
    img {
      max-width: 100%;
      border-radius: 12px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    select, button {
      padding: 12px 20px;
      margin: 10px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
      transition: 0.3s;
    }
    button {
      background: #007bff;
      color: white;
    }
    button:hover { background: #0056b3; }
    select { border: 1px solid #ccc; }
    a#download {
      display: inline-block;
      margin-top: 15px;
      background: #28a745;
      color: white;
      padding: 10px 18px;
      border-radius: 8px;
      text-decoration: none;
    }
    a#download:hover { background: #1e7e34; }
    .loading { color: #007bff; font-weight: bold; }
    #crop-area {
      display: none;
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.7);
      justify-content: center;
      align-items: center;
      z-index: 9999;
    }
    #crop-box {
      background: white;
      padding: 20px;
      border-radius: 12px;
      max-width: 95%;
      text-align: center;
    }
    #crop-box button {
      margin: 10px 5px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2><i class="fa-solid fa-scissors"></i> إزالة الخلفية وتحويل الصور</h2>
    <p>🚀 أداة سهلة لإزالة الخلفية، تحويل الصور، وقصها بشكل احترافي</p>
    <div class="upload-area" id="upload-area">
      <i class="fa-solid fa-cloud-arrow-up fa-2x"></i><br>
      <span>اسحب الصورة هنا أو انقر لاختيارها</span>
      <input type="file" id="file-input" accept="image/*" hidden>
    </div>
    <label for="format">📂 اختر صيغة التحميل:</label>
    <select id="format">
      <option value="png">PNG</option>
      <option value="jpeg">JPG</option>
      <option value="webp">WebP</option>
    </select><br>
    <button onclick="convertImage()"><i class="fa-solid fa-repeat"></i> تحويل</button>
    <button onclick="removeBackground()"><i class="fa-solid fa-eraser"></i> إزالة الخلفية</button>
    <button onclick="openCropper()"><i class="fa-solid fa-crop"></i> قص الصورة</button>
    <p id="status" class="loading"></p>

    <div class="preview-container">
      <div>
        <h3>📥 قبل</h3>
        <img id="before-preview" src="" alt="قبل">
      </div>
      <div>
        <h3>📤 بعد</h3>
        <img id="after-preview" src="" alt="بعد">
      </div>
    </div>
    <a id="download" style="display:none;"><i class="fa-solid fa-download"></i> تحميل الصورة</a>
  </div>

  <!-- نافذة القص -->
  <div id="crop-area">
    <div id="crop-box">
      <h3>✂️ قص الصورة</h3>
      <img id="crop-image" style="max-width:100%; border-radius:8px;"/>
      <br>
      <button onclick="confirmCrop()"><i class="fa-solid fa-check"></i> تأكيد</button>
      <button onclick="closeCropper()"><i class="fa-solid fa-xmark"></i> إلغاء</button>
    </div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>
  <script>
    const uploadArea = document.getElementById('upload-area');
    const fileInput = document.getElementById('file-input');
    const beforePreview = document.getElementById('before-preview');
    const afterPreview = document.getElementById('after-preview');
    const download = document.getElementById('download');
    const statusText = document.getElementById('status');
    const formatSelect = document.getElementById('format');
    const cropArea = document.getElementById('crop-area');
    const cropImage = document.getElementById('crop-image');
    let uploadedFile = null;
    let cropper = null;

    uploadArea.addEventListener('click', () => fileInput.click());
    fileInput.addEventListener('change', (e) => handleFile(e.target.files[0]));

    uploadArea.addEventListener('dragover', (e) => { e.preventDefault(); uploadArea.style.borderColor = "#007bff"; });
    uploadArea.addEventListener('dragleave', () => uploadArea.style.borderColor = "#aaa");
    uploadArea.addEventListener('drop', (e) => {
      e.preventDefault();
      uploadArea.style.borderColor = "#aaa";
      if (e.dataTransfer.files.length > 0) handleFile(e.dataTransfer.files[0]);
    });

    function handleFile(file) {
      if (!file.type.startsWith('image/')) return alert('⚠️ الرجاء اختيار صورة!');
      uploadedFile = file;
      const reader = new FileReader();
      reader.onload = (event) => {
        beforePreview.src = event.target.result;
        afterPreview.src = '';
        download.style.display = 'none';
      };
      reader.readAsDataURL(file);
    }

    function openCropper() {
      if (!beforePreview.src) return alert('⚠️ الرجاء رفع صورة أولاً');
      cropArea.style.display = 'flex';
      cropImage.src = beforePreview.src;
      setTimeout(() => { cropper = new Cropper(cropImage, { viewMode: 1, movable: true, zoomable: true }); }, 100);
    }

    function confirmCrop() {
      const canvas = cropper.getCroppedCanvas();
      beforePreview.src = canvas.toDataURL();
      uploadedFile = dataURLtoFile(beforePreview.src, 'cropped.png');
      closeCropper();
    }

    function closeCropper() {
      cropArea.style.display = 'none';
      if (cropper) cropper.destroy();
    }

    function dataURLtoFile(dataurl, filename) {
      const arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1], bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
      while (n--) u8arr[n] = bstr.charCodeAt(n);
      return new File([u8arr], filename, { type: mime });
    }

    function convertImage() {
      if (!beforePreview.src) return alert('⚠️ الرجاء رفع صورة أولاً');
      const selectedFormat = formatSelect.value;
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      const img = new Image();
      img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
        const converted = canvas.toDataURL(`image/${selectedFormat}`);
        afterPreview.src = converted;
        download.href = converted;
        download.download = `converted.${selectedFormat}`;
        download.style.display = 'inline-block';
      };
      img.src = beforePreview.src;
    }

    async function removeBackground() {
      if (!uploadedFile) return alert('⚠️ الرجاء رفع صورة أولاً');
      statusText.textContent = '⏳ جارٍ معالجة الصورة...';

      const formData = new FormData();
      formData.append('image_file', uploadedFile);
      formData.append('size', 'auto');

      try {
        const response = await fetch('https://api.remove.bg/v1.0/removebg', {
          method: 'POST',
          headers: { 'X-Api-Key': 'VrVtmhP13pw8w6WdRw99yNmk' },
          body: formData,
        });

        if (!response.ok) throw new Error('فشل الاتصال بـ API');
        const blob = await response.blob();
        const imageUrl = URL.createObjectURL(blob);
        afterPreview.src = imageUrl;
        download.href = imageUrl;
        download.download = 'removed-bg.png';
        download.style.display = 'inline-block';
        statusText.textContent = '✅ تمت إزالة الخلفية بنجاح!';
      } catch (error) {
        console.error(error);
        alert('❌ حدث خطأ أثناء إزالة الخلفية');
        statusText.textContent = '';
      }
    }
  </script>
</body>
</html>
