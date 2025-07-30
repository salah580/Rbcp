<!DOCTYPE html>
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
      max-width: 500px;
      margin: auto;
      border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    }
    input[type="file"], button {
      margin: 10px;
      padding: 10px;
    }
    img {
      max-width: 100%;
      margin-top: 15px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>🖼️ إزالة الخلفية وتحويل الصور</h2>
    <input type="file" id="upload" accept="image/*"><br>
    <button onclick="convertToPNG()">تحويل إلى PNG</button>
    <button onclick="removeBackground()">إزالة الخلفية</button>
    <div>
      <img id="preview" src="" alt="المعاينة ستظهر هنا">
    </div>
    <a id="download" style="display:none;" download="processed.png">تحميل الصورة</a>
  </div>

  <script>
    const upload = document.getElementById('upload');
    const preview = document.getElementById('preview');
    const download = document.getElementById('download');
    let uploadedImage = null;

    upload.addEventListener('change', (e) => {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (event) => {
          preview.src = event.target.result;
          uploadedImage = event.target.result;
        };
        reader.readAsDataURL(file);
      }
    });

    function convertToPNG() {
      if (!uploadedImage) return alert('الرجاء رفع صورة أولاً');
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      const img = new Image();
      img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
        const png = canvas.toDataURL('image/png');
        preview.src = png;
        download.href = png;
        download.style.display = 'block';
      };
      img.src = uploadedImage;
    }

    async function removeBackground() {
      if (!uploadedImage) return alert('الرجاء رفع صورة أولاً');
      alert("لإزالة الخلفية تحتاج استخدام API مثل remove.bg (يجب إضافة مفتاح API).");
      // هنا يمكن إضافة كود للتعامل مع API حقيقي لإزالة الخلفية
    }
  </script>
</body>
</html>
