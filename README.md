
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>إزالة الخلفية وتحويل الصور</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background: #f5f5f5;
      margin: 0;
      padding: 20px;
    }
    .container {
      background: white;
      padding: 20px;
      max-width: 750px;
      margin: auto;
      border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    }
    .upload-area {
      border: 2px dashed #aaa;
      border-radius: 10px;
      padding: 20px;
      margin: 15px 0;
      background: #fafafa;
      cursor: pointer;
    }
    .upload-area.dragover {
      border-color: #007bff;
      background: #e9f5ff;
    }
    .preview-container {
      display: flex;
      justify-content: space-around;
      margin-top: 15px;
    }
    .preview-container div {
      width: 45%;
    }
    img {
      max-width: 100%;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
    button, select {
      margin: 8px;
      padding: 10px 15px;
      cursor: pointer;
    }
    .loading {
      font-size: 14px;
      color: #007bff;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>🖼️ إزالة الخلفية وتحويل الصور</h2>
    <div class="upload-area" id="upload-area">
      اسحب وأفلت الصورة هنا أو انقر لاختيار ملف
      <input type="file" id="file-input" accept="image/*" hidden>
    </div>

    <label for="format">📂 اختر صيغة التحميل:</label>
    <select id="format">
      <option value="png">PNG</option>
      <option value="jpeg">JPG</option>
      <option value="webp">WebP</option>
    </select>
    <br>
    <button onclick="convertImage()">تحويل الصورة</button>
    <button onclick="removeBackground()">إزالة الخلفية</button>
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
    <a id="download" style="display:none;">تحميل الصورة</a>
  </div>

  <script>
    const uploadArea = document.getElementById('upload-area');
    const fileInput = document.getElementById('file-input');
    const beforePreview = document.getElementById('before-preview');
    const afterPreview = document.getElementById('after-preview');
    const download = document.getElementById('download');
    const statusText = document.getElementById('status');
    const formatSelect = document.getElementById('format');
    let uploadedFile = null;

    uploadArea.addEventListener('click', () => fileInput.click());
    fileInput.addEventListener('change', (e) => handleFile(e.target.files[0]));

    uploadArea.addEventListener('dragover', (e) => {
      e.preventDefault();
      uploadArea.classList.add('dragover');
    });
    uploadArea.addEventListener('dragleave', () => {
      uploadArea.classList.remove('dragover');
    });
    uploadArea.addEventListener('drop', (e) => {
      e.preventDefault();
      uploadArea.classList.remove('dragover');
      if (e.dataTransfer.files.length > 0) {
        handleFile(e.dataTransfer.files[0]);
      }
    });

    function handleFile(file) {
      if (!file.type.startsWith('image/')) return alert('الرجاء اختيار صورة فقط!');
      uploadedFile = file;
      const reader = new FileReader();
      reader.onload = (event) => {
        beforePreview.src = event.target.result;
        afterPreview.src = '';
        download.style.display = 'none';
      };
      reader.readAsDataURL(file);
    }

    function convertImage() {
      if (!beforePreview.src) return alert('الرجاء رفع صورة أولاً');
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
        download.style.display = 'block';
      };
      img.src = beforePreview.src;
    }

    async function removeBackground() {
      if (!uploadedFile) return alert('الرجاء رفع صورة أولاً');

      statusText.textContent = '⏳ جارٍ معالجة الصورة...';
      
      const formData = new FormData();
      formData.append('image_file', uploadedFile);
      formData.append('size', 'auto');

      try {
        const response = await fetch('https://api.remove.bg/v1.0/removebg', {
          method: 'POST',
          headers: { 'X-Api-Key': 'VrVtmhP13pw8w6WdRw99yNmk' }, // ✅ مفتاحك
          body: formData,
        });

        if (!response.ok) throw new Error('فشل الاتصال بـ API');

        const blob = await response.blob();
        const imageUrl = URL.createObjectURL(blob);
        afterPreview.src = imageUrl;
        download.href = imageUrl;
        download.download = 'removed-bg.png';
        download.style.display = 'block';
        statusText.textContent = '✅ تمت إزالة الخلفية بنجاح!';
      } catch (error) {
        console.error(error);
        alert('حدث خطأ أثناء إزالة الخلفية');
        statusText.textContent = '';
      }
    }
  </script>
</body>
</html>
