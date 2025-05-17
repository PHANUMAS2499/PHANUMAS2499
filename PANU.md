<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <title>เข้าสู่ระบบ - ระบบยืมคืนอุปกรณ์</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
</head>
<body class="p-4">

  <!-- หน้าล็อกอิน -->
  <div id="loginPage">
    <h2 class="mb-4">เข้าสู่ระบบ</h2>
    <div class="mb-3">
      <input type="password" class="form-control" id="passwordInput" placeholder="กรอกรหัสผ่าน">
    </div>
    <button class="btn btn-primary" onclick="checkLogin()">เข้าสู่ระบบ</button>
    <p class="text-danger mt-2" id="loginError" style="display:none;">รหัสผ่านไม่ถูกต้อง</p>
  </div>

  <!-- ระบบจัดการอุปกรณ์ -->
  <div id="mainApp" style="display:none;">
    <h2 class="mb-4">ระบบยืมคืนอุปกรณ์</h2>

    <form id="form" class="row g-3 mb-4">
      <div class="col-md-3">
        <input type="text" class="form-control" id="device" placeholder="ชื่ออุปกรณ์" required>
      </div>
      <div class="col-md-3">
        <input type="text" class="form-control" id="borrower" placeholder="ผู้ยืม" required>
      </div>
      <div class="col-md-2">
        <input type="date" class="form-control" id="borrowDate" required>
      </div>
      <div class="col-md-2">
        <input type="date" class="form-control" id="returnDate">
      </div>
      <div class="col-md-2">
        <button type="submit" class="btn btn-primary w-100">เพิ่มข้อมูล</button>
      </div>
    </form>

    <table class="table table-bordered" id="dataTable">
      <thead class="table-light">
        <tr>
          <th>#</th>
          <th>อุปกรณ์</th>
          <th>ผู้ยืม</th>
          <th>วันที่ยืม</th>
          <th>วันที่คืน</th>
          <th>จัดการ</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>

    <button class="btn btn-success mt-2" onclick="exportToExcel()">ส่งออกเป็น Excel</button>
  </div>

  <script>
    const correctPassword = "admin123";

    function checkLogin() {
      const input = document.getElementById('passwordInput').value;
      if (input === correctPassword) {
        document.getElementById('loginPage').style.display = 'none';
        document.getElementById('mainApp').style.display = 'block';
      } else {
        document.getElementById('loginError').style.display = 'block';
      }
    }

    let data = [];
    let editIndex = null;

    const form = document.getElementById('form');
    const dataTable = document.querySelector('#dataTable tbody');

    form.addEventListener('submit', function (e) {
      e.preventDefault();

      const device = document.getElementById('device').value;
      const borrower = document.getElementById('borrower').value;
      const borrowDate = document.getElementById('borrowDate').value;
      const returnDate = document.getElementById('returnDate').value;

      const entry = { device, borrower, borrowDate, returnDate };

      if (editIndex !== null) {
        data[editIndex] = entry;
        editIndex = null;
      } else {
        data.push(entry);
      }

      form.reset();
      renderTable();
    });

    function renderTable() {
      dataTable.innerHTML = '';
      data.forEach((entry, index) => {
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${index + 1}</td>
          <td>${entry.device}</td>
          <td>${entry.borrower}</td>
          <td>${entry.borrowDate}</td>
          <td>${entry.returnDate}</td>
          <td>
            <button class="btn btn-warning btn-sm me-1" onclick="editEntry(${index})">แก้ไข</button>
            <button class="btn btn-danger btn-sm" onclick="deleteEntry(${index})">ลบ</button>
          </td>
        `;
        dataTable.appendChild(row);
      });
    }

    function editEntry(index) {
      const entry = data[index];
      document.getElementById('device').value = entry.device;
      document.getElementById('borrower').value = entry.borrower;
      document.getElementById('borrowDate').value = entry.borrowDate;
      document.getElementById('returnDate').value = entry.returnDate;
      editIndex = index;
    }

    function deleteEntry(index) {
      if (confirm('คุณแน่ใจหรือไม่ว่าต้องการลบข้อมูลนี้?')) {
        data.splice(index, 1);
        renderTable();
      }
    }

    function exportToExcel() {
      const ws = XLSX.utils.json_to_sheet(data);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "BorrowList");
      XLSX.writeFile(wb, "borrowed_equipment.xlsx");
    }
  </script>

</body>
</html>
