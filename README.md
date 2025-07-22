# Home-Store
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأبواب والنوافذ</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      background-color: #f5f5f5;
      color: #333;
      margin: 20px;
    }
    .container {
      background: white;
      border-radius: 8px;
      padding: 20px;
      max-width: 800px;
      margin: auto;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }
    img.logo {
      width: 150px;
      display: block;
      margin: 0 auto 20px;
    }
    label {
      font-weight: bold;
      display: block;
      margin-top: 10px;
    }
    select, input {
      padding: 8px;
      width: 100%;
      margin-top: 5px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    button {
      margin-top: 15px;
      padding: 10px;
      background-color: #444;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background-color: #666;
    }
    .results {
      margin-top: 20px;
      padding: 10px;
      background-color: #eaeaea;
      border-radius: 4px;
    }
  </style>
</head>
<body>

<div class="container">
  <img src="logo.png" alt="شعار الشركة" class="logo" />

  <label>نوع المنتج:</label>
  <select id="type" onchange="updateDefaults()">
    <option value="دبل جلاس دبل فريم">دبل جلاس دبل فريم</option>
    <option value="دبل جلاس سنجل فريم">دبل جلاس سنجل فريم</option>
    <option value="سنجل فريم سنجل جلاس">سنجل فريم سنجل جلاس</option>
    <option value="سلايدنج">نوافذ سلايدنج</option>
    <option value="كهربائية">نوافذ كهربائية</option>
    <option value="سكاي لايت">سكاي لايت</option>
    <option value="كارتن وول">كارتن وول</option>
    <option value="باب WPC">باب WPC</option>
    <option value="باب ألمنيوم">باب ألمنيوم</option>
    <option value="باب دورة مياه">باب دورة مياه</option>
    <option value="باب سحب">باب سحب</option>
    <option value="باب فولدنج">باب فولدنج</option>
    <option value="باب مدخل">باب مدخل</option>
    <option value="باب حديقة">باب حديقة</option>
    <option value="حاجز">حاجز</option>
  </select>

  <label>النوع الفرعي (ثابت / حركة / حركتين):</label>
  <select id="subtype">
    <option value="ثابت">ثابت</option>
    <option value="حركة">حركة</option>
    <option value="حركتين">حركتين</option>
  </select>

  <label>الطول (متر):</label>
  <input type="number" id="length" step="0.01" />

  <label>العرض (متر):</label>
  <input type="number" id="width" step="0.01" />

  <label>الكمية:</label>
  <input type="number" id="quantity" value="1" min="1" />

  <label>إضافة:</label>
  <select id="addon" onchange="toggleCurtainInputs()">
    <option value="بدون">بدون</option>
    <option value="ستارة">ستارة</option>
    <option value="شبك">شبك</option>
  </select>

  <div id="curtainInputs" style="display:none;">
    <label>طول الستارة:</label>
    <input type="number" id="curtainLength" step="0.01" />
    <label>عرض الستارة:</label>
    <input type="number" id="curtainWidth" step="0.01" />
  </div>

  <button onclick="calculate()">احسب</button>
  <button onclick="clearResults()">🗑️ مسح</button>
  <button onclick="downloadPDF()">💾 حفظ PDF</button>

  <div class="results" id="results"></div>
</div>

<script>
  const prices = {
    "دبل جلاس دبل فريم": { ثابت: 34, حركة: 73, حركتين: 92, cbm: 0.13 },
    "دبل جلاس سنجل فريم": { ثابت: 31, حركة: 57, حركتين: 75, cbm: 0.07 },
    "سنجل فريم سنجل جلاس": { ثابت: 31, حركة: 57, حركتين: 75, cbm: 0.07 },
    "سلايدنج": { ثابت: 34, حركة: 57, حركتين: 75, cbm: 0.13 },
    "كهربائية": { ثابت: 40, حركة: 70, حركتين: 85, cbm: 0.13 },
    "سكاي لايت": { ثابت: 40, حركة: 70, حركتين: 85, cbm: 0.13 },
    "كارتن وول": { ثابت: 34, حركة: 34, حركتين: 34, cbm: 0.13 },
    "باب WPC": { ثابت: 165, cbm: 0.11 },
    "باب ألمنيوم": { ثابت: 165, cbm: 0.11 },
    "باب دورة مياه": { ثابت: 165, cbm: 0.11 },
    "باب سحب": { ثابت: 165, cbm: 0.13 },
    "باب فولدنج": { ثابت: 165, cbm: 0.13 },
    "باب مدخل": { ثابت: 165, cbm: 0.2 },
    "باب حديقة": { ثابت: 165, cbm: 0.2 },
    "حاجز": { ثابت: 165, cbm: 0.05 },
  };

  function updateDefaults() {
    const type = document.getElementById("type").value;
    if (["باب WPC", "باب ألمنيوم", "باب دورة مياه"].includes(type)) {
      document.getElementById("length").value = 2.2;
      document.getElementById("width").value = 1;
    }
  }

  function toggleCurtainInputs() {
    const addon = document.getElementById("addon").value;
    document.getElementById("curtainInputs").style.display = addon === "ستارة" ? "block" : "none";
  }

  function calculate() {
    const type = document.getElementById("type").value;
    const subtype = document.getElementById("subtype").value;
    const length = parseFloat(document.getElementById("length").value);
    const width = parseFloat(document.getElementById("width").value);
    const quantity = parseInt(document.getElementById("quantity").value);
    const addon = document.getElementById("addon").value;

    const area = length * width;
    const basePrice = prices[type][subtype] || prices[type].ثابت;
    const cbm = prices[type].cbm;
    const shipping = area * cbm * 48;
    let total = (area * basePrice + shipping) * quantity;

    if (addon === "ستارة") {
      const curtainArea = parseFloat(document.getElementById("curtainLength").value) * parseFloat(document.getElementById("curtainWidth").value);
      total += curtainArea * 26;
    }

    const resultText = `
      النوع: ${type} (${subtype})<br>
      المقاس: ${length} × ${width} متر<br>
      الكمية: ${quantity}<br>
      الإضافة: ${addon}<br>
      التكلفة مع الشحن: ${total.toFixed(2)} ريال
    `;

    document.getElementById("results").innerHTML += "<hr>" + resultText;
  }

  function clearResults() {
    document.getElementById("results").innerHTML = "";
  }

  function downloadPDF() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    doc.text(document.getElementById("results").innerText, 10, 10);
    doc.save("النتائج.pdf");
  }
</script>
</body>
</html>
