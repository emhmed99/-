# -<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>تقويم العمل</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f2f2f2;
      direction: rtl;
      text-align: center;
    }

    .calendar {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 5px;
      margin-top: 20px;
    }

    .day {
      background: #fff;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 5px;
      cursor: pointer;
    }

    .day:hover {
      background-color: #e0e0e0;
    }

    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin: 20px;
    }

    .result {
      margin-top: 20px;
      background: #fff;
      padding: 15px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }

    .modal {
      display: none;
      position: fixed;
      z-index: 999;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      overflow: auto;
      background: rgba(0,0,0,0.4);
    }

    .modal-content {
      background: white;
      margin: 15% auto;
      padding: 20px;
      border-radius: 10px;
      width: 300px;
    }

    input[type="time"] {
      margin: 5px 0;
    }

    .close {
      float: left;
      font-size: 18px;
      cursor: pointer;
      color: red;
    }
  </style>
</head>
<body>
  <div class="header">
    <button onclick="changeMonth(-1)">الشهر السابق</button>
    <h2 id="monthTitle"></h2>
    <button onclick="changeMonth(1)">الشهر التالي</button>
  </div>

  <label>سعر الساعة (دينار): <input type="number" id="hourRate" value="10"></label>
  <div class="calendar" id="calendar"></div>
  <button onclick="calculateSalary()">احسب الراتب</button>
  <div class="result" id="result"></div>

  <!-- Modal -->
  <div class="modal" id="modal">
    <div class="modal-content">
      <span class="close" onclick="closeModal()">&times;</span>
      <h3 id="modalDate"></h3>
      بداية: <input type="time" id="startTime"><br>
      نهاية: <input type="time" id="endTime"><br>
      <button onclick="saveTime()">حفظ</button>
    </div>
  </div>

  <script>
    const calendar = document.getElementById("calendar");
    const monthTitle = document.getElementById("monthTitle");
    const modal = document.getElementById("modal");
    const modalDate = document.getElementById("modalDate");
    const startTimeInput = document.getElementById("startTime");
    const endTimeInput = document.getElementById("endTime");

    let currentDate = new Date(2025, 3); // يبدأ من أبريل 2025
    let selectedDate = null;
    const workTimes = {};

    // تحميل البيانات من Local Storage
    const saved = localStorage.getItem("workTimes");
    if (saved) {
      Object.assign(workTimes, JSON.parse(saved));
    }

    function renderCalendar() {
      calendar.innerHTML = '';

      const year = currentDate.getFullYear();
      const month = currentDate.getMonth();

      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();

      monthTitle.textContent = currentDate.toLocaleDateString('ar-EG', { year: 'numeric', month: 'long' });

      // عرض أسماء أيام الأسبوع
      const weekDays = ['السبت', 'الأحد', 'الإثنين', 'الثلاثاء', 'الأربعاء', 'الخميس', 'الجمعة'];
      for (let i = 0; i < 7; i++) {
        calendar.innerHTML += `<div style="font-weight: bold; background:#ddd;">${weekDays[i]}</div>`;
      }

      // حساب الفراغات قبل بداية الشهر
      const offset = (firstDay === 0 ? 6 : firstDay - 1);
      for (let i = 0; i < offset; i++) {
        calendar.innerHTML += `<div></div>`;
      }

      // إنشاء الأيام
      for (let day = 1; day <= daysInMonth; day++) {
        const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
        const hasTime = workTimes[dateStr] ? "✔️" : "";
        calendar.innerHTML += `
          <div class="day" onclick="openModal('${dateStr}')">
            ${day} ${hasTime}
          </div>
        `;
      }
    }

    function changeMonth(dir) {
      currentDate.setMonth(currentDate.getMonth() + dir);
      if (currentDate < new Date(2025, 3)) currentDate = new Date(2025, 3);
      if (currentDate > new Date(2025, 11)) currentDate = new Date(2025, 11);
      renderCalendar();
    }

    function openModal(date) {
      selectedDate = date;
      modalDate.textContent = `اليوم: ${date}`;
      startTimeInput.value = workTimes[date]?.start || '';
      endTimeInput.value = workTimes[date]?.end || '';
      modal.style.display = "block";
    }

    function closeModal() {
      modal.style.display = "none";
    }

    function saveTime() {
      const start = startTimeInput.value;
      const end = endTimeInput.value;
      if (start && end && end > start) {
        workTimes[selectedDate] = { start, end };
      } else {
        delete workTimes[selectedDate];
      }

      // حفظ في Local Storage
      localStorage.setItem("workTimes", JSON.stringify(workTimes));

      closeModal();
      renderCalendar();
    }

    function timeToHours(timeStr) {
      const [h, m] = timeStr.split(":").map(Number);
      return h + m / 60;
    }

    function calculateSalary() {
      const rate = parseFloat(document.getElementById("hourRate").value);
      let totalHours = 0;
      for (const date in workTimes) {
        const start = timeToHours(workTimes[date].start);
        const end = timeToHours(workTimes[date].end);
        if (end > start) totalHours += end - start;
      }
      const salary = totalHours * rate;
      document.getElementById("result").innerHTML = `
        <p>إجمالي الساعات: ${totalHours.toFixed(2)} ساعة</p>
        <p>الراتب الإجمالي: ${salary.toFixed(2)} دينار</p>
      `;
    }

    renderCalendar();
  </script>
</body>
</html>
يحسب ساعات العمل
