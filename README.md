# T-gliche-Aufgaben-
<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Täglicher Aufgaben Tracker</title>

<style>
:root {
  --bg: #f4f6f9;
  --card: #ffffff;
  --text: #222;
  --accent: #4CAF50;
}

body.dark {
  --bg: #1e1e1e;
  --card: #2c2c2c;
  --text: #f4f4f4;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background: var(--bg);
  color: var(--text);
  display: flex;
  justify-content: center;
}

.container {
  width: 95%;
  max-width: 700px;
  margin: 20px;
}

.card {
  background: var(--card);
  padding: 20px;
  border-radius: 10px;
  margin-bottom: 20px;
}

button {
  padding: 8px 12px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  background: var(--accent);
  color: white;
}

input {
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #ccc;
}

.task {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
}

.task.done span {
  text-decoration: line-through;
  opacity: 0.6;
}

.progress-bar {
  height: 20px;
  background: #ddd;
  border-radius: 10px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: var(--accent);
  width: 0%;
  transition: 0.3s;
}

.small {
  font-size: 0.9em;
  opacity: 0.8;
}
</style>
</head>

<body>
<div class="container">

<div class="card">
<h2>📅 Tägliche Aufgaben</h2>

<input type="text" id="taskInput" placeholder="Neue Aufgabe...">
<button onclick="addTask()">Hinzufügen</button>
<button onclick="toggleDarkMode()">🌙</button>

<div id="taskList"></div>

<h3>Fortschritt</h3>
<div class="progress-bar">
  <div class="progress-fill" id="progressFill"></div>
</div>
<p id="progressText"></p>
<p id="motivation"></p>
</div>

<div class="card">
<h3>🔥 Streak</h3>
<p id="streak"></p>

<h3>📊 Statistik</h3>
<p id="stats"></p>

<button onclick="exportCSV()">CSV Export</button>
</div>

</div>

<script>
let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
let history = JSON.parse(localStorage.getItem("history")) || {};
let lastDate = localStorage.getItem("lastDate");

const today = new Date().toISOString().split("T")[0];

if (lastDate !== today) {
  saveDay();
  resetTasks();
  localStorage.setItem("lastDate", today);
}

function addTask() {
  const input = document.getElementById("taskInput");
  if (input.value.trim() === "") return;

  tasks.push({ text: input.value, done: false });
  input.value = "";
  save();
  render();
}

function toggleTask(index) {
  tasks[index].done = !tasks[index].done;
  save();
  render();
}

function deleteTask(index) {
  tasks.splice(index, 1);
  save();
  render();
}

function render() {
  const list = document.getElementById("taskList");
  list.innerHTML = "";

  let doneCount = 0;

  tasks.forEach((task, index) => {
    if (task.done) doneCount++;

    const div = document.createElement("div");
    div.className = "task" + (task.done ? " done" : "");

    div.innerHTML = `
      <span onclick="toggleTask(${index})">${task.text}</span>
      <button onclick="deleteTask(${index})">X</button>
    `;
    list.appendChild(div);
  });

  updateProgress(doneCount);
  updateStats();
}

function updateProgress(doneCount) {
  const total = tasks.length;
  const percent = total === 0 ? 0 : (doneCount / total) * 100;

  document.getElementById("progressFill").style.width = percent + "%";
  document.getElementById("progressText").innerText =
    `${doneCount} von ${total} Aufgaben erledigt`;

  document.getElementById("motivation").innerText =
    percent === 100 && total > 0
      ? "🎉 Alle Aufgaben erledigt! Stark!"
      : "";
}

function save() {
  localStorage.setItem("tasks", JSON.stringify(tasks));
}

function resetTasks() {
  tasks.forEach(task => task.done = false);
  save();
}

function saveDay() {
  if (tasks.length === 0) return;

  const allDone = tasks.every(task => task.done);
  history[lastDate] = allDone;
  localStorage.setItem("history", JSON.stringify(history));
}

function updateStats() {
  const days = Object.keys(history).length;
  const successDays = Object.values(history).filter(v => v).length;

  document.getElementById("stats").innerText =
    `${successDays} von ${days} Tagen vollständig geschafft`;

  document.getElementById("streak").innerText =
    `Aktuelle Serie: ${calculateStreak()} Tage`;
}

function calculateStreak() {
  const dates = Object.keys(history).sort().reverse();
  let streak = 0;
  for (let date of dates) {
    if (history[date]) streak++;
    else break;
  }
  return streak;
}

function exportCSV() {
  let csv = "Datum,Vollständig Erledigt\n";
  for (let date in history) {
    csv += `${date},${history[date]}\n`;
  }

  const blob = new Blob([csv], { type: "text/csv" });
  const url = URL.createObjectURL(blob);

  const a = document.createElement("a");
  a.href = url;
  a.download = "aufgaben_statistik.csv";
  a.click();
}

function toggleDarkMode() {
  document.body.classList.toggle("dark");
}

render();
</script>

</body>
</html>
