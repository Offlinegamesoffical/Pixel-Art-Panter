<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Pixel Art Painter</title>

<style>
body { font-family: Arial; text-align: center; }

#grid {
  display: grid;
  margin: 20px auto;
  justify-content: center;
}

.pixel {
  width: 10px;
  height: 10px;
  border: 1px solid #eee;
  cursor: pointer;
}

#saveList {
  margin-top: 10px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 5px;
}

.saveItem {
  padding: 5px 10px;
  border: 1px solid #aaa;
  cursor: pointer;
  background: #f5f5f5;
}

.saveItem:hover {
  background: #ddd;
}

#stats, #updateLog {
  margin: 10px auto;
  width: 350px;
  text-align: left;
  border: 1px solid #ccc;
  padding: 10px;
  background: #fafafa;
}
</style>

</head>
<body>

<h1>Pixel Cube Painter <span style="font-size:15px;color:#888;">1.0</span></h1>

<!-- TOOLBAR -->
<div id="toolbar">
<input type="text" id="imgUrl" placeholder="Image URL">
<button onclick="loadImage()">Load Image</button>
<button onclick="saveProgress()">Save</button>
<button onclick="judgePainting()">AI Judge</button>
<p id="judgeResult"></p>
</div>
<button onclick="loadRandomImage()">Random Image</button>

<!-- MODES -->
<br>

<h3>Tools</h3>

<input type="color" id="customColor" value="#ff0000">

<button onclick="enablePaintMode()">Enable Custom Paint</button>
<button onclick="disablePaintMode()">Disable Custom Paint</button>
<button onclick="enableEraser()">Eraser Mode</button>

<p id="paintModeStatus">Mode: Image Paint</p>

<p id="status"></p>

<!-- STATS -->
<div id="stats">
<h3>Stats</h3>
<div id="statText">No data yet</div>
</div>

<!-- UPDATE LOG -->
<div id="updateLog">
<h3>Update Log</h3>

<b>1.0!!!!! - BIG UPDATE!!!!</b><br><br>

1.0 is out!!!!! I hope you all enjoy the ai judge! oficially Offlinegames<br><br>

Thank You for painting with us

Pixel Cube Painter
© 2026 Offlinegames co. ©

This code may not be copied, modified, or redistributed without permission.
</div>

<!-- SAVES -->
<h3>Saves</h3>
<div id="saveList"></div>

<!-- GRID -->
<div id="grid"></div>
<canvas id="canvas" style="display:none;"></canvas>

<script>
let targetColors = [];
let currentState = [];
let width = 0;
let height = 0;
let currentSaveName = "default";

let paintMode = false;
let eraserMode = false;
function judgePainting() {
  if (!targetColors.length || !currentState.length) {
    document.getElementById("judgeResult").innerText = "No painting to judge.";
    return;
  }

  let score = 0;

  for (let i = 0; i < targetColors.length; i++) {
    const t = targetColors[i];
    const c = currentState[i];

    if (!t || !c) continue;

    const tr = parseInt(t.split('(')[1]);
    const tg = parseInt(t.split(',')[1]);
    const tb = parseInt(t.split(',')[2]);

    let cr, cg, cb;

    if (c.startsWith("#")) {
      cr = parseInt(c.substr(1,2), 16);
      cg = parseInt(c.substr(3,2), 16);
      cb = parseInt(c.substr(5,2), 16);
    } else if (c.startsWith("rgb")) {
      cr = parseInt(c.split('(')[1]);
      cg = parseInt(c.split(',')[1]);
      cb = parseInt(c.split(',')[2]);
    } else {
      continue;
    }

    const diff = Math.sqrt(
      (tr - cr) ** 2 +
      (tg - cg) ** 2 +
      (tb - cb) ** 2
    );

    const similarity = 1 - (diff / 441.67);
    score += similarity;
  }

  let finalScore = ((score / targetColors.length) * 100).toFixed(1);

  let rating = "";

  if (finalScore > 90) rating = "🔥 MASTERPIECE";
  else if (finalScore > 75) rating = "🎨 Great job!";
  else if (finalScore > 50) rating = "👍 Decent";
  else rating = "🧪 Abstract chaos";

  document.getElementById("judgeResult").innerText =
    `AI Score: ${finalScore}/100 - ${rating}`;
}
function loadRandomImage() {
  // Random seed so image changes every time
  const seed = Math.floor(Math.random() * 100000);

  // Using picsum (free random images)
  let url = `https://picsum.photos/seed/${seed}/300`;

  // Put it into your existing loader
  document.getElementById('imgUrl').value = url;

  loadImage();
}

// LOAD IMAGE
function loadImage() {
  let url = document.getElementById('imgUrl').value;
  url = "https://images.weserv.nl/?url=" + encodeURIComponent(url);

  const img = new Image();
  document.getElementById('status').innerText = "Loading...";

  img.crossOrigin = "anonymous";

  img.onload = function() {
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    const maxPixels = 2500;
    const scale = Math.min(
      Math.sqrt(maxPixels / (img.width * img.height)),
      1
    );

    width = Math.floor(img.width * scale);
    height = Math.floor(img.height * scale);

    canvas.width = width;
    canvas.height = height;

    ctx.drawImage(img, 0, 0, width, height);

    const data = ctx.getImageData(0, 0, width, height).data;

    targetColors = [];

    for (let i = 0; i < data.length; i += 4) {
      targetColors.push(`rgb(${data[i]},${data[i+1]},${data[i+2]})`);
    }

    currentSaveName = "save_" + Date.now();

    loadProgress();
    initGrid();
    renderSaveList();
    updateStats();

    document.getElementById('status').innerText =
      `Grid: ${width} x ${height}`;
  };

  img.onerror = function() {
    document.getElementById('status').innerText = "❌ Failed to load image.";
  };

  img.src = url;
}

// TOOLS
function enablePaintMode() {
  paintMode = true;
  eraserMode = false;
  document.getElementById("paintModeStatus").innerText = "Mode: Custom Paint";
}

function disablePaintMode() {
  paintMode = false;
  eraserMode = false;
  document.getElementById("paintModeStatus").innerText = "Mode: Image Paint";
}

function enableEraser() {
  eraserMode = true;
  paintMode = false;
  document.getElementById("paintModeStatus").innerText = "Mode: Eraser";
}

// GRID
function initGrid() {
  const grid = document.getElementById('grid');
  grid.innerHTML = '';
  grid.style.gridTemplateColumns = `repeat(${width}, 10px)`;

  for (let i = 0; i < width * height; i++) {
    const div = document.createElement('div');
    div.className = 'pixel';
    div.style.background = currentState[i];

    div.onclick = () => {
      let color;

      if (eraserMode) {
        color = "#ffffff";
      } else if (paintMode) {
        color = document.getElementById("customColor").value;
      } else {
        color = targetColors[i];
      }

      currentState[i] = color;
      div.style.background = color;

      saveProgress();
      checkComplete();
      updateStats();
    };

    grid.appendChild(div);
  }
}

// SAVE
function saveProgress() {
  let saves = JSON.parse(localStorage.getItem('saves')) || {};

  saves[currentSaveName] = {
    state: currentState,
    width,
    height,
    target: targetColors
  };

  localStorage.setItem('saves', JSON.stringify(saves));
  renderSaveList();
}

// LOAD SAVE
function loadSave(name) {
  let saves = JSON.parse(localStorage.getItem('saves')) || {};
  if (!saves[name]) return;

  currentSaveName = name;

  const save = saves[name];
  width = save.width;
  height = save.height;
  currentState = save.state;
  targetColors = save.target;

  initGrid();
  updateStats();

  document.getElementById('status').innerText =
    "Loaded: " + name;
}

// DELETE SAVE
function deleteSave(name) {
  let saves = JSON.parse(localStorage.getItem('saves')) || {};
  delete saves[name];
  localStorage.setItem('saves', JSON.stringify(saves));
  renderSaveList();
}

// SAVE LIST
function renderSaveList() {
  let saves = JSON.parse(localStorage.getItem('saves')) || {};
  let list = document.getElementById("saveList");

  list.innerHTML = "";

  Object.keys(saves).forEach(name => {
    let div = document.createElement("div");
    div.className = "saveItem";
    div.innerText = name;

    div.onclick = () => loadSave(name);

    div.oncontextmenu = (e) => {
      e.preventDefault();
      deleteSave(name);
    };

    list.appendChild(div);
  });
}

// LOAD PROGRESS
function loadProgress() {
  let saves = JSON.parse(localStorage.getItem('saves')) || {};

  if (saves[currentSaveName]) {
    currentState = saves[currentSaveName].state;
  } else {
    currentState = Array(width * height).fill('#ffffff');
  }
}

// STATS
function updateStats() {
  let total = width * height;
  let painted = currentState.filter(c => c !== "#ffffff").length;
  let percent = total ? ((painted / total) * 100).toFixed(1) : 0;

  document.getElementById("statText").innerHTML = `
    Save: ${currentSaveName}<br>
    Painted: ${painted} / ${total}<br>
    Completion: ${percent}%
  `;
}

// CHECK COMPLETE
function checkComplete() {
  if (currentState.every((c, i) => c === targetColors[i])) {
    document.getElementById('status').innerText = "✅ Completed!";
  }
}

// INIT
renderSaveList();
</script>

</body>
</html>
