html ↓
```html
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>SGF Animation Viewer</title>
<style>
  body { font-family: sans-serif; }
  #board {
    display: grid;
    gap: 2px;
    margin-top: 10px;
  }
  .cell {
    width: 40px;
    height: 40px;
    background: #f0d9b5;
    border: 1px solid #555;
    position: relative;
  }
  .stone {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    position: absolute;
    top: 4px;
    left: 4px;
  }
  .black { background: #000; }
  .white { background: #fff; border: 1px solid #000; }
  #controls { margin-top: 10px; }
</style>
</head>
<body>

<h2>SGF アニメーション再生（盤サイズ自動判定）</h2>

<p>ここに SGF をペースト：</p>
<textarea id="sgfInput" rows="8" cols="80"></textarea>
<br>
<button id="parseBtn">SGF を読み込む</button>

<div id="controls">
  <button id="playBtn">▶ 自動再生</button>
  <button id="stepBtn">⏭ 1手進める</button>
  <button id="resetBtn">⏹ リセット</button>
  <span id="info"></span>
</div>

<div id="board"></div>

<script>
let size = 19;
let moves = [];
let currentMoveIndex = 0;
let autoplayTimer = null;

// 盤を完全に作り直す
function buildBoard() {
  const boardEl = document.getElementById('board');
  boardEl.innerHTML = "";  // 古い盤を完全削除

  boardEl.style.gridTemplateColumns = `repeat(${size}, 40px)`;
  boardEl.style.gridTemplateRows = `repeat(${size}, 40px)`;

  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const cell = document.createElement('div');
      cell.className = 'cell';
      cell.dataset.x = x;
      cell.dataset.y = y;
      boardEl.appendChild(cell);
    }
  }
}

// SZ[n] を読む
function detectBoardSize(sgf) {
  const m = sgf.match(/SZ

\[(\d+)\]

/);
  if (m) return parseInt(m[1], 10);
  return 19;
}

// SGF の着手だけ読む
function parseSGF(sgf) {
  const moveRegex = /;(B|W)

\[([a-z]{0,2})\]

/g;
  const result = [];
  let match;

  while ((match = moveRegex.exec(sgf)) !== null) {
    const color = match[1];
    const coord = match[2];

    if (coord.length === 2) {
      const x = coord.charCodeAt(0) - 97;
      const y = coord.charCodeAt(1) - 97;
      result.push({ color, x, y, pass: false });
    } else {
      result.push({ color, pass: true });
    }
  }
  return result;
}

// 石を全部消す
function clearBoard() {
  document.querySelectorAll('.stone').forEach(s => s.remove());
}

// A1 表記
function coordToHuman(x, y) {
  return String.fromCharCode(65 + x) + (y + 1);
}

// 1手進める
function playMove(index) {
  if (index >= moves.length) return;

  const move = moves[index];
  const infoEl = document.getElementById('info');

  if (move.pass) {
    infoEl.textContent = `${index + 1}手目: ${move.color === 'B' ? '黒' : '白'} パス`;
    return;
  }

  const cell = document.querySelector(`.cell[data-x="${move.x}"][data-y="${move.y}"]`);
  if (!cell) return;

  const oldStone = cell.querySelector('.stone');
  if (oldStone) oldStone.remove();

  const stone = document.createElement('div');
  stone.className = 'stone ' + (move.color === 'B' ? 'black' : 'white');
  cell.appendChild(stone);

  infoEl.textContent =
    `${index + 1}手目: ${move.color === 'B' ? '黒' : '白'} (${coordToHuman(move.x, move.y)})`;
}

// 自動再生
function startAutoplay() {
  stopAutoplay();
  autoplayTimer = setInterval(() => {
    if (currentMoveIndex >= moves.length) {
      stopAutoplay();
      return;
    }
    playMove(currentMoveIndex++);
  }, 700);
}

function stopAutoplay() {
  if (autoplayTimer) {
    clearInterval(autoplayTimer);
    autoplayTimer = null;
  }
}

// SGF 読み込み
document.getElementById('parseBtn').addEventListener('click', () => {
  const sgfText = document.getElementById('sgfInput').value;

  size = detectBoardSize(sgfText);
  buildBoard();  // ← 盤を完全に作り直す

  moves = parseSGF(sgfText);
  currentMoveIndex = 0;
  clearBoard();
  stopAutoplay();

  document.getElementById('info').textContent =
    `読み込み完了: 盤サイズ ${size} 路 / 手数 ${moves.length} 手`;
});

// 再生ボタン
document.getElementById('playBtn').addEventListener('click', startAutoplay);

// 1手進める
document.getElementById('stepBtn').addEventListener('click', () => {
  if (currentMoveIndex < moves.length) playMove(currentMoveIndex++);
});

// リセット
document.getElementById('resetBtn').addEventListener('click', () => {
  clearBoard();
  currentMoveIndex = 0;
  stopAutoplay();
  document.getElementById('info').textContent = 'リセットしました';
});
</script>

</body>
</html>
```
