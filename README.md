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
      background: linear-gradient(135deg, #eef2f3, #d9e7f7);
      margin: 0;
      padding: 20px;
      color: #333;
    }

    /* الحاوية الرئيسية */
    .container {
      background: white;
      padding: 25px;
      max-width: 850px;
      margin: auto;
      border-radius: 16px;
      text-align: center;
      box-shadow: 0 0 15px rgba(13, 110, 253, 0.5);
      animation: fadeIn 1s ease;
    }

    /* عنوان رئيسي مع توهج */
    h2 {
      color: #2c3e50;
      margin-bottom: 10px;
      font-size: 28px;
    }
    h2 i {
      color: #0d6efd;
      margin-left: 8px;
      filter: drop-shadow(0 0 8px #0d6efd);
      animation: pulse 2s infinite;
    }

    /* منطقة الرفع */
    .upload-area {
      border: 2px dashed #0d6efd;
      border-radius: 12px;
      padding: 30px;
      margin: 20px 0;
      background: #fdfdfd;
      cursor: pointer;
      transition: 0.3s;
      filter: drop-shadow(0 0 6px rgba(13, 110, 253, 0.4));
    }
    .upload-area:hover {
      background: #e8f2ff;
      transform: scale(1.02);
      filter: drop-shadow(0 0 12px rgba(13, 110, 253, 0.7));
    }

    /* أيقونة رفع */
    .upload-area i {
      color: #0d6efd;
      font-size: 28px;
      margin-bottom: 8px;
      animation: float 3s ease-in-out infinite;
    }

    /* منطقة العرض */
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
      animation: fadeUp 0.8s ease;
    }
    img {
      max-width: 100%;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
      transition: transform 0.3s ease;
    }
    img:hover { transform: scale(1.03); }

    /* الأزرار */
    select, button {
      padding: 12px 20px;
      margin: 10px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
      transition: transform 0.3s ease, box-shadow 0.3s ease;
    }
    button {
      background: #0d6efd;
      color: white;
      box-shadow: 0 0 8px rgba(13, 110, 253, 0.6);
    }
    button:hover {
      background: #0a58ca;
      transform: scale(1.05);
      box-shadow: 0 0 14px rgba(10, 88, 202, 0.9);
    }

    /* رابط التحميل */
    a#download {
      display: inline-block;
      margin-top: 15px;
      background: #198754;
      color: white;
      padding: 10px 18px;
      border-radius: 8px;
      text-decoration: none;
      box-shadow: 0 0 8px rgba(25, 135, 84, 0.7);
      transition: transform 0.3s ease, box-shadow 0.3s ease;
    }
    a#download:hover {
      background: #146c43;
      transform: scale(1.05);
      box-shadow: 0 0 14px rgba(20, 108, 67, 0.9);
    }

    /* نص التحميل */
    .loading { color: #007bff; font-weight: bold; }

    /* نافذة القص */
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
      animation: zoomIn 0.4s ease;
    }
    #crop-box button { margin: 10px 5px; }

    /* الحركات */
    @keyframes fadeIn { from {opacity: 0; transform: translateY(-20px);} to {opacity: 1; transform: translateY(0);} }
    @keyframes fadeUp { from {opacity: 0; transform: translateY(20px);} to {opacity: 1; transform: translateY(0);} }
    @keyframes zoomIn { from {transform: scale(0.7); opacity: 0;} to {transform: scale(1); opacity: 1;} }
    @keyframes pulse { 0%,100% {transform: scale(1);} 50% {transform: scale(1.1);} }
    @keyframes float { 0%,100% {transform: translateY(0);} 50% {transform: translateY(-6px);} }
  </style>
</head>
<body>
  <div class="container">
    <h2><i class="fa-solid fa-scissors"></i> إزالة الخلفية وتحويل الصور</h2>
    <p>🚀 أداة سهلة لإزالة الخلفية، تحويل الصور، وقصها بشكل احترافي</p>

    <!-- منطقة الرفع -->
    <div class="upload-area" id="upload-area">
      <i class="fa-solid fa-cloud-arrow-up"></i><br>
      <span>اسحب الصورة هنا أو انقر لاختيارها</span>
      <input type="file" id="file-input" accept="image/*" hidden>
    </div>

    <label for="format">📂 اختر صيغة التحميل:</label>
    <select id="format">
      <option value="png">PNG</option>
      <option value="jpeg">JPG</option>
      <option value="webp">WebP</option>
    </select><br>

    <!-- الأزرار -->
    <button onclick="convertImage()"><i class="fa-solid fa-repeat"></i> تحويل</button>
    <button onclick="removeBackground()"><i class="fa-solid fa-eraser"></i> إزالة الخلفية</button>
    <button onclick="openCropper()"><i class="fa-solid fa-crop"></i> قص الصورة</button>
    <p id="status" class="loading"></p>

    <!-- المعاينة -->
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
    uploadArea.addEventListener('dragleave', () => uploadArea.style.borderColor = "#0d6efd");
    uploadArea.addEventListener('drop', (e) => {
      e.preventDefault();
      uploadArea.style.borderColor = "#0d6efd";
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
