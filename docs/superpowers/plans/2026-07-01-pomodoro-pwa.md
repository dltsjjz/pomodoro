# 番茄钟 PWA 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 构建一个可在手机浏览器上运行的极简番茄钟 PWA，支持自定义时长、工作/休息切换、响铃通知。

**Architecture:** 单页面 HTML 应用，CSS/JS 内嵌，manifest.json + sw.js 实现 PWA 可安装与离线访问。

**Tech Stack:** HTML5 + CSS3 + Vanilla JS, GitHub Pages 部署

---

### Task 1: 创建项目文件结构和 manifest.json

**Files:**
- Create: `manifest.json`
- Create: `sw.js`

- [ ] **Step 1: 创建 manifest.json**

```json
{
  "name": "番茄钟",
  "short_name": "番茄",
  "start_url": ".",
  "display": "standalone",
  "theme_color": "#E53935",
  "background_color": "#F5F5F5",
  "icons": [
    {
      "src": "data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🍅</text></svg>",
      "sizes": "100x100",
      "type": "image/svg+xml"
    }
  ]
}
```

- [ ] **Step 2: 创建 sw.js（基础缓存）**

```js
const CACHE = "pomodoro-v1";
self.addEventListener("install", e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll([".", "index.html", "manifest.json"])));
});
self.addEventListener("fetch", e => {
  e.respondWith(caches.match(e.request).then(r => r || fetch(e.request)));
});
```

- [ ] **Step 3: 提交**

```bash
git add manifest.json sw.js
git commit -m "feat: add PWA manifest and service worker"
```

---

### Task 2: 创建番茄钟主页面（HTML + CSS）

**Files:**
- Create: `index.html`

- [ ] **Step 1: 创建 index.html 骨架和样式**

完整代码如下（HTML结构 + CSS样式）：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<meta name="theme-color" content="#E53935">
<title>番茄钟</title>
<link rel="manifest" href="manifest.json">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100dvh;
  background: #F5F5F5;
  color: #333;
  transition: background 0.5s;
  -webkit-tap-highlight-color: transparent;
  user-select: none;
}
body.work { background: #FFF5F5; }
body.break { background: #F1F8E9; }
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 28px;
  padding: 20px;
  width: 100%;
  max-width: 400px;
}
.emoji { font-size: 64px; transition: transform 0.3s; }
.timer-ring {
  position: relative;
  width: 240px;
  height: 240px;
}
.timer-ring svg { transform: rotate(-90deg); width: 240px; height: 240px; }
.timer-ring .bg { fill: none; stroke: #E0E0E0; stroke-width: 8; }
.timer-ring .progress { fill: none; stroke: #E53935; stroke-width: 8; stroke-linecap: round; transition: stroke-dashoffset 0.3s, stroke 0.5s; }
body.break .timer-ring .progress { stroke: #43A047; }
.timer-text {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
  font-size: 48px;
  font-weight: 600;
  font-variant-numeric: tabular-nums;
  letter-spacing: 2px;
}
.buttons { display: flex; gap: 16px; }
.btn {
  padding: 14px 36px;
  border: none;
  border-radius: 50px;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.2s;
  color: #fff;
}
.btn:active { transform: scale(0.95); }
.btn-start { background: #E53935; box-shadow: 0 4px 15px rgba(229,57,53,0.4); }
body.break .btn-start { background: #43A047; box-shadow: 0 4px 15px rgba(67,160,71,0.4); }
.btn-reset { background: #757575; box-shadow: 0 4px 15px rgba(117,117,117,0.3); }
.status {
  font-size: 14px;
  font-weight: 500;
  color: #E53935;
  text-transform: uppercase;
  letter-spacing: 3px;
}
body.break .status { color: #43A047; }
.settings-toggle {
  font-size: 14px;
  color: #999;
  cursor: pointer;
  padding: 8px 16px;
  border: 1px solid #ddd;
  border-radius: 20px;
  background: #fff;
}
.settings {
  display: none;
  flex-direction: column;
  gap: 12px;
  background: #fff;
  padding: 16px 20px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  width: 100%;
  max-width: 280px;
}
.settings.open { display: flex; }
.setting-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}
.setting-row label { font-size: 15px; white-space: nowrap; }
.setting-row input {
  width: 70px;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  text-align: center;
}
</style>
</head>
<body class="work">
<div class="container">
  <div class="emoji">🍅</div>
  <div class="timer-ring">
    <svg viewBox="0 0 240 240">
      <circle class="bg" cx="120" cy="120" r="110"/>
      <circle class="progress" id="progress" cx="120" cy="120" r="110"
        stroke-dasharray="691.15" stroke-dashoffset="0"/>
    </svg>
    <div class="timer-text" id="timer">25:00</div>
  </div>
  <div class="status" id="status">工作中</div>
  <div class="buttons">
    <button class="btn btn-start" id="btnStart">开始</button>
    <button class="btn btn-reset" id="btnReset">重置</button>
  </div>
  <div class="settings-toggle" id="settingsToggle">⚙ 设置 (25 / 5 分钟)</div>
  <div class="settings" id="settings">
    <div class="setting-row">
      <label>工作 (分钟)</label>
      <input type="number" id="workTime" value="25" min="1" max="120">
    </div>
    <div class="setting-row">
      <label>休息 (分钟)</label>
      <input type="number" id="breakTime" value="5" min="1" max="60">
    </div>
  </div>
</div>
```

- [ ] **Step 2: 提交**

```bash
git add index.html
git commit -m "feat: add pomodoro timer HTML structure and styles"
```

---

### Task 3: 添加番茄钟核心 JS 逻辑

**Files:**
- Modify: `index.html`（在 `</div>` 结束标签前、`</body>` 前添加 JS）

- [ ] **Step 1: 添加完整 JS 逻辑**

在 `</div>`（container的闭合标签）之后、`</body>` 之前插入：

```html
<script>
const CIRCUMFERENCE = 2 * Math.PI * 110; // r=110

// DOM refs
const timerEl = document.getElementById("timer");
const progressEl = document.getElementById("progress");
const statusEl = document.getElementById("status");
const btnStart = document.getElementById("btnStart");
const btnReset = document.getElementById("btnReset");
const workInput = document.getElementById("workTime");
const breakInput = document.getElementById("breakTime");
const settingsToggle = document.getElementById("settingsToggle");
const settingsEl = document.getElementById("settings");
const bodyEl = document.body;

// State
let workMinutes = parseInt(workInput.value) || 25;
let breakMinutes = parseInt(breakInput.value) || 5;
let totalSeconds = workMinutes * 60;
let remaining = totalSeconds;
let interval = null;
let isWork = true;
let running = false;

// Load saved
const saved = JSON.parse(localStorage.getItem("pomodoro") || "{}");
if (saved.work) { workMinutes = saved.work; workInput.value = workMinutes; }
if (saved.break) { breakMinutes = saved.break; breakInput.value = breakMinutes; }
totalSeconds = workMinutes * 60;
remaining = totalSeconds;

function save() {
  localStorage.setItem("pomodoro", JSON.stringify({ work: workMinutes, break: breakMinutes }));
}

function format(sec) {
  const m = Math.floor(sec / 60);
  const s = sec % 60;
  return String(m).padStart(2, "0") + ":" + String(s).padStart(2, "0");
}

function updateDisplay() {
  timerEl.textContent = format(remaining);
  const offset = CIRCUMFERENCE * (1 - remaining / totalSeconds);
  progressEl.style.strokeDashoffset = offset;
}

function setPhase(work) {
  isWork = work;
  if (isWork) {
    bodyEl.className = "work";
    statusEl.textContent = "工作中";
    totalSeconds = workMinutes * 60;
  } else {
    bodyEl.className = "break";
    statusEl.textContent = "休息中";
    totalSeconds = breakMinutes * 60;
  }
  remaining = totalSeconds;
  updateDisplay();
}

function notify() {
  // Play sound
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.frequency.value = 800;
    osc.type = "sine";
    gain.gain.value = 0.3;
    osc.start();
    osc.stop(ctx.currentTime + 0.3);
    setTimeout(() => {
      const osc2 = ctx.createOscillator();
      osc2.connect(gain);
      osc2.frequency.value = 1000;
      osc2.type = "sine";
      osc2.start();
      osc2.stop(ctx.currentTime + 0.3);
    }, 400);
    setTimeout(() => {
      const osc3 = ctx.createOscillator();
      osc3.connect(gain);
      osc3.frequency.value = 1200;
      osc3.type = "sine";
      osc3.start();
      osc3.stop(ctx.currentTime + 0.5);
    }, 800);
  } catch (e) {}

  // Vibrate
  if (navigator.vibrate) navigator.vibrate([200, 100, 200, 100, 400]);

  // Notification
  if ("Notification" in window && Notification.permission === "granted") {
    new Notification(isWork ? "休息时间到！" : "工作时间到！", {
      body: isWork ? "该休息了" : "开始工作吧",
      icon: "data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>🍅</text></svg>"
    });
  }
}

function tick() {
  if (remaining <= 0) {
    clearInterval(interval);
    interval = null;
    running = false;
    btnStart.textContent = "开始";
    notify();
    setPhase(!isWork);
    return;
  }
  remaining--;
  updateDisplay();
}

btnStart.addEventListener("click", () => {
  if (running) {
    clearInterval(interval);
    interval = null;
    running = false;
    btnStart.textContent = "继续";
    return;
  }

  if ("Notification" in window && Notification.permission === "default") {
    Notification.requestPermission();
  }

  running = true;
  btnStart.textContent = "暂停";
  interval = setInterval(tick, 1000);
});

btnReset.addEventListener("click", () => {
  clearInterval(interval);
  interval = null;
  running = false;
  btnStart.textContent = "开始";
  setPhase(isWork);
});

workInput.addEventListener("change", () => {
  workMinutes = Math.max(1, parseInt(workInput.value) || 25);
  workInput.value = workMinutes;
  save();
  if (!running && isWork) { totalSeconds = workMinutes * 60; remaining = totalSeconds; updateDisplay(); }
});

breakInput.addEventListener("change", () => {
  breakMinutes = Math.max(1, parseInt(breakInput.value) || 5);
  breakInput.value = breakMinutes;
  save();
  if (!running && !isWork) { totalSeconds = breakMinutes * 60; remaining = totalSeconds; updateDisplay(); }
});

settingsToggle.addEventListener("click", () => {
  const open = settingsEl.classList.toggle("open");
  settingsToggle.textContent = open
    ? "⚙ 收起设置"
    : `⚙ 设置 (${workMinutes} / ${breakMinutes} 分钟)`;
});

updateDisplay();

// Register service worker
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("sw.js");
}
</script>
</body>
</html>
```

- [ ] **Step 2: 提交**

```bash
git add index.html
git commit -m "feat: add pomodoro timer core logic"
```

---

### Task 4: 验证和部署

- [ ] **Step 1: 本地验证**

用浏览器打开 `index.html`，验证：
- 倒计时显示正常
- 开始/暂停/重置按钮功能正常
- 设置面板可展开，修改时长生效
- 计时结束时播放声音、振动、弹出通知

- [ ] **Step 2: 推送到 GitHub 并启用 Pages**

```bash
# 创建 GitHub 仓库（需要先有 gh CLI 登录）
gh repo create pomodoro --public --source . --push
# 启用 GitHub Pages（Settings > Pages > Source: main branch, / (root)）
```

- [ ] **Step 3: 手机测试**

在手机 Chrome 打开 `https://<username>.github.io/pomodoro`，点击菜单 → "添加到主屏幕"。

- [ ] **Step 4: 提交最终版本**

```bash
git add -A
git commit -m "chore: finalize pomodoro PWA"
git push
```
