<script>
  const COLORS = [
    '#e74c3c','#e67e22','#f1c40f','#2ecc71','#1abc9c',
    '#3498db','#9b59b6','#e91e63','#00bcd4','#ff5722',
    '#8bc34a','#ff9800','#673ab7','#009688','#f44336'
  ];

  let items = ['Pizza','Burger','Sushi','Tacos','Pasta'];
  let rotation = 0;
  let spinning = false;
  let animFrame;

  const canvas = /** @type {HTMLCanvasElement} */ (document.getElementById('wheel'));

  const ctx = /** @type {CanvasRenderingContext2D} */ (canvas.getContext('2d'));
  const cx = canvas.width / 2;
  const cy = canvas.height / 2;
  const radius = canvas.width / 2 - 4;


  function drawWheel() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    if (items.length === 0) {
      ctx.fillStyle = 'rgba(255,255,255,0.05)';
      ctx.beginPath();
      ctx.arc(cx, cy, radius, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = 'rgba(255,255,255,0.3)';
      ctx.font = '18px Segoe UI';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('Add items to spin!', cx, cy);
      return;
    }
    const arc = (Math.PI * 2) / items.length;
    items.forEach((item, i) => {
      const start = rotation + i * arc;
      const end = start + arc;
      // Slice
      ctx.beginPath();
      ctx.moveTo(cx, cy);
      ctx.arc(cx, cy, radius, start, end);
      ctx.closePath();
      ctx.fillStyle = COLORS[i % COLORS.length];
      ctx.fill();
      ctx.strokeStyle = 'rgba(255,255,255,0.3)';
      ctx.lineWidth = 2;
      ctx.stroke();
      // Text
      ctx.save();
      ctx.translate(cx, cy);
      ctx.rotate(start + arc / 2);
      ctx.textAlign = 'right';
      ctx.fillStyle = '#fff';
      ctx.shadowColor = 'rgba(0,0,0,0.5)';
      ctx.shadowBlur = 4;
      const fontSize = Math.min(18, Math.max(10, 200 / items.length));
      ctx.font = `bold ${fontSize}px Segoe UI`;
      ctx.textBaseline = 'middle';
      const maxLen = 14;
      const label = item.length > maxLen ? item.slice(0, maxLen) + '…' : item;
      ctx.fillText(label, radius - 14, 0);
      ctx.restore();
    });
    // Outer ring
    ctx.beginPath();
    ctx.arc(cx, cy, radius, 0, Math.PI * 2);
    ctx.strokeStyle = 'rgba(255,215,0,0.6)';
    ctx.lineWidth = 5;
    ctx.stroke();
  }

  function renderList() {
    const list = document.getElementById('itemsList');
    if (!list) return
    if (items.length === 0) {
      list.innerHTML = '<div class="empty-note">No items yet. Add some above!</div>';
      return;
    }
    list.innerHTML = items.map((item, i) => `
      <div class="item-tag" style="background:${COLORS[i % COLORS.length]}">
        <span>${item}</span>
        <button onclick="removeItem(${i})" title="Remove">✕</button>
      </div>
    `).join('');
  }

  function addItem() {
    const input =/** @type {HTMLInputElement} */ (document.getElementById('itemInput'));
    if (!input) return

    const val = input.value.trim();
    if (!val) return;
    items.push(val);
    input.value = '';
    renderList();
    drawWheel();
  }

  /** @param {number} i */
  function removeItem(i) {
    items.splice(i, 1);
    renderList();
    drawWheel();
  }

  function spin() {
    if (spinning || items.length < 2) return;
    spinning = true;
    const spnBtn =/** @type {HTMLButtonElement} */ (document.getElementById('spinBtn'))
    spnBtn.disabled = true;

    const totalSpins = 6 + Math.random() * 6;
    const extra = Math.random() * Math.PI * 2;
    const targetRotation = rotation + totalSpins * Math.PI * 2 + extra;
    const duration = 4000 + Math.random() * 2000;
    const start = performance.now();
    const startRot = rotation;

    /** @param {number} t */
    function easeOut(t) {
      return 1 - Math.pow(1 - t, 4);
    }

    /** @param {DOMHighResTimeStamp} now */
    function frame(now) {
      const elapsed = now - start;
      const t = Math.min(elapsed / duration, 1);
      rotation = startRot + (targetRotation - startRot) * easeOut(t);
      drawWheel();
      if (t < 1) {
        animFrame = requestAnimationFrame(frame);
      } else {
        rotation = targetRotation % (Math.PI * 2);
        spinning = false;

        spnBtn.disabled = false;
        showResult();
      }
    }
    requestAnimationFrame(frame);
  }

  function showResult() {
    // Pointer is at right (0 radians from center). We find which slice is at angle 0.
    const arc = (Math.PI * 2) / items.length;
    // Normalize rotation
    const norm = ((rotation % (Math.PI * 2)) + Math.PI * 2) % (Math.PI * 2);
    // Pointer is at angle 0 (right side). Find which segment contains angle 0 relative to rotation.
    const pointerAngle = (Math.PI * 2 - norm) % (Math.PI * 2);
    const index = Math.floor(pointerAngle / arc) % items.length;
    const resultText = document.getElementById('resultText');
    const resultBanner = document.getElementById('resultBanner');

    if (!resultText || !resultBanner ) return

    resultText.textContent = items[index];
    resultBanner.style.display = 'flex';
  }

  function closeResult() {
    const resultBanner = document.getElementById('resultBanner');

    if ( !resultBanner ) return

    resultBanner.style.display = 'none';
  }

  renderList();
  drawWheel();
</script>
<h1>🎡 WHEEL OF FORTUNE</h1>
<div class="container">
  <div class="wheel-section">
    <div class="wheel-wrapper">
      <canvas id="wheel" width="420" height="420"></canvas>
      <div class="pointer"></div>
      <div class="center-circle"></div>
    </div>
    <button id="spinBtn" onclick={spin}>SPIN!</button>
  </div>
  <div class="controls">
    <h2>⚙ MANAGE ITEMS</h2>
    <div class="input-row">
      <input type="text" id="itemInput" placeholder="Add an item..." maxlength="30"> <!-- add on enter addItem -->
      <button onclick={addItem}>+</button>
    </div>
    <div class="items-list" id="itemsList"></div>
  </div>
</div>
<div id="resultBanner" class="result-banner" style="display:none;">
  <div class="result-card">
    <p>🎉 The wheel has spoken!</p>
    <h2 id="resultText"></h2>
    <button onclick={closeResult}>Close</button>
  </div>
</div>

<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #fff;
    padding: 20px;
  }
  h1 {
    font-size: 2.5rem;
    margin-bottom: 30px;
    text-shadow: 0 0 20px rgba(255,215,0,0.8);
    color: #ffd700;
    letter-spacing: 3px;
  }
  .container {
    display: flex;
    flex-wrap: wrap;
    gap: 40px;
    align-items: center;
    justify-content: center;
  }
  .wheel-section {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }
  .wheel-wrapper {
    position: relative;
    width: 420px;
    height: 420px;
  }
  canvas {
    border-radius: 50%;
    box-shadow: 0 0 40px rgba(255,215,0,0.4), 0 0 80px rgba(255,215,0,0.1);
  }
  .pointer {
    position: absolute;
    top: 50%;
    right: -22px;
    transform: translateY(-50%);
    width: 0;
    height: 0;
    border-top: 20px solid transparent;
    border-bottom: 20px solid transparent;
    border-right: 44px solid #ffd700;
    filter: drop-shadow(0 0 8px rgba(255,215,0,0.9));
    z-index: 10;
  }
  .center-circle {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 40px;
    height: 40px;
    background: radial-gradient(circle, #fff 0%, #ffd700 100%);
    border-radius: 50%;
    box-shadow: 0 0 15px rgba(255,215,0,0.9);
    z-index: 10;
  }
  #spinBtn {
    padding: 14px 50px;
    font-size: 1.2rem;
    font-weight: bold;
    background: linear-gradient(135deg, #ffd700, #ff8c00);
    color: #1a1a2e;
    border: none;
    border-radius: 50px;
    cursor: pointer;
    letter-spacing: 2px;
    box-shadow: 0 4px 20px rgba(255,215,0,0.4);
    transition: transform 0.1s, box-shadow 0.1s;
  }
  #spinBtn:hover:not(:disabled) {
    transform: scale(1.05);
    box-shadow: 0 6px 30px rgba(255,215,0,0.6);
  }
  #spinBtn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  .controls {
    background: rgba(255,255,255,0.05);
    border: 1px solid rgba(255,255,255,0.1);
    border-radius: 16px;
    padding: 24px;
    width: 280px;
    backdrop-filter: blur(10px);
  }
  .controls h2 {
    font-size: 1.1rem;
    margin-bottom: 16px;
    color: #ffd700;
    text-align: center;
    letter-spacing: 1px;
  }
  .input-row {
    display: flex;
    gap: 8px;
    margin-bottom: 16px;
  }
  .input-row input {
    flex: 1;
    padding: 10px 14px;
    border-radius: 8px;
    border: 1px solid rgba(255,255,255,0.2);
    background: rgba(255,255,255,0.08);
    color: #fff;
    font-size: 0.95rem;
    outline: none;
    transition: border-color 0.2s;
  }
  .input-row input:focus {
    border-color: #ffd700;
  }
  .input-row input::placeholder { color: rgba(255,255,255,0.4); }
  .input-row button {
    padding: 10px 16px;
    border-radius: 8px;
    border: none;
    background: linear-gradient(135deg, #ffd700, #ff8c00);
    color: #1a1a2e;
    font-weight: bold;
    font-size: 1.2rem;
    cursor: pointer;
    transition: transform 0.1s;
  }
  .input-row button:hover { transform: scale(1.1); }
  .items-list {
    max-height: 300px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  .items-list::-webkit-scrollbar { width: 4px; }
  .items-list::-webkit-scrollbar-track { background: transparent; }
  .items-list::-webkit-scrollbar-thumb { background: rgba(255,215,0,0.4); border-radius: 2px; }
  .item-tag {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 8px 12px;
    border-radius: 8px;
    font-size: 0.9rem;
    color: #1a1a2e;
    font-weight: 600;
  }
  .item-tag button {
    background: none;
    border: none;
    color: rgba(0,0,0,0.5);
    cursor: pointer;
    font-size: 1rem;
    line-height: 1;
    transition: color 0.2s;
  }
  .item-tag button:hover { color: #000; }
  .result-banner {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: rgba(0,0,0,0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    animation: fadeIn 0.3s ease;
  }
  .result-card {
    background: linear-gradient(135deg, #1a1a2e, #0f3460);
    border: 2px solid #ffd700;
    border-radius: 20px;
    padding: 40px 60px;
    text-align: center;
    box-shadow: 0 0 60px rgba(255,215,0,0.4);
    animation: popIn 0.4s cubic-bezier(0.34,1.56,0.64,1);
  }
  .result-card p { font-size: 1rem; color: rgba(255,255,255,0.7); margin-bottom: 10px; }
  .result-card h2 { font-size: 2rem; color: #ffd700; margin-bottom: 24px; text-shadow: 0 0 20px rgba(255,215,0,0.6); }
  .result-card button {
    padding: 12px 40px;
    border-radius: 50px;
    border: none;
    background: linear-gradient(135deg, #ffd700, #ff8c00);
    color: #1a1a2e;
    font-weight: bold;
    font-size: 1rem;
    cursor: pointer;
    letter-spacing: 1px;
  }
  .empty-note {
    text-align: center;
    color: rgba(255,255,255,0.3);
    font-size: 0.85rem;
    padding: 20px;
  }
  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
  @keyframes popIn { from { transform: scale(0.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }
</style>