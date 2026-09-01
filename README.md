<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>情绪天气 · Mood Weather</title>
<style>
  :root {
    --bg: #faf8f5;
    --text: #2d2d2d;
    --text-secondary: #6b6b6b;
    --text-tertiary: #999;
    --border: #e0dcd6;
    --card: #ffffff;
    --card-hover: #f5f2ee;
  }
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    overflow-x: hidden;
    transition: background 1.2s ease;
  }
  #weather-canvas {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    pointer-events: none;
    z-index: 0;
  }
  .weather-overlay {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    pointer-events: none;
    z-index: 0;
    transition: opacity 1.2s ease;
    opacity: 0;
  }
  .weather-overlay.active { opacity: 1; }
  .sun-particle {
    position: absolute;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(255,210,120,0.5) 0%, transparent 70%);
    animation: floatUp 12s infinite ease-in-out;
  }
  @keyframes floatUp {
    0%, 100% { transform: translateY(100vh) scale(0); opacity: 0; }
    15% { opacity: 0.6; }
    85% { opacity: 0.6; }
    100% { transform: translateY(-10vh) scale(1); opacity: 0; }
  }
  .bg-sunny { background: linear-gradient(180deg, #fdf8f0 0%, #faf5ec 100%) !important; }
  .bg-cloudy { background: linear-gradient(180deg, #f0f2f5 0%, #eceef2 100%) !important; }
  .bg-rainy { background: linear-gradient(180deg, #e8e6e2 0%, #e2e0dc 100%) !important; }
  .bg-stormy { background: linear-gradient(180deg, #dddcda 0%, #d5d4d2 100%) !important; }
  .bg-snowy { background: linear-gradient(180deg, #eef2f5 0%, #e8ecf0 100%) !important; }
  .bg-rainbow { background: linear-gradient(180deg, #faf8f5 0%, #f5f2ee 100%) !important; }
  .overlay-sunny { background: radial-gradient(ellipse at 30% 20%, rgba(255,220,150,0.15) 0%, transparent 60%); }
  .overlay-cloudy { background: linear-gradient(180deg, rgba(200,210,220,0.25) 0%, transparent 50%); }
  .overlay-rainbow {
    background: linear-gradient(135deg,
      rgba(255,200,200,0.1) 0%, rgba(255,220,180,0.1) 20%,
      rgba(255,255,200,0.1) 40%, rgba(200,255,200,0.1) 60%,
      rgba(200,220,255,0.1) 80%, rgba(220,200,255,0.1) 100%);
  }
  .app-container {
    position: relative;
    z-index: 1;
    max-width: 520px;
    margin: 0 auto;
    padding: 16px 14px 50px;
    min-height: 100vh;
  }
  .page { display: none; animation: fadeIn 0.4s ease; }
  .page.active { display: block; }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
  .nav-bar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 14px;
    padding: 2px 0;
  }
  .nav-back, .nav-action {
    width: 36px; height: 36px;
    border-radius: 10px;
    border: 1px solid var(--border);
    background: var(--card);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 16px;
    color: var(--text-secondary);
    transition: all 0.2s ease;
    flex-shrink: 0;
  }
  .nav-back:hover, .nav-action:hover { border-color: var(--text); color: var(--text); }
  .nav-title { font-size: 17px; font-weight: 500; text-align: center; flex: 1; }
  .nav-spacer { width: 36px; flex-shrink: 0; }
  .home-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 80vh;
    text-align: center;
  }
  .home-emoji { font-size: 64px; margin-bottom: 20px; animation: gentleBounce 3s infinite ease-in-out; }
  @keyframes gentleBounce { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-6px); } }
  .home-title { font-size: 26px; font-weight: 500; margin-bottom: 6px; letter-spacing: 2px; }
  .home-sub { font-size: 14px; color: var(--text-secondary); margin-bottom: 36px; line-height: 1.7; }
  .home-btn {
    padding: 14px 44px;
    border-radius: 30px;
    border: 1.5px solid var(--text);
    background: var(--card);
    color: var(--text);
    font-size: 15px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.3s ease;
    font-family: inherit;
    letter-spacing: 1px;
  }
  .home-btn:hover { background: var(--text); color: var(--card); transform: translateY(-2px); box-shadow: 0 4px 16px rgba(0,0,0,0.1); }
  .calendar-header { text-align: center; margin-bottom: 14px; }
  .calendar-header h2 { font-size: 18px; font-weight: 500; margin-bottom: 2px; }
  .calendar-header p { font-size: 13px; color: var(--text-tertiary); }
  .month-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding: 0 2px;
  }
  .month-nav button {
    width: 32px; height: 32px;
    border-radius: 8px;
    border: 1px solid var(--border);
    background: var(--card);
    cursor: pointer;
    font-size: 16px;
    color: var(--text-secondary);
    transition: all 0.2s ease;
  }
  .month-nav button:hover { border-color: var(--text); color: var(--text); }
  .month-label { font-size: 16px; font-weight: 500; }
  .weekdays { display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px; margin-bottom: 4px; }
  .weekday { text-align: center; font-size: 12px; color: var(--text-tertiary); padding: 5px 0; }
  .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 5px; }
  .day-cell {
    aspect-ratio: 1;
    border-radius: 12px;
    border: 1.5px solid transparent;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    position: relative;
    transition: all 0.25s ease;
    background: var(--card);
    box-shadow: 0 1px 3px rgba(0,0,0,0.04);
  }
  .day-cell:hover { border-color: var(--border); transform: translateY(-2px); box-shadow: 0 4px 12px rgba(0,0,0,0.06); }
  .day-cell.other-month { opacity: 0.18; pointer-events: none; }
  .day-cell.today { border-color: #c9a87c; }
  .day-cell.has-mood { background: var(--card-hover); }
  .day-num { font-size: 12px; color: var(--text-secondary); line-height: 1; font-weight: 500; }
  .day-cell.today .day-num { color: #c9a87c; }
  .day-mood { font-size: 16px; line-height: 1; margin-top: 1px; }
  .day-dot { width: 4px; height: 4px; border-radius: 50%; margin-top: 2px; }
  .calendar-actions {
    display: flex;
    gap: 8px;
    margin-top: 14px;
  }
  .calendar-actions button {
    flex: 1;
    padding: 10px;
    border-radius: 10px;
    border: 1px solid var(--border);
    background: var(--card);
    font-size: 13px;
    font-family: inherit;
    cursor: pointer;
    transition: all 0.2s ease;
    color: var(--text-secondary);
  }
  .calendar-actions button:hover { border-color: var(--text); color: var(--text); }
  .mood-page-bg { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; pointer-events: none; transition: all 1s ease; }
  .mood-page { position: relative; z-index: 1; }
  .mood-date { text-align: center; margin-bottom: 22px; }
  .mood-date h2 { font-size: 20px; font-weight: 500; margin-bottom: 4px; }
  .mood-date p { font-size: 13px; color: var(--text-secondary); }
  .mood-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 18px; }
  .mood-card {
    border: 2px solid var(--border);
    border-radius: 16px;
    padding: 18px 6px;
    text-align: center;
    cursor: pointer;
    background: var(--card);
    transition: all 0.3s ease;
  }
  .mood-card:hover { transform: translateY(-3px); box-shadow: 0 6px 20px rgba(0,0,0,0.08); border-color: var(--text); }
  .mood-card.selected { border-color: var(--text); transform: scale(1.02); box-shadow: 0 6px 20px rgba(0,0,0,0.1); }
  .mood-emoji { font-size: 36px; line-height: 1; margin-bottom: 6px; display: block; }
  .mood-label { font-size: 13px; color: var(--text-secondary); font-weight: 500; }
  .mood-quote {
    text-align: center;
    font-size: 14px;
    color: var(--text-secondary);
    font-style: italic;
    line-height: 1.7;
    margin-bottom: 14px;
    padding: 14px;
    border-radius: 12px;
    background: rgba(255,255,255,0.5);
    min-height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .mood-note-area { margin-bottom: 14px; }
  .mood-note-area textarea {
    width: 100%;
    border: 1.5px solid var(--border);
    border-radius: 14px;
    padding: 12px 14px;
    font-size: 14px;
    font-family: inherit;
    resize: none;
    outline: none;
    background: var(--card);
    color: var(--text);
    line-height: 1.6;
    transition: border-color 0.2s ease;
  }
  .mood-note-area textarea:focus { border-color: var(--text); }
  .mood-actions { display: flex; gap: 10px; }
  .mood-actions button {
    flex: 1;
    padding: 12px;
    border-radius: 12px;
    border: 1.5px solid var(--border);
    background: var(--card);
    font-size: 14px;
    font-family: inherit;
    cursor: pointer;
    transition: all 0.2s ease;
    font-weight: 500;
    color: var(--text-secondary);
  }
  .mood-actions button:hover { border-color: var(--text); color: var(--text); }
  .mood-actions .btn-primary { background: var(--text); color: var(--card); border-color: var(--text); }
  .mood-actions .btn-danger { border-color: #d4a5a5; color: #b07878; }
  .mood-actions .btn-danger:hover { background: #fff0f0; }
  .detail-modal {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.3);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 100;
    backdrop-filter: blur(4px);
    padding: 20px;
  }
  .detail-modal.show { display: flex; }
  .detail-content {
    background: var(--card);
    border-radius: 20px;
    padding: 24px;
    width: 100%;
    max-width: 360px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.15);
    animation: modalIn 0.3s ease;
  }
  @keyframes modalIn { from { opacity: 0; transform: scale(0.95) translateY(10px); } to { opacity: 1; transform: scale(1) translateY(0); } }
  .detail-header { display: flex; align-items: center; gap: 12px; margin-bottom: 14px; }
  .detail-emoji { font-size: 32px; }
  .detail-title { font-size: 17px; font-weight: 500; }
  .detail-date { font-size: 12px; color: var(--text-tertiary); }
  .detail-note { font-size: 14px; color: var(--text-secondary); line-height: 1.7; padding: 12px; background: var(--bg); border-radius: 10px; margin-bottom: 14px; min-height: 50px; }
  .detail-actions { display: flex; gap: 8px; }
  .detail-actions button {
    flex: 1;
    padding: 10px;
    border-radius: 10px;
    border: 1px solid var(--border);
    background: transparent;
    font-size: 14px;
    font-family: inherit;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  .detail-actions .btn-primary { background: var(--text); color: var(--card); border-color: var(--text); }
  .report-section { margin-bottom: 22px; }
  .report-section h3 { font-size: 16px; font-weight: 500; margin-bottom: 10px; display: flex; align-items: center; gap: 6px; }
  .report-card {
    background: var(--card);
    border-radius: 16px;
    padding: 18px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.04);
    border: 1px solid var(--border);
  }
  .mood-bars { display: flex; flex-direction: column; gap: 10px; }
  .mood-bar-row { display: flex; align-items: center; gap: 10px; }
  .mood-bar-emoji { font-size: 20px; width: 28px; text-align: center; }
  .mood-bar-label { font-size: 13px; color: var(--text-secondary); width: 40px; }
  .mood-bar-track { flex: 1; height: 8px; background: var(--bg); border-radius: 4px; overflow: hidden; }
  .mood-bar-fill { height: 100%; border-radius: 4px; transition: width 0.8s ease; }
  .mood-bar-count { font-size: 12px; color: var(--text-tertiary); width: 24px; text-align: right; }
  .dominant-mood { text-align: center; padding: 18px; background: var(--bg); border-radius: 14px; margin-top: 12px; }
  .dominant-mood .big-emoji { font-size: 44px; line-height: 1; margin-bottom: 8px; }
  .dominant-mood .big-label { font-size: 16px; font-weight: 500; margin-bottom: 4px; }
  .dominant-mood .big-desc { font-size: 13px; color: var(--text-secondary); line-height: 1.6; }
  .curve-container { width: 100%; height: 180px; position: relative; }
  .curve-svg { width: 100%; height: 100%; }
  .curve-tooltip {
    position: absolute;
    background: var(--text);
    color: var(--card);
    padding: 4px 10px;
    border-radius: 6px;
    font-size: 11px;
    pointer-events: none;
    opacity: 0;
    transition: opacity 0.2s ease;
    white-space: nowrap;
  }
  .share-preview {
    display: flex;
    justify-content: center;
    margin-bottom: 16px;
    position: relative;
  }
  .share-card-wrapper {
    position: relative;
    width: 340px;
    height: 567px;
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 16px 48px rgba(0,0,0,0.15);
  }
  .share-card-wrapper canvas {
    width: 100%;
    height: 100%;
    display: block;
  }
  .share-custom-note {
    margin-bottom: 16px;
  }
  .share-custom-note textarea {
    width: 100%;
    border: 1.5px solid var(--border);
    border-radius: 14px;
    padding: 12px 14px;
    font-size: 14px;
    font-family: inherit;
    resize: none;
    outline: none;
    background: var(--card);
    color: var(--text);
    line-height: 1.6;
    transition: border-color 0.2s ease;
  }
  .share-custom-note textarea:focus { border-color: var(--text); }
  .share-custom-note label {
    display: block;
    font-size: 13px;
    color: var(--text-secondary);
    margin-bottom: 6px;
    font-weight: 500;
  }
  .share-actions { display: flex; gap: 10px; }
  .share-actions button {
    flex: 1;
    padding: 12px;
    border-radius: 12px;
    border: 1.5px solid var(--border);
    background: var(--card);
    font-size: 14px;
    font-family: inherit;
    cursor: pointer;
    transition: all 0.2s ease;
    font-weight: 500;
  }
  .share-actions button:hover { border-color: var(--text); }
  .share-actions .btn-primary { background: var(--text); color: var(--card); border-color: var(--text); }
  .stats-panel {
    display: flex;
    gap: 10px;
    margin-top: 14px;
    padding-top: 12px;
    border-top: 1px solid var(--border);
  }
  .stat-item { flex: 1; text-align: center; padding: 10px; border-radius: 10px; background: var(--card); box-shadow: 0 1px 3px rgba(0,0,0,0.04); }
  .stat-num { font-size: 22px; font-weight: 500; color: var(--text); }
  .stat-label { font-size: 11px; color: var(--text-tertiary); margin-top: 2px; }
  @media (max-width: 480px) {
    .app-container { padding: 12px 10px 40px; }
    .share-card-wrapper { width: 300px; height: 500px; }
    .mood-grid { gap: 8px; }
    .mood-card { padding: 14px 4px; }
    .mood-emoji { font-size: 30px; }
  }
</style>
<base target="_blank">
<base target="_blank">
</head>
<body>

<canvas id="weather-canvas"></canvas>
<div class="weather-overlay" id="weather-overlay"></div>

<div class="app-container">

  <div class="page active" id="page-home">
    <div class="home-content">
      <div class="home-emoji">🌤️</div>
      <h1 class="home-title">情绪天气</h1>
      <p class="home-sub">今天心里是什么天气？<br>记录每一天的心情，像看天气预报一样温柔。</p>
      <button class="home-btn" onclick="goToCalendar()">进入日历</button>
    </div>
  </div>

  <div class="page" id="page-calendar">
    <div class="nav-bar">
      <button class="nav-back" onclick="goToHome()">←</button>
      <div class="nav-title">情绪日历</div>
      <div class="nav-spacer"></div>
    </div>
    <div class="calendar-header">
      <h2>你的心情地图</h2>
      <p>点击日期记录或查看心情</p>
    </div>
    <div class="month-nav">
      <button onclick="changeMonth(-1)">‹</button>
      <div class="month-label" id="month-label"></div>
      <button onclick="changeMonth(1)">›</button>
    </div>
    <div class="weekdays">
      <div class="weekday">日</div><div class="weekday">一</div><div class="weekday">二</div>
      <div class="weekday">三</div><div class="weekday">四</div><div class="weekday">五</div><div class="weekday">六</div>
    </div>
    <div class="calendar-grid" id="calendar-grid"></div>
    <div class="stats-panel" id="stats-panel"></div>
    <div class="calendar-actions">
      <button onclick="goToReport()">📊 月度报告</button>
      <button onclick="goToShare()">🖼️ 分享卡片</button>
    </div>
  </div>

  <div class="page" id="page-mood">
    <div class="mood-page-bg" id="mood-bg"></div>
    <div class="mood-page">
      <div class="nav-bar">
        <button class="nav-back" onclick="goToCalendar()">←</button>
        <div class="nav-title" id="mood-nav-title">记录心情</div>
        <div class="nav-spacer"></div>
      </div>
      <div class="mood-date">
        <h2 id="mood-date-title"></h2>
        <p id="mood-date-sub">选择今天的天气心情</p>
      </div>
      <div class="mood-grid" id="mood-grid"></div>
      <div class="mood-quote" id="mood-quote">点击上方选择一个心情...</div>
      <div class="mood-note-area">
        <textarea id="mood-note" rows="3" placeholder="写点什么...（可选）"></textarea>
      </div>
      <div class="mood-actions" id="mood-actions">
        <button onclick="goToCalendar()">取消</button>
        <button class="btn-primary" onclick="saveMood()">记录今天</button>
      </div>
    </div>
  </div>

  <div class="page" id="page-report">
    <div class="nav-bar">
      <button class="nav-back" onclick="goToCalendar()">←</button>
      <div class="nav-title">月度报告</div>
      <div class="nav-spacer"></div>
    </div>
    <div class="report-section">
      <h3>📊 心情分布</h3>
      <div class="report-card">
        <div class="mood-bars" id="mood-bars"></div>
        <div class="dominant-mood" id="dominant-mood"></div>
      </div>
    </div>
    <div class="report-section">
      <h3>📈 心情曲线</h3>
      <div class="report-card">
        <div class="curve-container" id="curve-container">
          <svg class="curve-svg" id="curve-svg"></svg>
          <div class="curve-tooltip" id="curve-tooltip"></div>
        </div>
      </div>
    </div>
  </div>

  <div class="page" id="page-share">
    <div class="nav-bar">
      <button class="nav-back" onclick="goToCalendar()">←</button>
      <div class="nav-title">分享卡片</div>
      <div class="nav-spacer"></div>
    </div>
    <div class="share-preview">
      <div class="share-card-wrapper">
        <canvas id="share-canvas" width="720" height="1200"></canvas>
      </div>
    </div>
    <div class="share-custom-note">
      <label>✏️ 卡片备注（可自定义）</label>
      <textarea id="share-note-input" rows="2" placeholder="在这里写下你想分享的话..."></textarea>
    </div>
    <div class="share-actions">
      <button class="btn-primary" onclick="downloadShareCard()">💾 保存高清图片</button>
    </div>
  </div>

</div>

<div class="detail-modal" id="detail-modal" onclick="closeDetail(event)">
  <div class="detail-content" onclick="event.stopPropagation()">
    <div class="detail-header">
      <div class="detail-emoji" id="detail-emoji"></div>
      <div>
        <div class="detail-title" id="detail-title"></div>
        <div class="detail-date" id="detail-date"></div>
      </div>
    </div>
    <div class="detail-note" id="detail-note"></div>
    <div class="detail-actions">
      <button onclick="closeDetail()">关闭</button>
      <button onclick="editFromDetail()">✏️ 编辑</button>
      <button class="btn-primary" onclick="shareThisDay()">🖼️ 分享</button>
    </div>
  </div>
</div>

<script>
const moods = [
  { emoji: '☀️', label: '晴朗', value: 'sunny', color: '#e8a87c', quote: '阳光很好，你也很好。', score: 3 },
  { emoji: '⛅', label: '多云', value: 'cloudy', color: '#b8c5d6', quote: '云层之上，阳光从未离开。', score: 1 },
  { emoji: '🌧️', label: '小雨', value: 'rainy', color: '#8faec7', quote: '雨会停，天会晴，没什么会一直糟糕。', score: -1 },
  { emoji: '⛈️', label: '雷雨', value: 'stormy', color: '#a8a8a8', quote: '雷声再大，也盖不过你的心跳。', score: -3 },
  { emoji: '❄️', label: '下雪', value: 'snowy', color: '#c4d4e0', quote: '慢一点，像雪一样安静地落。', score: 0 },
  { emoji: '🌈', label: '彩虹', value: 'rainbow', color: '#c7b07a', quote: '你值得所有美好的天气。', score: 4 }
];

let entries = {};
let currentDate = new Date();
let selectedDateKey = null;
let selectedMoodValue = null;
let detailDateKey = null;
let weatherAnimId = null;
let shareCardSeed = 0;
let currentShareMood = null;
let currentShareDate = '';
let shareCardAnimId = null;

function loadData() {
  const saved = localStorage.getItem('moodWeatherEntries_v4');
  if (saved) entries = JSON.parse(saved);
}
function saveData() {
  localStorage.setItem('moodWeatherEntries_v4', JSON.stringify(entries));
}

function showPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  window.scrollTo(0, 0);
}
function goToHome() { showPage('page-home'); stopWeather(); stopShareAnim(); }
function goToCalendar() {
  showPage('page-calendar');
  renderCalendar();
  applyCalendarWeather();
  stopShareAnim();
}
function goToMood(year, month, day) {
  selectedDateKey = formatKey(year, month, day);
  const entry = entries[selectedDateKey];
  selectedMoodValue = entry ? entry.mood : null;
  document.getElementById('mood-date-title').textContent = `${year}年${month+1}月${day}日`;
  document.getElementById('mood-date-sub').textContent = entry ? '修改今天的心情' : '选择今天的天气心情';
  document.getElementById('mood-nav-title').textContent = entry ? '编辑心情' : '记录心情';
  document.getElementById('mood-note').value = entry ? (entry.note || '') : '';
  const actions = document.getElementById('mood-actions');
  if (entry) {
    actions.innerHTML = `
      <button class="btn-danger" onclick="deleteMood()">🗑️ 删除</button>
      <button onclick="goToCalendar()">取消</button>
      <button class="btn-primary" onclick="saveMood()">保存修改</button>
    `;
  } else {
    actions.innerHTML = `
      <button onclick="goToCalendar()">取消</button>
      <button class="btn-primary" onclick="saveMood()">记录今天</button>
    `;
  }
  renderMoodGrid();
  updateMoodQuote();
  showPage('page-mood');
  if (selectedMoodValue) startWeatherEffect(selectedMoodValue);
}
function goToReport() { showPage('page-report'); renderReport(); stopWeather(); stopShareAnim(); }
function goToShare() { showPage('page-share'); generateShareCard(); stopWeather(); }

function formatKey(y, m, d) {
  return `${y}-${String(m).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
}
function formatDateKey(d) {
  return `${d.getFullYear()}-${String(d.getMonth()).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
}
function parseKey(key) {
  const [y, m, d] = key.split('-').map(Number);
  return { year: y, month: m, day: d };
}

function applyCalendarWeather() {
  const today = new Date();
  const todayKey = formatDateKey(today);
  const entry = entries[todayKey];
  if (entry) { startWeatherEffect(entry.mood); }
  else { stopWeather(); }
}

function renderCalendar() {
  const y = currentDate.getFullYear(), m = currentDate.getMonth();
  document.getElementById('month-label').textContent = `${y}年 ${m+1}月`;
  const firstDay = new Date(y, m, 1);
  const lastDay = new Date(y, m+1, 0);
  const startOffset = firstDay.getDay();
  const daysInMonth = lastDay.getDate();
  const grid = document.getElementById('calendar-grid');
  grid.innerHTML = '';
  const prevLast = new Date(y, m, 0).getDate();
  for (let i = startOffset-1; i >= 0; i--) {
    const cell = document.createElement('div');
    cell.className = 'day-cell other-month';
    cell.innerHTML = `<div class="day-num">${prevLast-i}</div>`;
    grid.appendChild(cell);
  }
  const now = new Date();
  for (let i = 1; i <= daysInMonth; i++) {
    const key = formatKey(y, m, i);
    const entry = entries[key];
    const mood = entry ? moods.find(m => m.value === entry.mood) : null;
    const isToday = now.getFullYear()===y && now.getMonth()===m && now.getDate()===i;
    const cell = document.createElement('div');
    cell.className = 'day-cell' + (isToday ? ' today' : '') + (entry ? ' has-mood' : '');
    cell.innerHTML = `<div class="day-num">${i}</div>${mood ? `<div class="day-mood">${mood.emoji}</div>` : ''}${mood ? `<div class="day-dot" style="background:${mood.color}"></div>` : ''}`;
    cell.onclick = () => { if (entry) showDetail(key); else goToMood(y, m, i); };
    grid.appendChild(cell);
  }
  const totalCells = startOffset + daysInMonth;
  const remaining = (7 - (totalCells % 7)) % 7;
  for (let i = 1; i <= remaining; i++) {
    const cell = document.createElement('div');
    cell.className = 'day-cell other-month';
    cell.innerHTML = `<div class="day-num">${i}</div>`;
    grid.appendChild(cell);
  }
  renderStats();
}
function changeMonth(delta) {
  currentDate.setMonth(currentDate.getMonth() + delta);
  renderCalendar();
  applyCalendarWeather();
}

function renderStats() {
  const keys = Object.keys(entries);
  const total = keys.length;
  const thisMonth = keys.filter(k => {
    const [y, m] = k.split('-').map(Number);
    return y === currentDate.getFullYear() && m === currentDate.getMonth();
  }).length;
  let streak = 0;
  const d = new Date();
  while (true) {
    const key = formatDateKey(d);
    if (entries[key]) { streak++; d.setDate(d.getDate()-1); }
    else break;
  }
  document.getElementById('stats-panel').innerHTML = `
    <div class="stat-item"><div class="stat-num">${total}</div><div class="stat-label">总记录</div></div>
    <div class="stat-item"><div class="stat-num">${thisMonth}</div><div class="stat-label">本月</div></div>
    <div class="stat-item"><div class="stat-num">${streak}</div><div class="stat-label">连续天</div></div>`;
}

function renderMoodGrid() {
  const grid = document.getElementById('mood-grid');
  grid.innerHTML = '';
  moods.forEach(m => {
    const card = document.createElement('div');
    card.className = 'mood-card' + (selectedMoodValue === m.value ? ' selected' : '');
    card.innerHTML = `<span class="mood-emoji">${m.emoji}</span><div class="mood-label">${m.label}</div>`;
    card.onclick = () => {
      selectedMoodValue = m.value;
      renderMoodGrid();
      updateMoodQuote();
      startWeatherEffect(m.value);
    };
    grid.appendChild(card);
  });
}
function updateMoodQuote() {
  const m = moods.find(x => x.value === selectedMoodValue);
  document.getElementById('mood-quote').textContent = m ? m.quote : '点击上方选择一个心情...';
}
function saveMood() {
  if (!selectedMoodValue) { alert('请先选择一个心情'); return; }
  entries[selectedDateKey] = {
    mood: selectedMoodValue,
    note: document.getElementById('mood-note').value.trim()
  };
  saveData();
  goToCalendar();
}
function deleteMood() {
  if (!confirm('确定要删除这条记录吗？')) return;
  delete entries[selectedDateKey];
  saveData();
  goToCalendar();
}

function showDetail(key) {
  detailDateKey = key;
  const entry = entries[key];
  const mood = moods.find(m => m.value === entry.mood);
  const { year, month, day } = parseKey(key);
  document.getElementById('detail-emoji').textContent = mood.emoji;
  document.getElementById('detail-title').textContent = mood.label;
  document.getElementById('detail-date').textContent = `${year}年${month+1}月${day}日`;
  document.getElementById('detail-note').textContent = entry.note || '（没有写备注）';
  document.getElementById('detail-modal').classList.add('show');
}
function closeDetail(e) {
  if (!e || e.target === document.getElementById('detail-modal')) {
    document.getElementById('detail-modal').classList.remove('show');
  }
}
function editFromDetail() {
  closeDetail();
  const { year, month, day } = parseKey(detailDateKey);
  goToMood(year, month, day);
}
function shareThisDay() {
  closeDetail();
  generateShareCard(detailDateKey);
  showPage('page-share');
}

const canvas = document.getElementById('weather-canvas');
const ctx = canvas.getContext('2d');
let particles = [];

function resizeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

function stopWeather() {
  if (weatherAnimId) cancelAnimationFrame(weatherAnimId);
  weatherAnimId = null;
  particles = [];
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  const overlay = document.getElementById('weather-overlay');
  overlay.className = 'weather-overlay';
  overlay.innerHTML = '';
  document.body.className = '';
}

function startWeatherEffect(type) {
  stopWeather();
  const overlay = document.getElementById('weather-overlay');
  document.body.classList.add('bg-' + type);

  if (type === 'rainy' || type === 'stormy') {
    overlay.className = 'weather-overlay active';
    particles = [];
    const count = type === 'stormy' ? 300 : 160;
    for (let i = 0; i < count; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        speed: 7 + Math.random() * 5,
        length: 12 + Math.random() * 18,
        opacity: 0.1 + Math.random() * 0.18
      });
    }
    animateRain();
  }
  else if (type === 'snowy') {
    overlay.className = 'weather-overlay active';
    particles = [];
    for (let i = 0; i < 100; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        r: 1.5 + Math.random() * 2.5,
        speed: 0.4 + Math.random() * 1.2,
        drift: (Math.random() - 0.5) * 0.4,
        opacity: 0.25 + Math.random() * 0.35
      });
    }
    animateSnow();
  }
  else if (type === 'sunny') {
    overlay.className = 'weather-overlay active overlay-sunny';
    for (let i = 0; i < 12; i++) {
      const p = document.createElement('div');
      p.className = 'sun-particle';
      const size = 20 + Math.random() * 50;
      p.style.cssText = `width:${size}px;height:${size}px;left:${Math.random()*100}%;animation-delay:${Math.random()*10}s;animation-duration:${7+Math.random()*6}s;`;
      overlay.appendChild(p);
    }
  }
  else if (type === 'cloudy') {
    overlay.className = 'weather-overlay active overlay-cloudy';
  }
  else if (type === 'rainbow') {
    overlay.className = 'weather-overlay active overlay-rainbow';
    particles = [];
    const colors = ['#e8a87c', '#c7b07a', '#8fb38f', '#8faec7', '#9b8fb5'];
    for (let i = 0; i < 50; i++) {
      particles.push({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        r: 2 + Math.random() * 3,
        color: colors[Math.floor(Math.random() * colors.length)],
        speed: 0.3 + Math.random() * 0.6,
        drift: (Math.random() - 0.5) * 0.8,
        opacity: 0.15 + Math.random() * 0.25
      });
    }
    animateRainbow();
  }
}

function animateRain() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.strokeStyle = 'rgba(120, 140, 160, 0.22)';
  ctx.lineWidth = 1;
  particles.forEach(p => {
    ctx.beginPath();
    ctx.moveTo(p.x, p.y);
    ctx.lineTo(p.x - 1, p.y + p.length);
    ctx.stroke();
    p.y += p.speed;
    p.x -= 0.5;
    if (p.y > canvas.height) { p.y = -p.length; p.x = Math.random() * canvas.width; }
  });
  weatherAnimId = requestAnimationFrame(animateRain);
}
function animateSnow() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  particles.forEach(p => {
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
    ctx.fillStyle = `rgba(255, 255, 255, ${p.opacity})`;
    ctx.fill();
    p.y += p.speed;
    p.x += p.drift;
    if (p.y > canvas.height + 10) { p.y = -10; p.x = Math.random() * canvas.width; }
    if (p.x > canvas.width + 10) p.x = -10;
    if (p.x < -10) p.x = canvas.width + 10;
  });
  weatherAnimId = requestAnimationFrame(animateSnow);
}
function animateRainbow() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  particles.forEach(p => {
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
    ctx.fillStyle = p.color;
    ctx.globalAlpha = p.opacity;
    ctx.fill();
    ctx.globalAlpha = 1;
    p.y -= p.speed;
    p.x += p.drift;
    if (p.y < -10) { p.y = canvas.height + 10; p.x = Math.random() * canvas.width; }
  });
  weatherAnimId = requestAnimationFrame(animateRainbow);
}

function renderReport() {
  const y = currentDate.getFullYear();
  const m = currentDate.getMonth();
  const monthKeys = Object.keys(entries).filter(k => {
    const [ey, em] = k.split('-').map(Number);
    return ey === y && em === m;
  }).sort();

  const counts = {};
  moods.forEach(m => counts[m.value] = 0);
  monthKeys.forEach(k => {
    const mv = entries[k].mood;
    if (counts[mv] !== undefined) counts[mv]++;
  });
  const total = monthKeys.length;
  const maxCount = Math.max(...Object.values(counts), 1);

  const barsContainer = document.getElementById('mood-bars');
  barsContainer.innerHTML = '';
  moods.forEach(m => {
    const count = counts[m.value];
    const pct = total > 0 ? (count / maxCount * 100) : 0;
    const row = document.createElement('div');
    row.className = 'mood-bar-row';
    row.innerHTML = `
      <div class="mood-bar-emoji">${m.emoji}</div>
      <div class="mood-bar-label">${m.label}</div>
      <div class="mood-bar-track"><div class="mood-bar-fill" style="width:0%;background:${m.color}" data-width="${pct}%"></div></div>
      <div class="mood-bar-count">${count}</div>`;
    barsContainer.appendChild(row);
  });
  setTimeout(() => {
    barsContainer.querySelectorAll('.mood-bar-fill').forEach(el => {
      el.style.width = el.dataset.width;
    });
  }, 100);

  let dominant = moods[0];
  let maxC = 0;
  moods.forEach(m => {
    if (counts[m.value] > maxC) { maxC = counts[m.value]; dominant = m; }
  });
  const domContainer = document.getElementById('dominant-mood');
  if (total === 0) {
    domContainer.innerHTML = `<div class="big-desc">本月还没有记录心情哦～</div>`;
  } else {
    const avgScore = monthKeys.reduce((sum, k) => {
      const mv = entries[k].mood;
      const mood = moods.find(x => x.value === mv);
      return sum + (mood ? mood.score : 0);
    }, 0) / total;
    let desc = '';
    if (avgScore >= 2) desc = '本月心情总体很晴朗，继续保持这份温暖吧！';
    else if (avgScore >= 0) desc = '本月心情平稳，有起有落才是生活的常态。';
    else if (avgScore >= -2) desc = '本月有些阴郁，但雨后总会见彩虹。';
    else desc = '本月经历了一些风雨，记得对自己温柔一点。';
    domContainer.innerHTML = `
      <div class="big-emoji">${dominant.emoji}</div>
      <div class="big-label">本月主导：${dominant.label}</div>
      <div class="big-desc">${desc}</div>`;
  }

  renderCurve(monthKeys);
}

function renderCurve(monthKeys) {
  const svg = document.getElementById('curve-svg');
  const container = document.getElementById('curve-container');
  const tooltip = document.getElementById('curve-tooltip');
  svg.innerHTML = '';
  if (monthKeys.length < 2) {
    svg.innerHTML = `<text x="50%" y="50%" text-anchor="middle" fill="#999" font-size="13">记录少于2天，暂无曲线</text>`;
    return;
  }

  const w = container.clientWidth;
  const h = 180;
  const padding = { top: 20, right: 20, bottom: 30, left: 30 };
  const chartW = w - padding.left - padding.right;
  const chartH = h - padding.top - padding.bottom;

  const points = monthKeys.map((k, i) => {
    const mv = entries[k].mood;
    const mood = moods.find(x => x.value === mv);
    return {
      x: padding.left + (i / (monthKeys.length - 1)) * chartW,
      y: padding.top + chartH - ((mood.score + 4) / 8) * chartH,
      score: mood.score,
      emoji: mood.emoji,
      label: mood.label,
      date: k,
      note: entries[k].note || ''
    };
  });

  for (let i = 0; i <= 4; i++) {
    const y = padding.top + (i / 4) * chartH;
    const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    line.setAttribute('x1', padding.left);
    line.setAttribute('y1', y);
    line.setAttribute('x2', w - padding.right);
    line.setAttribute('y2', y);
    line.setAttribute('stroke', '#e8e4df');
    line.setAttribute('stroke-width', '1');
    svg.appendChild(line);
  }

  let pathD = `M ${points[0].x} ${points[0].y}`;
  for (let i = 1; i < points.length; i++) {
    const prev = points[i-1];
    const curr = points[i];
    const cp1x = prev.x + (curr.x - prev.x) * 0.5;
    const cp1y = prev.y;
    const cp2x = prev.x + (curr.x - prev.x) * 0.5;
    const cp2y = curr.y;
    pathD += ` C ${cp1x} ${cp1y}, ${cp2x} ${cp2y}, ${curr.x} ${curr.y}`;
  }
  const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  path.setAttribute('d', pathD);
  path.setAttribute('fill', 'none');
  path.setAttribute('stroke', '#7a9eb8');
  path.setAttribute('stroke-width', '2.5');
  path.setAttribute('stroke-linecap', 'round');
  svg.appendChild(path);

  const areaD = pathD + ` L ${points[points.length-1].x} ${padding.top + chartH} L ${points[0].x} ${padding.top + chartH} Z`;
  const area = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  area.setAttribute('d', areaD);
  area.setAttribute('fill', 'rgba(122, 158, 184, 0.08)');
  area.setAttribute('stroke', 'none');
  svg.appendChild(area);

  points.forEach((p, i) => {
    const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
    circle.setAttribute('cx', p.x);
    circle.setAttribute('cy', p.y);
    circle.setAttribute('r', '5');
    circle.setAttribute('fill', '#fff');
    circle.setAttribute('stroke', '#7a9eb8');
    circle.setAttribute('stroke-width', '2');
    circle.style.cursor = 'pointer';
    circle.addEventListener('mouseenter', () => {
      circle.setAttribute('r', '7');
      tooltip.style.left = (p.x - 30) + 'px';
      tooltip.style.top = (p.y - 36) + 'px';
      tooltip.textContent = `${p.date.slice(5)} ${p.emoji} ${p.label}`;
      tooltip.style.opacity = '1';
    });
    circle.addEventListener('mouseleave', () => {
      circle.setAttribute('r', '5');
      tooltip.style.opacity = '0';
    });
    svg.appendChild(circle);
  });

  const step = Math.ceil(points.length / 6);
  points.forEach((p, i) => {
    if (i % step === 0 || i === points.length - 1) {
      const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      text.setAttribute('x', p.x);
      text.setAttribute('y', h - 8);
      text.setAttribute('text-anchor', 'middle');
      text.setAttribute('fill', '#999');
      text.setAttribute('font-size', '10');
      text.textContent = p.date.slice(8) + '日';
      svg.appendChild(text);
    }
  });
}

let shareParticles = [];

function stopShareAnim() {
  if (shareCardAnimId) cancelAnimationFrame(shareCardAnimId);
  shareCardAnimId = null;
  shareParticles = [];
}

function generateShareCard(specificKey) {
  stopShareAnim();
  shareCardSeed++;

  let entry, mood, dateStr, note;
  if (specificKey && entries[specificKey]) {
    entry = entries[specificKey];
    mood = moods.find(m => m.value === entry.mood);
    const { year, month, day } = parseKey(specificKey);
    dateStr = `${year}年${month+1}月${day}日`;
    note = entry.note || '';
  } else {
    const keys = Object.keys(entries).sort();
    if (keys.length > 0) {
      const k = keys[keys.length - 1];
      entry = entries[k];
      mood = moods.find(m => m.value === entry.mood);
      const { year, month, day } = parseKey(k);
      dateStr = `${year}年${month+1}月${day}日`;
      note = entry.note || '';
    } else {
      mood = moods[0];
      dateStr = '今天';
      note = '开始记录你的心情吧～';
    }
  }

  currentShareMood = mood;
  currentShareDate = dateStr;

  const input = document.getElementById('share-note-input');
  input.value = note && note !== '开始记录你的心情吧～' ? note : '';
  input.oninput = () => { drawShareCardFrame(); };

  initShareParticles(mood.value);
  drawShareCardFrame();
}

function initShareParticles(type) {
  shareParticles = [];
  const w = 720, h = 1200;

  if (type === 'rainy' || type === 'stormy') {
    const count = type === 'stormy' ? 100 : 60;
    for (let i = 0; i < count; i++) {
      shareParticles.push({
        type: 'rain',
        x: Math.random() * w,
        y: Math.random() * h,
        speed: 5 + Math.random() * 4,
        length: 12 + Math.random() * 18,
        opacity: 0.06 + Math.random() * 0.1
      });
    }
    if (type === 'stormy') {
      for (let i = 0; i < 2; i++) {
        shareParticles.push({
          type: 'lightning',
          x: 150 + Math.random() * (w - 300),
          y: 60 + Math.random() * 150,
          opacity: 0,
          flashSpeed: 0.015 + Math.random() * 0.02,
          flashDir: 1,
          segments: generateLightningSegments()
        });
      }
    }
  }
  else if (type === 'snowy') {
    for (let i = 0; i < 60; i++) {
      shareParticles.push({
        type: 'snow',
        x: Math.random() * w,
        y: Math.random() * h,
        r: 2 + Math.random() * 3,
        speed: 0.4 + Math.random() * 0.8,
        drift: (Math.random() - 0.5) * 0.4,
        opacity: 0.25 + Math.random() * 0.35
      });
    }
  }
  else if (type === 'sunny') {
    for (let i = 0; i < 25; i++) {
      shareParticles.push({
        type: 'sun',
        x: Math.random() * w,
        y: Math.random() * h,
        r: 3 + Math.random() * 10,
        speed: 0.2 + Math.random() * 0.4,
        opacity: 0.08 + Math.random() * 0.15,
        phase: Math.random() * Math.PI * 2
      });
    }
  }
  else if (type === 'rainbow') {
    const colors = ['#e8a87c', '#c7b07a', '#8fb38f', '#8faec7', '#9b8fb5'];
    for (let i = 0; i < 50; i++) {
      shareParticles.push({
        type: 'rb',
        x: Math.random() * w,
        y: Math.random() * h,
        r: 3 + Math.random() * 5,
        color: colors[Math.floor(Math.random() * colors.length)],
        speed: 0.25 + Math.random() * 0.4,
        drift: (Math.random() - 0.5) * 0.5,
        opacity: 0.12 + Math.random() * 0.18
      });
    }
  }
  else if (type === 'cloudy') {
    for (let i = 0; i < 6; i++) {
      shareParticles.push({
        type: 'cloud',
        x: Math.random() * w,
        y: 40 + Math.random() * 250,
        r: 50 + Math.random() * 80,
        speed: 0.15 + Math.random() * 0.2,
        opacity: 0.04 + Math.random() * 0.05
      });
    }
  }
}

function generateLightningSegments() {
  const segs = [];
  let x = 0, y = 0;
  for (let i = 0; i < 8; i++) {
    const nx = x + (Math.random() - 0.5) * 50;
    const ny = y + 25 + Math.random() * 35;
    segs.push({ x1: x, y1: y, x2: nx, y2: ny });
    x = nx; y = ny;
  }
  return segs;
}

function hexToRgba(hex, alpha) {
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  return `rgba(${r},${g},${b},${alpha})`;
}

function drawShareCardFrame() {
  const canvas = document.getElementById('share-canvas');
  const c = canvas.getContext('2d');
  const w = 720, h = 1200;
  const mood = currentShareMood;
  const dateStr = currentShareDate;
  const customNote = document.getElementById('share-note-input').value.trim();

  // 背景
  const bgColors = {
    sunny:  ['#FFF8F0', '#FFF0E0'],
    cloudy: ['#F0F4F8', '#E8EEF5'],
    rainy:  ['#E8EEF5', '#E0E8F0'],
    stormy: ['#E8E8E8', '#E0E0E0'],
    snowy:  ['#F0F5FA', '#E8F0F8'],
    rainbow:['#FAF8F5', '#F5F0E8']
  };
  const bg = bgColors[mood.value] || bgColors.sunny;
  const grd = c.createLinearGradient(0, 0, 0, h);
  grd.addColorStop(0, bg[0]);
  grd.addColorStop(1, bg[1]);
  c.fillStyle = grd;
  c.fillRect(0, 0, w, h);

  // 天气粒子动画
  updateShareParticles();
  drawShareWeather(c, w, h);

  // 装饰圆
  c.fillStyle = 'rgba(255,255,255,0.35)';
  c.beginPath();
  c.arc(w - 80, 180, 200, 0, Math.PI * 2);
  c.fill();
  c.fillStyle = 'rgba(255,255,255,0.2)';
  c.beginPath();
  c.arc(100, h - 200, 150, 0, Math.PI * 2);
  c.fill();

  // 小圆点装饰
  const decoColor = mood.color;
  for (let i = 0; i < 12; i++) {
    c.fillStyle = hexToRgba(decoColor, 0.1);
    c.beginPath();
    c.arc(50 + (i * 57) % 620, 350 + (i * 93) % 600, 4 + (i % 4) * 3, 0, Math.PI * 2);
    c.fill();
  }

  // 顶部标签
  c.fillStyle = 'rgba(0,0,0,0.05)';
  roundRect(c, w/2 - 60, 40, 120, 36, 18);
  c.fill();
  c.font = '500 22px sans-serif';
  c.fillStyle = '#888';
  c.textAlign = 'center';
  c.textBaseline = 'middle';
  c.fillText('情绪天气', w/2, 58);

  // 大表情
  c.font = '160px sans-serif';
  c.textAlign = 'center';
  c.textBaseline = 'alphabetic';
  c.fillText(mood.emoji, w/2, 320);

  // 心情标签
  c.font = 'bold 44px sans-serif';
  c.fillStyle = '#2d2d2d';
  c.fillText(`今日心情：${mood.label}`, w/2, 390);

  // 日期
  c.font = '28px sans-serif';
  c.fillStyle = '#999';
  c.fillText(dateStr, w/2, 440);

  // 分隔装饰线
  c.strokeStyle = hexToRgba(decoColor, 0.3);
  c.lineWidth = 2;
  c.beginPath();
  c.moveTo(120, 480);
  c.lineTo(w - 120, 480);
  c.stroke();

  // 寄语
  c.font = 'italic 32px sans-serif';
  c.fillStyle = '#555';
  wrapText(c, mood.quote, w/2, 540, 560, 44);

  // 自定义备注区域
  const noteToShow = customNote || '';
  if (noteToShow) {
    c.fillStyle = 'rgba(255,255,255,0.55)';
    roundRect(c, 60, 640, w - 120, 180, 24);
    c.fill();

    c.font = '26px sans-serif';
    c.fillStyle = '#666';
    c.textAlign = 'left';
    wrapTextLeft(c, noteToShow, 100, 690, w - 200, 38);
  }

  // 底部装饰
  c.textAlign = 'center';
  c.font = '22px sans-serif';
  c.fillStyle = '#bbb';
  c.fillText('— 情绪天气 · Mood Weather —', w/2, h - 70);
  c.font = '28px sans-serif';
  c.fillText('🌤️', w/2, h - 35);

  shareCardAnimId = requestAnimationFrame(drawShareCardFrame);
}

function updateShareParticles() {
  const w = 720, h = 1200;
  shareParticles.forEach(p => {
    if (p.type === 'rain') {
      p.y += p.speed;
      p.x -= 0.5;
      if (p.y > h) { p.y = -p.length; p.x = Math.random() * w; }
    }
    else if (p.type === 'snow') {
      p.y += p.speed;
      p.x += p.drift;
      if (p.y > h + 10) { p.y = -10; p.x = Math.random() * w; }
    }
    else if (p.type === 'sun') {
      p.y -= p.speed;
      p.phase += 0.02;
      p.x += Math.sin(p.phase) * 0.3;
      if (p.y < -20) { p.y = h + 20; p.x = Math.random() * w; }
    }
    else if (p.type === 'rb') {
      p.y -= p.speed;
      p.x += p.drift;
      if (p.y < -10) { p.y = h + 10; p.x = Math.random() * w; }
    }
    else if (p.type === 'cloud') {
      p.x += p.speed;
      if (p.x > w + p.r) p.x = -p.r;
    }
    else if (p.type === 'lightning') {
      p.opacity += p.flashSpeed * p.flashDir;
      if (p.opacity > 0.8) { p.opacity = 0.8; p.flashDir = -1; }
      if (p.opacity < 0) { p.opacity = 0; p.flashDir = 1; p.flashSpeed = 0.01 + Math.random() * 0.02; }
    }
  });
}

function drawShareWeather(c, w, h) {
  shareParticles.forEach(p => {
    if (p.type === 'rain') {
      c.strokeStyle = `rgba(130, 150, 170, ${p.opacity})`;
      c.lineWidth = 1.5;
      c.beginPath();
      c.moveTo(p.x, p.y);
      c.lineTo(p.x - 1, p.y + p.length);
      c.stroke();
    }
    else if (p.type === 'snow') {
      c.beginPath();
      c.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      c.fillStyle = `rgba(255, 255, 255, ${p.opacity})`;
      c.fill();
    }
    else if (p.type === 'sun') {
      c.beginPath();
      c.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      c.fillStyle = `rgba(255, 210, 120, ${p.opacity})`;
      c.fill();
    }
    else if (p.type === 'rb') {
      c.beginPath();
      c.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      c.fillStyle = p.color;
      c.globalAlpha = p.opacity;
      c.fill();
      c.globalAlpha = 1;
    }
    else if (p.type === 'cloud') {
      c.fillStyle = `rgba(180, 190, 200, ${p.opacity})`;
      c.beginPath();
      c.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      c.arc(p.x + p.r * 0.6, p.y - p.r * 0.3, p.r * 0.8, 0, Math.PI * 2);
      c.arc(p.x - p.r * 0.5, p.y - p.r * 0.2, p.r * 0.7, 0, Math.PI * 2);
      c.fill();
    }
    else if (p.type === 'lightning') {
      if (p.opacity > 0.05) {
        c.strokeStyle = `rgba(255, 255, 240, ${p.opacity})`;
        c.lineWidth = 3;
        c.shadowColor = 'rgba(255, 255, 200, 0.8)';
        c.shadowBlur = 20;
        c.beginPath();
        p.segments.forEach((seg, i) => {
          if (i === 0) c.moveTo(p.x + seg.x1, p.y + seg.y1);
          c.lineTo(p.x + seg.x2, p.y + seg.y2);
        });
        c.stroke();
        c.shadowBlur = 0;
      }
    }
  });
}

function roundRect(c, x, y, w, h, r) {
  c.beginPath();
  c.moveTo(x + r, y);
  c.lineTo(x + w - r, y);
  c.quadraticCurveTo(x + w, y, x + w, y + r);
  c.lineTo(x + w, y + h - r);
  c.quadraticCurveTo(x + w, y + h, x + w - r, y + h);
  c.lineTo(x + r, y + h);
  c.quadraticCurveTo(x, y + h, x, y + h - r);
  c.lineTo(x, y + r);
  c.quadraticCurveTo(x, y, x + r, y);
  c.closePath();
}

function wrapText(c, text, x, y, maxWidth, lineHeight) {
  const chars = text.split('');
  let line = '';
  let lines = [];
  for (let ch of chars) {
    const test = line + ch;
    if (c.measureText(test).width > maxWidth && line) {
      lines.push(line);
      line = ch;
    } else line = test;
  }
  lines.push(line);
  lines.forEach((l, i) => {
    c.fillText(l, x, y + i * lineHeight);
  });
}

function wrapTextLeft(c, text, x, y, maxWidth, lineHeight) {
  const chars = text.split('');
  let line = '';
  let lines = [];
  for (let ch of chars) {
    const test = line + ch;
    if (c.measureText(test).width > maxWidth && line) {
      lines.push(line);
      line = ch;
    } else line = test;
  }
  lines.push(line);
  lines = lines.slice(0, 4);
  lines.forEach((l, i) => {
    c.fillText(l, x, y + i * lineHeight);
  });
}

function regenerateShareCard() {
  initShareParticles(currentShareMood.value);
}

function downloadShareCard() {
  const canvas = document.getElementById('share-canvas');
  const link = document.createElement('a');
  link.download = 'mood-weather-card.png';
  link.href = canvas.toDataURL('image/png');
  link.click();
}

loadData();
</script>
</body>
</html>
