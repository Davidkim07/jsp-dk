<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>출석체크 앱</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 30px;
    }
    button {
      padding: 12px 20px;
      font-size: 16px;
      margin: 10px;
    }
    #status {
      font-size: 18px;
      margin-top: 15px;
    }
    .calendar {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 5px;
      max-width: 600px;
      margin: 30px auto;
    }
    .day {
      border: 1px solid #ccc;
      padding: 8px;
      height: 80px;
      font-size: 14px;
      position: relative;
      background-color: #fff;
    }
    .today {
      background-color: #e0f0ff;
    }
    .checked {
      background-color: #f0fff0;
    }
    .name-list {
      margin-top: 5px;
      font-size: 13px;
    }
    .name-tag {
      display: inline-block;
      padding: 2px 6px;
      border-radius: 10px;
      margin: 1px 2px;
      font-size: 12px;
      color: white;
    }
    .서연 {
      background-color: hotpink;
    }
    .동현 {
      background-color: royalblue;
    }
  </style>
</head>
<body>
  <h1>📅 출석체크</h1>

  <div>
    <button onclick="checkAttendance('서연')">서연 출석하기</button>
    <button onclick="checkAttendance('동현')">동현 출석하기</button>
  </div>

  <div id="status"></div>

  <h2>이번 달 출석 달력</h2>
  <div id="calendar" class="calendar"></div>

  <h3>출석 일수 통계</h3>
  <div id="attendance-stats">
    <p>서연: <span id="seoyeon-attendance">0</span>일</p>
    <p>동현: <span id="donghyun-attendance">0</span>일</p>
  </div>

  <div>
    <button onclick="changeMonth(-1)">이전 달</button>
    <button onclick="changeMonth(1)">다음 달</button>
  </div>

  <script>
    let currentMonth = new Date().getMonth(); // 현재 월
    let currentYear = new Date().getFullYear(); // 현재 연도
    const todayStr = new Date().toISOString().split('T')[0]; // 오늘 날짜

    function getAttendanceData(name) {
      return JSON.parse(localStorage.getItem(`attendance_${name}_${currentYear}-${currentMonth + 1}`)) || [];
    }

    function saveAttendanceData(name, data) {
      localStorage.setItem(`attendance_${name}_${currentYear}-${currentMonth + 1}`, JSON.stringify(data));
    }

    function checkAttendance(name) {
      const data = getAttendanceData(name);
      if (data.includes(todayStr)) {
        document.getElementById('status').innerText = `✅ ${name}님은 이미 출석했어요!`;
        return;
      }

      data.push(todayStr);
      saveAttendanceData(name, [...new Set(data)]);
      document.getElementById('status').innerText = `🎉 ${name}님 출석 완료!`;

      updateCalendarCell(todayStr, name);
      updateAttendanceStats();
    }

    function updateCalendarCell(dateStr, name) {
      const cell = document.querySelector(`[data-date='${dateStr}']`);
      if (!cell) return;

      cell.classList.add('checked');

      const nameList = cell.querySelector('.name-list');
      const existingTags = [...nameList.querySelectorAll('.name-tag')].map(el => el.innerText);

      if (!existingTags.includes(name)) {
        const tag = document.createElement('span');
        tag.className = `name-tag ${name}`;
        tag.innerText = name;
        nameList.appendChild(tag);
      }
    }

    function renderCalendar() {
      const calendar = document.getElementById('calendar');
      calendar.innerHTML = '';

      const firstDay = new Date(currentYear, currentMonth, 1);
      const lastDay = new Date(currentYear, currentMonth + 1, 0);
      const startDay = firstDay.getDay();
      const totalDays = lastDay.getDate();

      for (let i = 0; i < startDay; i++) {
        const empty = document.createElement('div');
        calendar.appendChild(empty);
      }

      for (let day = 1; day <= totalDays; day++) {
        const date = new Date(currentYear, currentMonth, day);
        const dateStr = date.toISOString().split('T')[0];

        const cell = document.createElement('div');
        cell.className = 'day';
        cell.setAttribute('data-date', dateStr);

        if (dateStr === todayStr) cell.classList.add('today');

        cell.innerHTML = `
          <div>${day}</div>
          <div class="name-list"></div>
        `;

        calendar.appendChild(cell);
      }
    }

    function renderAllAttendance() {
      ['서연', '동현'].forEach(name => {
        const data = getAttendanceData(name);
        data.forEach(dateStr => {
          updateCalendarCell(dateStr, name);
        });
      });
    }

    function updateAttendanceStats() {
      let seoyeonData = getAttendanceData('서연');
      let donghyunData = getAttendanceData('동현');
      document.getElementById('seoyeon-attendance').innerText = seoyeonData.length;
      document.getElementById('donghyun-attendance').innerText = donghyunData.length;
    }

    function changeMonth(direction) {
      const currentDate = new Date();
      const currentDay = currentDate.getDate();
      const currentMonthEndDate = new Date(currentYear, currentMonth + 1, 0).getDate();

      // 다음 달 첫날이 되기 전날에만 초기화
      if (currentDay === currentMonthEndDate) {
        clearAttendanceData();
      }

      currentMonth += direction;

      if (currentMonth > 11) {
        currentMonth = 0;
        currentYear++;
      } else if (currentMonth < 0) {
        currentMonth = 11;
        currentYear--;
      }

      renderCalendar();
      renderAllAttendance();
      updateAttendanceStats();
    }

    function clearAttendanceData() {
      // 이전 달의 출석 데이터를 초기화
      localStorage.removeItem(`attendance_서연_${currentYear}-${currentMonth}`);
      localStorage.removeItem(`attendance_동현_${currentYear}-${currentMonth}`);
    }

    window.onload = () => {
      renderCalendar();
      renderAllAttendance();
      updateAttendanceStats();
    };
  </script>
</body>
</html>
