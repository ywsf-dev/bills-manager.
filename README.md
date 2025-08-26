<!doctype html>
<html lang="ar" dir="rtl"> 
<head> 
  <meta charset="UTF-8"> 
  <title>مدير الفواتير - Bills Manager</title> 
  <meta name="viewport" content="width=device-width, initial-scale=1.0"> 
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css" rel="stylesheet"> 
  <style>
    body { background: #f8f9fa; font-family: 'Tahoma', sans-serif; user-select: none; }
    .invoice-box { background: #fff; padding: 20px; border-radius: 16px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); margin-bottom: 20px; border: 2px solid #dee2e6; }
    table input { width: 100%; text-align: center; }
    .btn-add { background: #198754; color: white; }
    .btn-del { background: #dc3545; color: white; }
    .invoice-header { background: #0d6efd; color: white; padding: 10px; border-radius: 12px; margin-bottom: 15px; }
    footer { margin-top: 40px; text-align: center; font-size: 14px; color: #555; }
  </style> 
</head> 
<body oncontextmenu="return false" onselectstart="return false" onkeydown="return disableKeys(event)"> 
  <div class="container py-4"> 
    <h3 class="text-center mb-4">📑 مدير الفواتير</h3> 

    <!-- بيانات عامة -->
    <div class="row mb-3"> 
      <div class="col-md-6">
        <label class="form-label">اسم البائع:</label> 
        <input type="text" class="form-control" id="sellerName" placeholder="أدخل اسم البائع"> 
      </div>
      <div class="col-md-6">
        <label class="form-label">اسم المشتري:</label> 
        <input type="text" class="form-control" id="buyerName" placeholder="أدخل اسم المشتري"> 
      </div>
    </div>

    <!-- منطقة الفواتير -->
    <div id="invoicesContainer"></div>

    <!-- زر إضافة فاتورة جديدة -->
    <button class="btn btn-primary mb-3" onclick="addInvoice()">➕ إضافة فاتورة جديدة</button>

    <!-- المجموع الكلي لكل الفواتير -->
    <h4 class="text-end mt-4">💰 مجموع كل الفواتير: <span id="allInvoicesTotal">0</span></h4>

    <!-- خيارات الحفظ -->
    <div class="d-flex justify-content-center gap-2 mt-4"> 
      <button class="btn btn-success" onclick="savePDF()">💾 حفظ كـ PDF</button> 
      <button class="btn btn-info" onclick="saveImage()">🖼️ حفظ كصورة</button> 
    </div>
  </div> 

  <!-- سياسة الخصوصية -->
  <footer>
    <p>جميع الحقوق محفوظة © 2025 - هذا التطبيق ملك <strong>Yousef Sheikh Abdullah</strong>.</p>
    <p>🚫 يمنع نسخ أو سرقة أو إعادة استخدام محتوى هذا التطبيق بأي شكل.</p>
  </footer>

  <!-- مكتبات -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script> 
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script> 

  <script>
    // منع الاختصارات مثل Ctrl+U و Ctrl+S و Ctrl+C و F12
    function disableKeys(e) {
      if (
        (e.ctrlKey && (e.key === "u" || e.key === "s" || e.key === "c" || e.key === "x" || e.key === "a" || e.key === "p")) || 
        e.key === "F12"
      ) {
        e.preventDefault();
        return false;
      }
    }

    let invoiceCount = 0;

    function addInvoice() {
      invoiceCount++;
      let container = document.getElementById("invoicesContainer");

      let box = document.createElement("div");
      box.className = "invoice-box";
      box.setAttribute("data-id", invoiceCount);

      box.innerHTML = `
        <div class="invoice-header d-flex justify-content-between align-items-center">
          <div>
            📅 التاريخ: <input type="date" class="form-control d-inline-block w-auto invoice-date ms-2">
          </div>
          <div>
            🧾 رقم الفاتورة: <input type="number" class="form-control d-inline-block w-auto invoice-number" value="${invoiceCount}" min="1">
          </div>
        </div>

        <table class="table table-bordered table-striped text-center align-middle mb-3">
          <thead class="table-primary">
            <tr>
              <th>نوع البضاعة</th>
              <th>الوزن</th>
              <th>السعر</th>
              <th>الناتج</th>
              <th>إجراء</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><input type="text" class="form-control item-name" placeholder="أدخل البضاعة"></td>
              <td><input type="number" class="form-control item-weight" value="1" min="1"></td>
              <td><input type="number" class="form-control item-price" value="0" min="0"></td>
              <td><input type="text" class="form-control item-total" value="0" readonly></td>
              <td><button class="btn btn-del btn-sm" onclick="deleteRow(this)">🗑️</button></td>
            </tr>
          </tbody>
        </table>
        <button class="btn btn-add mb-2" onclick="addRow(this)">➕ إضافة بند</button>
        <h6 class="text-end">إجمالي الفاتورة: <span class="invoice-total">0</span></h6>
      `;

      container.appendChild(box);
      bindEvents(box);
      updateAllInvoicesTotal();
    }

    function addRow(btn) {
      let tbody = btn.closest(".invoice-box").querySelector("tbody");
      let row = document.createElement("tr");
      row.innerHTML = `
        <td><input type="text" class="form-control item-name" placeholder="أدخل البضاعة"></td>
        <td><input type="number" class="form-control item-weight" value="1" min="1"></td>
        <td><input type="number" class="form-control item-price" value="0" min="0"></td>
        <td><input type="text" class="form-control item-total" value="0" readonly></td>
        <td><button class="btn btn-del btn-sm" onclick="deleteRow(this)">🗑️</button></td>
      `;
      tbody.appendChild(row);
      bindEvents(row);
    }

    function deleteRow(btn) {
      btn.closest("tr").remove();
      updateInvoiceTotal(btn.closest(".invoice-box"));
    }

    function bindEvents(container) {
      container.querySelectorAll(".item-weight, .item-price").forEach(input => {
        input.addEventListener("input", () => updateInvoiceTotal(container));
      });
    }

    function updateInvoiceTotal(container) {
      let total = 0;
      container.querySelectorAll("tbody tr").forEach(row => {
        let weight = row.querySelector(".item-weight").value || 0;
        let price = row.querySelector(".item-price").value || 0;
        let rowTotal = weight * price;
        row.querySelector(".item-total").value = rowTotal;
        total += rowTotal;
      });
      container.querySelector(".invoice-total").innerText = total;
      updateAllInvoicesTotal();
    }

    function updateAllInvoicesTotal() {
      let sum = 0;
      document.querySelectorAll(".invoice-total").forEach(span => {
        sum += parseFloat(span.innerText) || 0;
      });
      document.getElementById("allInvoicesTotal").innerText = sum;
    }

    async function saveImage() {
      let element = document.body;
      let canvas = await html2canvas(element, { scale: 2 });
      let link = document.createElement("a");
      link.download = "الفواتير.png";
      link.href = canvas.toDataURL("image/png");
      link.click();
    }

    async function savePDF() {
      const { jsPDF } = window.jspdf;
      let doc = new jsPDF("p", "pt", "a4");
      let element = document.body;
      let canvas = await html2canvas(element, { scale: 2 });
      let imgData = canvas.toDataURL("image/png");
      let imgWidth = 595.28; 
      let pageHeight = 841.89; 
      let imgHeight = canvas.height * imgWidth / canvas.width;
      let heightLeft = imgHeight;
      let position = 0;

      doc.addImage(imgData, "PNG", 0, position, imgWidth, imgHeight);
      heightLeft -= pageHeight;

      while (heightLeft > 0) {
        position = heightLeft - imgHeight;
        doc.addPage();
        doc.addImage(imgData, "PNG", 0, position, imgWidth, imgHeight);
        heightLeft -= pageHeight;
      }
      doc.save("الفواتير.pdf");
    }
  </script> 
</body>
</html>
