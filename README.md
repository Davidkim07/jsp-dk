<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>우리의 특별한 홈페이지 💖</title>
  <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.1/firebase-database-compat.js"></script>
  <style>
    /* 전반적인 배경 및 폰트 설정 */
    body { 
      margin: 0; padding: 0;
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #ffe3f1, #fff5e6);
      color: #333;
    }
    /* 네비게이션 바 */
    nav { 
      background: rgba(255,255,255,0.8);
      display: flex;
      justify-content: center;
      padding: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      position: sticky; top: 0;
      z-index: 100;
    }
    nav button {
      background: #fff;
      border: 2px solid #ff9fc3;
      border-radius: 25px;
      color: #ff4d6d;
      padding: 10px 20px;
      margin: 0 8px;
      font-size: 16px;
      cursor: pointer;
      transition: background 0.3s;
    }
    nav button:hover {
      background: #ff9fc3;
      color: #fff;
    }
    nav button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
    /* 콘텐츠 영역 */
    #content {
      max-width: 800px;
      margin: 30px auto;
      background: rgba(255,255,255,0.9);
      border-radius: 16px;
      padding: 20px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05);
    }
    h2, h3 {
      color: #ff4d6d;
      margin-bottom: 10px;
    }
    /* 계획과 규칙 박스 */
    .card {
      border: 1px solid #ffd1e6;
      border-radius: 12px;
      background: #fff;
      padding: 15px;
      margin-bottom: 20px;
      line-height: 1.6;
    }
    .day-title {
      font-weight: bold;
      color: #ff4d6d;
      margin-top: 15px;
    }
    .schedule-item {
      margin: 5px 0;
    }
    .rules-list {
      list-style: none;
      padding: 0;
    }
    .rules-list li {
      margin: 8px 0;
      padding-left: 20px;
      position: relative;
    }
    .rules-list li:before {
      content: "💗";
      position: absolute;
      left: 0;
    }
  </style>
</head>
<body>
  <nav>
    <button id="btn-my-plan">내 계획</button>
    <button id="btn-seoyeon-plan">서연 계획</button>
    <button id="btn-rules">💖 규칙</button>
    <button id="btn-attendance" disabled>✅ 출석체크</button>
    <button id="btn-survey">📝 설문조사</button>
  </nav>
  <div id="content">
    <h2>환영합니다!</h2>
    <p>메뉴에서 원하는 기능을 선택해주세요 💕</p>
  </div>

  <script>
    // Firebase 초기화 (본인 설정값 입력)
    const firebaseConfig = { /* YOUR FIREBASE CONFIG */ };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    const content = document.getElementById('content');
    const btnMyPlan = document.getElementById('btn-my-plan');
    const btnSeoyeonPlan = document.getElementById('btn-seoyeon-plan');
    const btnRules = document.getElementById('btn-rules');
    const btnAttendance = document.getElementById('btn-attendance');
    const btnSurvey = document.getElementById('btn-survey');

    let rulesAccepted = false;

    // 동현 계획 데이터
    const donghyunPlans = {
      '월, 수, 목, 금': [
        '7:00 - 스트레칭 후 말랭이에게 문자 보내기',
        '7:40 - 학교 갈 준비',
        '8:20 - 서연이와 연락 및 응원',
        'Key Point! 수업 후 자기 공부 열심히 하기',
        '16:30 - 학교 끝, 말랭이에게 문자',
        '17:00 - 말랭이에게 전화하기 (피곤할 경우 패스)',
        '18:50 - 식사 후 학원 준비',
        '19:00~22:00 - 학원 진도 2단원 (쉬는 시간에 서연이에게 전화)',
        '22:00~23:00 - 운동',
        '23:00 - 서연이와 전화하기',
        '23:20~02:00 - 스터디 카페에서 공부',
        '02:00~07:00 - 취침 (필요시 연락 유지)'
      ],
      '화요일': [
        '16:30 - 학교 끝, 말랭이에게 연락',
        '점심시간 - 말랭이 연락',
        '18:50~20:00 - 수학 공부',
        '20:00~21:00 - 식사 및 운동 준비',
        '21:00~23:00 - 운동',
        '23:00~01:00 - 스터디 카페 및 연락',
        '01:00~07:00 - 취침'
      ]
    };
    // 서연 계획 데이터
    const seoyeonPlans = {
      '월, 수, 목': [
        '05:40 - 스트레칭 후 오빠 연락 확인',
        '06:00 - 전화하며 학교 준비',
        '06:10 - 버스 탑승 및 오빠와 통화 (피곤 시 눈 붙이기)',
        '06:40 - 메이크업 및 아침',
        '07:10 - 수업 집중',
        '08:00 - 오빠에게 짧은 응원 연락',
        '09:00 - ROTC 전 전화',
        '09:53 - 불독 타임: 숙제/시험준비 우선',
        '11:40 - 런치: 숙제/시험준비, 오빠와 통화',
        '13:15 - 6교시 이동 시 연락',
        '13:20 - 7교시 AP 수업 집중',
        '14:10 - 수업 종료 & 버스 탑승',
        '14:30 - 오빠에게 학교 끝 연락',
        '14:50 - 집 도착 연락',
        '15:00 - 점심 및 옷 갈아입기',
        '15:30 - 낮잠 (불가 시 일기/숙제)',
        '15:40 - 줄넘기 & 샤워',
        '저녁 - 강의 듣기, 숙제, 가족 산책',
        '19:00 - 오빠에게 브리핑',
        '19:20 - 오빠 학교 배웅 후 자유시간',
        '23:00 - 취침 준비'
      ],
      '화, 금': [
        '05:40 - 스트레칭 후 오빠 연락 확인',
        '06:00 - 전화하며 학교 준비',
        '06:10 - 버스 탑승',
        '06:40 - 메이크업 및 아침',
        '07:10 - 수업 집중',
        '08:00 - 오빠 응원 연락',
        '09:00 - ROTC 전 전화',
        '09:53 - 불독 타임',
        '11:40 - 런치 & 통화',
        '13:15 - 6교시 이동 연락',
        '13:20 - 7교시 집중',
        '14:10 - 버스 탑승',
        '14:30 - 오빠에게 연락',
        '14:50 - 집 도착 연락',
        '15:00 - 점심',
        '16:00 - 낮잠',
        '18:20 - 기상 & 교회 준비',
        '18:40 - 통화 & 하루 브리핑',
        '19:20 - 교회 배웅',
        '22:00 - 교회 종료 후 휴식',
        '22:30 - 씻기 및 마무리',
        '23:00~24:00 - 남은 작업 완료'
      ]
    };
    // 규칙 데이터
    const rules = [
      '나쁜 말 하지 않기 (내가 듣는다고 생각하기)',
      '야, 니, 너 대신 이름 부르기',
      '존댓말 사용하기',
      '말 다 듣고 나서 말하기',
      '싸움 후 15분은 "알겠어"만 말하기',
      '싸움 회피 금지 (서로 이해 과정)',
      '"헤어지자" 절대 금지',
      '화해 손길은 무조건 받아주기',
      '"사랑해"로 마무리하기',
      '밤샘 시 배려하기',
      '건강 우선! 피곤하면 쉬기',
      '이성 친구 사적 대화 금지',
      '표현 자주 많이 하기',
      '하루 브리핑 필수',
      '사랑하는 마음 잊지 않기 ❤️'
    ];

    // 렌더링 함수들
    function renderPlan(plans) {
      let html = '<h2>오늘의 계획</h2>';
      Object.entries(plans).forEach(([day, list]) => {
        html += `<div class="card"><div class="day-title">${day}</div>`;
        list.forEach(item => html += `<div class="schedule-item">• ${item}</div>`);
        html += '</div>';
      });
      content.innerHTML = html;
    }
    function renderRules() {
      let html = '<h2>💖 우리 사랑 지키는 규칙</h2><ul class="rules-list">';
      rules.forEach(r => html += `<li>${r}</li>`);
      html += '</ul><div style="text-align:center;"><button id="btn-read">다 읽었음</button></div>';
      content.innerHTML = html;
      document.getElementById('btn-read').onclick = () => {
        rulesAccepted = true;
        btnAttendance.disabled = false;
        alert('규칙 확인 완료! 출석체크가 활성화되었습니다.');
      };
    }
    function renderAttendance() {
      if (!rulesAccepted) { alert('먼저 규칙을 읽어주세요.'); return; }
      // 출석체크 뷰는 이전 코드 재사용
      content.innerHTML = '<h2>✅ 출석 체크</h2><button id="btn-check">출석하기</button><div id="calendar" class="calendar" style="margin-top:20px;"></div>';
      document.getElementById('btn-check').onclick = () => {
        const name = prompt('이름을 입력해주세요 (김동현 혹은 박서연)');
        const date = new Date().toISOString().split('T')[0];
        db.ref('attendance/' + date).push(name);
      };
      renderCalendar();
      db.ref('attendance').on('value', renderCalendar);
    }
    function renderSurvey() {
      // ...설문조사 기존 코드
      let html = '<h2>📝 설문조사</h2><div id="emojis">';
      ['😄','😊','😐','😞','😡'].forEach(e => html += `<button class="emo">${e}</button>`);
      html += '</div><div id="survey-input" style="margin-top:15px;"></div>';
      content.innerHTML = html;
      document.querySelectorAll('.emo').forEach(btn => btn.onclick = () => {
        const sel = btn.innerText;
        document.getElementById('survey-input').innerHTML = `<h3>${sel} 이유</h3><textarea id="resp" rows="3" style="width:100%;border:1px solid #ddd;border-radius:8px;padding:8px;"></textarea><br><button id="save">저장</button>`;
        document.getElementById('save').onclick = () => {
          const txt = document.getElementById('resp').value;
          db.ref('survey').push({emoji:sel,text:txt});
          alert('설문이 저장되었습니다. 감사합니다!');
        };
      });
    }
    // 이벤트 바인딩
    btnMyPlan.onclick = () => renderPlan(donghyunPlans);
    btnSeoyeonPlan.onclick = () => renderPlan(seoyeonPlans);
    btnRules.onclick = renderRules;
    btnAttendance.onclick = renderAttendance;
    btnSurvey.onclick = renderSurvey;
  </script>
</body>
</html>
