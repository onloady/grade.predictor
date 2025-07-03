<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>1학기 기말고사 내신 등급컷 예측기</title>
  <style>
    * { box-sizing: border-box; }
    body {
      font-family: 'Noto Sans KR', 'Malgun Gothic', Arial, sans-serif;
      background-color: #F2F2F2;
      padding: 20px;
      max-width: 480px;
      margin: 0 auto;
      color: #333;
      line-height: 1.5;
    }
    h2 {
      text-align: center;
      margin-bottom: 25px;
      font-weight: 700;
      color: #333;
    }
    label {
      font-weight: 600;
      color: #333;
      display: block;
      margin-bottom: 8px;
    }
    select, input[type="text"] {
      width: 100%;
      padding: 12px 15px;
      font-size: 1.1em;
      border: 1.8px solid #cbd2e0;
      border-radius: 6px;
      transition: border-color 0.3s ease;
      outline: none;
    }
    select:focus, input[type="text"]:focus {
      border-color: #5577ff;
      box-shadow: 0 0 8px rgba(85, 119, 255, 0.3);
    }
    button {
      width: 100%;
      padding: 14px 0;
      margin-top: 12px;
      font-size: 1.1em;
      font-weight: 600;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      transition: background-color 0.25s ease;
      color: white;
      user-select: none;
    }
    #submitBtn { background-color: #658cf6; }
    #submitBtn:hover:not(:disabled) { background-color: #aab8ff; }
    #submitBtn:disabled {
      background-color: #aab8ff;
      cursor: not-allowed;
    }
    #cancelBtn {
      background-color: #f06074;
      margin-top: 8px;
      display: none;
    }
    #cancelBtn:hover { background-color: #ffa1a8; }
    #cancelBtn:disabled {
      background-color: #f7a6a6;
      cursor: not-allowed;
    }
    #scoreInput:disabled {
      background-color: #eee;
      color: #777;
    }
    #gradeResult {
      margin-top: 30px;
      padding: 20px;
      background: white;
      border-radius: 12px;
      box-shadow: 0 2px 10px rgb(0 0 0 / 0.1);
      color: #333;
    }
    #gradeResult h3 {
      margin-bottom: 15px;
      font-weight: 700;
      border-bottom: 2px solid #658cf6;
      padding-bottom: 8px;
    }
    .trust-stars {
      font-family: 'Segoe UI Symbol', Arial, sans-serif;
      font-size: 1.6em;
      letter-spacing: 6px;
      display: inline-block;
      vertical-align: middle;
      color: #666;
    }
    .trust-stars span {
      display: inline-block;
      width: 1em;
      text-align: center;
      color: #666;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }
    th, td {
      padding: 8px;
      border-bottom: 1px solid #ccc;
      border-right: 1px solid #ccc;
      text-align: left;
    }
    td:last-child, th:last-child { border-right: none; }
    thead { background-color: #f0f2f5; }
    .trust-section {
      margin-top: 20px;
      font-size: 0.95em;
      color: #555;
    }
    .trust-stars-section {
      font-weight: 600;
      margin-bottom: 8px;
    }
    .trust-divider {
      border-top: 1px solid #ddd;
      margin: 8px 0;
    }
    .trust-criteria {
      font-size: 0.85em;
      color: #666;
      line-height: 1.4;
      white-space: pre-line;
    }
    /* 카카오톡 채널 추가 영역 흰 배경 박스 스타일 */
    #kakaoChannel {
      margin-top: 20px;
      background: white;
      border-radius: 12px;
      padding: 20px 16px;
      box-shadow: 0 2px 12px rgba(0,0,0,0.1);
      text-align: center;
      color: #333;
    }
    #kakaoChannel p {
      font-weight: 700;
      margin-bottom: 12px;
      font-size: 1.05em;
    }
    #kakaoChannel button {
      background-color: #FEE500;
      color: #000;
      border: none;
      padding: 12px 20px;
      border-radius: 8px;
      font-weight: 600;
      font-size: 1em;
      cursor: pointer;
      user-select: none;
      transition: background-color 0.3s ease;
    }
    #kakaoChannel button:hover {
      background-color: #fdd835;
    }
    @media (max-width: 400px) {
      body { padding: 15px 12px; }
      select, input[type="text"], button {
        font-size: 1em;
        padding: 10px 12px;
      }
    }
  </style>
</head>
<body>
  <h2>1학기 기말고사 내신 등급컷 예측기</h2>

  <label for="subjectSelect"></label>
  <select id="subjectSelect" onchange="refreshStatusAndGrades()">
    <option value="" disabled selected>과목 선택</option>
    <option value="국어">국어</option>
    <option value="영어">영어</option>
    <option value="수학">수학</option>
    <option value="사회">사회</option>
    <option value="과학">과학</option>
    <option value="한국사">한국사</option>
  </select>

  <input type="text" id="scoreInput" placeholder="점수 입력( 정확한 점수를 입력해주세요.)" disabled />
  <button id="submitBtn" onclick="addScore()" disabled>제출</button>
  <button id="cancelBtn" onclick="cancelLatestScore()">제출 취소</button>

  <div id="gradeResult"></div>

  <div id="kakaoChannel" style="display:none;">
    <p>예상 등급컷 제공 알림을 받고 싶다면?</p>
    <button onclick="window.open('http://pf.kakao.com/_DxeZFn', '_blank')">
      💬 카카오톡 채널 추가하기
    </button>
  </div>

  <script>
    const data = {
      국어: {}, 영어: {}, 수학: {}, 사회: {}, 과학: {}, 한국사: {}
    };
    const submissionHistory = {
      국어: [], 영어: [], 수학: [], 사회: [], 과학: [], 한국사: []
    };

    function getSelectedSubject() {
      return document.getElementById('subjectSelect').value;
    }

    function validateScore(rawValue) {
      if (/^0\d/.test(rawValue)) return false;
      const num = Number(rawValue);
      if (isNaN(num) || num < 3 || num > 100) return false;
      if (rawValue.includes('.')) {
        const parts = rawValue.split('.');
        if (parts[1].length > 1) return false;
      }
      return true;
    }

    function addScore() {
  const subject = getSelectedSubject();
  const input = document.getElementById('scoreInput');
  const rawValue = input.value.trim();

  if (!validateScore(rawValue)) {
    alert('소수점 첫째 자리까지를 포함한 3~100까지의 숫자만 입력할 수 있습니다.');
    return;
  }

  if (localStorage.getItem('submitted_' + subject) === 'true') {
    alert('이미 제출하셨습니다. 중복 제출은 불가합니다.');
    return;
  }

  // 👉 최대 320명 제한 조건 추가
  if (Object.keys(data[subject]).length >= 320) {
    alert('해당 과목은 이미 320명 이상의 데이터가 수집되어 더 이상 입력할 수 없습니다.');
    return;
  }

  const score = Number(rawValue);
  const newKey = 'user_' + Date.now() + Math.random().toString(36).slice(2);
  data[subject][newKey] = score;
  submissionHistory[subject].push(newKey);
  input.value = '';

  localStorage.setItem('submitted_' + subject, 'true');
  document.getElementById('submitBtn').textContent = '제출 완료';

  refreshStatusAndGrades();
}

    function cancelLatestScore() {
      const subject = getSelectedSubject();
      const history = submissionHistory[subject];
      if (history.length === 0) {
        alert('취소할 제출 기록이 없습니다.');
        return;
      }
      const lastKey = history.pop();
      delete data[subject][lastKey];

      if (history.length === 0) {
        localStorage.removeItem('submitted_' + subject);
      }

      refreshStatusAndGrades();
    }

    function refreshStatusAndGrades() {
      const subject = getSelectedSubject();
      const input = document.getElementById('scoreInput');
      const submitBtn = document.getElementById('submitBtn');
      const kakaoDiv = document.getElementById('kakaoChannel');

      const isValid = subject !== '';
      const isSubmitted = localStorage.getItem('submitted_' + subject) === 'true';

      input.disabled = !isValid || isSubmitted;
      submitBtn.disabled = !isValid || isSubmitted;
      submitBtn.textContent = isSubmitted ? '제출 완료' : '제출';

      if (!isValid) {
        document.getElementById('gradeResult').innerHTML = `<p>과목을 먼저 선택하세요.</p>`;
        kakaoDiv.style.display = 'none';
        document.getElementById('cancelBtn').style.display = 'none';
        return;
      }

      calculateGradeCuts(subject);
      updateCancelButton();

      // 항상 카카오톡 채널 추가 영역 보이기
      kakaoDiv.style.display = 'block';
    }

    function calculateGradeCuts(subject) {
      const subjectData = data[subject];
      const scores = Object.values(subjectData);
      const total = scores.length;
      const gradeResult = document.getElementById('gradeResult');

      if (total < 30) {
        gradeResult.innerHTML = `<div class="subjectBlock">
          <h3>[${subject}] 데이터 수집 중 (현재 ${total}명의 데이터 수집)</h3>
          <p>30명 이상의 데이터 수집 후 실시간 예상 등급컷 제공됩니다.</p>
        </div>`;
        return;
      }

      const sorted = [...scores].sort((a, b) => b - a);
      const cuts = [
        Math.floor(total * 0.10),
        Math.floor(total * 0.34),
        Math.floor(total * 0.66),
        Math.floor(total * 0.90),
        total
      ];
      const gradeCuts = cuts.map(i => sorted[Math.max(0, i - 1)] ?? NaN);

      let trust = '';
      if (total >= 90) trust = '★★★★☆';
      else if (total >= 60) trust = '★★★✮☆';
      else if (total >= 30) trust = '★★★☆☆';

      function renderTrustStars(stars) {
        return stars.split('').map(ch => `<span>${ch}</span>`).join('');
      }

      let html = `<div class="subjectBlock">
        <h3>[${subject}] 예상 등급컷 (현재 ${total}명의 데이터 수집)</h3>
        <table>
          <thead><tr><th>등급</th><th>예상 등급컷</th></tr></thead>
          <tbody>
            <tr><td>1등급</td><td>${gradeCuts[0].toFixed(1)}</td></tr>
            <tr><td>2등급</td><td>${gradeCuts[1].toFixed(1)}</td></tr>
            <tr><td>3등급</td><td>${gradeCuts[2].toFixed(1)}</td></tr>
            <tr><td>4등급</td><td>${gradeCuts[3].toFixed(1)}</td></tr>
          </tbody>
        </table>`;

      html += `<div class="trust-section">
        <div class="trust-stars-section">신뢰도 ${renderTrustStars(trust)}</div>
        <div class="trust-divider"></div>
        <div class="trust-criteria">신뢰도 기준<br/>
          ★★★☆☆ : 30명 이상의 데이터 수집<br/>
          ★★★✮☆ : 60명 이상의 데이터 수집<br/>
          ★★★★☆ : 90명 이상의 데이터 수집</div>
        </div></div>`;

      gradeResult.innerHTML = html;
    }

    function updateCancelButton() {
      const subject = getSelectedSubject();
      const cancelBtn = document.getElementById('cancelBtn');
      cancelBtn.style.display = submissionHistory[subject].length > 0 ? 'block' : 'none';
    }

    window.onload = function () {
      refreshStatusAndGrades();
    };

  </script>
</body>
</html>
