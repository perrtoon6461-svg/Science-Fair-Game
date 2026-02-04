# Science-Fair-Game
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Plant Pressure Experiment</title>

<style>
body {
  font-family: Arial, sans-serif;
  background: #e8f5e9;
  text-align: center;
}

.hidden { display: none; }

.box {
  background: white;
  width: 360px;
  margin: 20px auto;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.2);
}

button {
  width: 90%;
  padding: 10px;
  margin: 5px;
  font-size: 16px;
}

input {
  width: 90%;
  padding: 8px;
  margin: 6px;
}

.plant {
  font-size: 60px;
  animation: grow 2s infinite alternate;
}

@keyframes grow {
  from { transform: scale(1); }
  to { transform: scale(1.25); }
}

.star {
  font-size: 30px;
  cursor: pointer;
  color: gray;
}
.star.selected { color: gold; }

table {
  width: 100%;
  border-collapse: collapse;
}
td, th {
  border: 1px solid #aaa;
  padding: 6px;
}
</style>
</head>

<body>

<!-- START -->
<div class="box" id="startBox">
  <h2>🌱 Plant Experiment</h2>
  <input id="name" placeholder="Nickname (optional)">
  <input id="className" placeholder="Class (required)">
  <button onclick="startGame()">Start</button>
</div>

<!-- GAME -->
<div class="box hidden" id="gameBox">
  <div class="plant">🌱</div>
  <h3 id="question"></h3>
  <div id="choices"></div>
</div>

<!-- 7 DAYS -->
<div class="box hidden" id="daysLater">
  <h2>⏳ 7 Days Later…</h2>
  <div class="plant">🌿</div>
  <button onclick="showResult()">View Result</button>
</div>

<!-- RESULT -->
<div class="box hidden" id="resultBox">
  <h3 id="resultText"></h3>

  <p>⭐ Rate the experiment</p>
  <div id="stars"></div>

  <button onclick="finish()">Finish</button>
</div>

<!-- ADMIN -->
<div class="box hidden" id="adminBox">
  <h2>🔐 Admin Panel</h2>
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Class</th>
        <th>Water</th>
        <th>PSI</th>
        <th>Rating</th>
        <th>Time</th>
        <th>❌</th>
      </tr>
    </thead>
    <tbody id="dataTable"></tbody>
  </table>

  <button onclick="deleteAll()">Delete All</button>
  <button onclick="location.reload()">Exit</button>
</div>

<script>
/* PASSWORDS */
const ADMIN_PASS = "admin123";
const DELETE_PASS = "delete456";

/* DATA */
let current = { water:"", psi:"", rating:0 };

/* STAR RATING */
const starsDiv = document.getElementById("stars");
for (let i = 1; i <= 10; i++) {
  let s = document.createElement("span");
  s.innerHTML = "★";
  s.className = "star";
  s.onclick = () => setRating(i * 0.5);
  starsDiv.appendChild(s);
}

function setRating(val) {
  current.rating = val;
  document.querySelectorAll(".star").forEach((s, i) => {
    s.classList.toggle("selected", i < val * 2);
  });
}

/* START */
function startGame() {
  if (!className.value) return alert("Please enter your class");
  if (!name.value) name.value = "Anonymous";

  startBox.classList.add("hidden");
  gameBox.classList.remove("hidden");
  waterStep();
}

/* STEPS */
function waterStep() {
  question.innerText = "Step 1: Choose water amount";
  choices.innerHTML = `
    <button onclick="pickWater('50')">50 mL</button>
    <button onclick="pickWater('80')">80 mL</button>
    <button onclick="pickWater('120')">120 mL</button>`;
}

function pickWater(w) {
  current.water = w;
  psiStep();
}

function psiStep() {
  question.innerText = "Step 2: Choose air pressure";
  choices.innerHTML = `
    <button onclick="pickPsi('0')">0 PSI</button>
    <button onclick="pickPsi('5')">5 PSI</button>
    <button onclick="pickPsi('10')">10 PSI</button>`;
}

function pickPsi(p) {
  current.psi = p;
  gameBox.classList.add("hidden");
  daysLater.classList.remove("hidden");
}

/* RESULT */
function showResult() {
  daysLater.classList.add("hidden");
  resultBox.classList.remove("hidden");

  let text = "🌱 Plant survived.";
  if (current.water !== "80") {
    text = current.water === "120"
      ? "❌ The water overflowed and the plant drowned."
      : "❌ Too little water slowed the plant’s growth.";
  } else if (current.psi !== "0") {
    text = "⚠️ The plant survived but did not reach its maximum height due to air pressure.";
  }

  resultText.innerText = text;
}

/* SAVE */
function finish() {
  let data = JSON.parse(localStorage.getItem("records") || "[]");
  data.push({
    name: name.value,
    className: className.value,
    water: current.water,
    psi: current.psi,
    rating: current.rating,
    time: new Date().toLocaleTimeString()
  });
  localStorage.setItem("records", JSON.stringify(data));
  location.reload();
}

/* ADMIN SHORTCUT */
document.addEventListener("keydown", e => {
  if (e.ctrlKey && e.shiftKey && e.key.toLowerCase() === "a") {
    let p = prompt("Admin password:");
    if (p === ADMIN_PASS) showAdmin();
  }
});

function showAdmin() {
  startBox.classList.add("hidden");
  gameBox.classList.add("hidden");
  resultBox.classList.add("hidden");
  adminBox.classList.remove("hidden");
  loadTable();
}

function loadTable() {
  let data = JSON.parse(localStorage.getItem("records") || "[]");
  dataTable.innerHTML = "";
  data.forEach((r,i) => {
    dataTable.innerHTML += `
      <tr>
        <td>${r.name}</td>
        <td>${r.className}</td>
        <td>${r.water}</td>
        <td>${r.psi}</td>
        <td>${r.rating}</td>
        <td>${r.time}</td>
        <td><button onclick="deleteOne(${i})">X</button></td>
      </tr>`;
  });
}

function deleteOne(i) {
  if (prompt("Delete password:") !== DELETE_PASS) return;
  let data = JSON.parse(localStorage.getItem("records"));
  data.splice(i,1);
  localStorage.setItem("records", JSON.stringify(data));
  loadTable();
}

function deleteAll() {
  if (prompt("Delete ALL password:") === DELETE_PASS) {
    localStorage.removeItem("records");
    loadTable();
  }
}
</script>

</body>
</html>
