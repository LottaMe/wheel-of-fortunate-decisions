<script>
  import { onMount } from 'svelte';

  const COLORS = [
    '#B66AD9','#D96A95','#D96AD7','#916AD9','#D9716A','#D9B2D8'
  ];

  /** @type {string[]} */
  let items = $state([]);

  /** @type {HTMLCanvasElement} */
  let canvas;

  let newItem = $state('');
  let rotation = $state(0);
  let spinning = $state(false);
  let result = $state('');
  let showResult = $state(false);

  onMount(() => {
    drawWheel();
  });

  function drawWheel() {
    const ctx = /** @type {CanvasRenderingContext2D} */ (canvas.getContext('2d'));
    const cx = canvas.width / 2;
    const cy = canvas.height / 2;
    const radius = canvas.width / 2 - 4;

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

      ctx.beginPath();
      ctx.moveTo(cx, cy);
      ctx.arc(cx, cy, radius, start, end);
      ctx.closePath();
      ctx.fillStyle = COLORS[i % COLORS.length];
      ctx.fill();
      ctx.strokeStyle = 'rgba(0, 162, 103, 0.557)';
      ctx.lineWidth = 2;
      ctx.stroke();

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
      const label = item.length > maxLen ? item.slice(0, maxLen) + '\u2026' : item;
      ctx.fillText(label, radius - 14, 0);
      ctx.restore();
    });

    ctx.beginPath();
    ctx.arc(cx, cy, radius, 0, Math.PI * 2);
    ctx.strokeStyle = 'rgba(0, 162, 103, 0.557)';
    ctx.lineWidth = 5;
    ctx.stroke();
  }

  function addItem() {
    const val = newItem.trim();
    if (!val) return;
    items = [...items, val];
    newItem = '';
    drawWheel();
  }

  /** @param {number} i */
  function removeItem(i) {
    items = items.filter((_, idx) => idx !== i);
    drawWheel();
  }

  /** @param {KeyboardEvent} e */
  function handleKeydown(e) {
    if (e.key === 'Enter') addItem();
  }

  function spin() {
    if (spinning || items.length < 2) return;
    spinning = true;

    const totalSpins = 6 + Math.random() * 6;
    const extra = Math.random() * Math.PI * 2;
    const targetRotation = rotation + totalSpins * Math.PI * 2 + extra;
    const duration = 4000 + Math.random() * 2000;
    const startTime = performance.now();
    const startRot = rotation;

    /** @param {number} t */
    function easeOut(t) {
      return 1 - Math.pow(1 - t, 4);
    }

    /** @param {DOMHighResTimeStamp} now */
    function frame(now) {
      const elapsed = now - startTime;
      const t = Math.min(elapsed / duration, 1);
      rotation = startRot + (targetRotation - startRot) * easeOut(t);
      drawWheel();
      if (t < 1) {
        requestAnimationFrame(frame);
      } else {
        rotation = targetRotation % (Math.PI * 2);
        spinning = false;
        announceResult();
      }
    }

    requestAnimationFrame(frame);
  }

  function announceResult() {
    const arc = (Math.PI * 2) / items.length;
    const norm = ((rotation % (Math.PI * 2)) + Math.PI * 2) % (Math.PI * 2);
    const pointerAngle = (Math.PI * 2 - norm) % (Math.PI * 2);
    const index = Math.floor(pointerAngle / arc) % items.length;
    result = items[index];
    showResult = true;
  }
</script>

<main>
  <h1>🎡 WHEEL OF FORTUNE</h1>

  <div class="container">
    <div class="wheel-section">
      <div class="wheel-wrapper">
        <canvas bind:this={canvas} width="420" height="420"></canvas>
        <div class="pointer"></div>
        <div class="center-circle"></div>
      </div>
      <button class="spin-btn" onclick={spin} disabled={spinning || items.length < 2}>
        SPIN!
      </button>
    </div>

    <div class="controls">
      <h2>⚙ MANAGE ITEMS</h2>
      <div class="input-row">
        <input
          type="text"
          bind:value={newItem}
          onkeydown={handleKeydown}
          placeholder="Add an item..."
          maxlength="30"
        />
        <button onclick={addItem}>+</button>
      </div>
      <div class="items-list">
        {#if items.length === 0}
          <div class="empty-note">No items yet. Add some above!</div>
        {:else}
          {#each items as item, i}
            <div class="item-tag" style="background: {COLORS[i % COLORS.length]}">
              <span>{item}</span>
              <button onclick={() => removeItem(i)} title="Remove">✕</button>
            </div>
          {/each}
        {/if}
      </div>
    </div>
  </div>

  {#if showResult}
     <div
      class="result-banner"
      role="presentation"
      onclick={(e) => { if (e.target === e.currentTarget) showResult = false; }}
      onkeydown={(e) => { if ((e.key === 'Escape' || e.key === 'Enter') && e.target === e.currentTarget) showResult = false; }}
    >
      <div class="result-card">
        <p>🎉 The wheel has spoken!</p>
        <h2>{result}</h2>
        <button onclick={() => (showResult = false)}>Close</button>
      </div>
    </div>
  {/if}
</main>

<style>
  :global(*) { margin: 0; padding: 0; box-sizing: border-box; }
  :global(body) {
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
    min-height: 100vh;
    color: #f4f4f4;
  }

  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    padding: 20px;
  }

  h1 {
    font-size: 2.5rem;
    margin-bottom: 30px;
    text-shadow: 0 0 20px rgb(0, 255, 161, 0.8);
    color: #00ffa1;
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
    box-shadow: 0 0 40px rgba(0, 162, 103, 0.557), 0 0 80px rgba(0, 255, 30, 0.1);
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
    border-right: 44px solid rgba(0, 162, 103);
    filter: drop-shadow(0 0 8px rgba(0, 162, 103, 0.9));
    z-index: 10;
  }

  .center-circle {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 40px;
    height: 40px;
    background: radial-gradient(circle, #f4f4f4 0%, #00ffa1 100%);
    border-radius: 50%;
    box-shadow: 0 0 15px rgba(0, 162, 103, 0.9);
    z-index: 10;
  }

  .spin-btn {
    padding: 14px 50px;
    font-size: 1.2rem;
    font-weight: bold;
    background: linear-gradient(135deg, #00ffa1, #006eff);
    color: #1a1a2e;
    border: none;
    border-radius: 50px;
    cursor: pointer;
    letter-spacing: 2px;
    box-shadow: 0 4px 20px rgba(0, 162, 103, 0.557);
    transition: transform 0.1s, box-shadow 0.1s;
  }

  .spin-btn:hover:not(:disabled) {
    transform: scale(1.05);
    box-shadow: 0 6px 30px rgba(0, 162, 103, 0.557);
  }

  .spin-btn:disabled {
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
    color: #00ffa1;
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
    color: #f4f4f4;
    font-size: 0.95rem;
    outline: none;
    transition: border-color 0.2s;
  }

  .input-row input:focus { border-color: #00ffa1; }
  .input-row input::placeholder { color: rgba(255,255,255,0.4); }

  .input-row button {
    padding: 10px 16px;
    border-radius: 8px;
    border: none;
    background: linear-gradient(135deg, #00ffa1, #006eff);
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
  .items-list::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.4); border-radius: 2px; }

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

  .item-tag button:hover { color: #050505; }

  .empty-note {
    text-align: center;
    color: rgba(0, 162, 103, 0.557);
    font-size: 0.85rem;
    padding: 20px;
  }

  .result-banner {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    animation: fadeIn 0.3s ease;
  }

  .result-card {
    background: linear-gradient(135deg, #1a1a2e, #0f3460);
    border: 2px solid #00ffa1;
    border-radius: 20px;
    padding: 40px 60px;
    text-align: center;
    box-shadow: 0 0 60px rgba(0, 162, 103, 0.557);
    animation: popIn 0.4s cubic-bezier(0.34,1.56,0.64,1);
  }

  .result-card p { font-size: 1rem; color: rgba(255,255,255,0.7); margin-bottom: 10px; }

  .result-card h2 {
    font-size: 2rem;
    color: #00ffa1;
    margin-bottom: 24px;
    text-shadow: 0 0 20px rgba(0, 162, 103, 0.557);
  }

  .result-card button {
    padding: 12px 40px;
    border-radius: 50px;
    border: none;
    background: linear-gradient(135deg, #00ffa1, #006eff);
    color: #1a1a2e;
    font-weight: bold;
    font-size: 1rem;
    cursor: pointer;
    letter-spacing: 1px;
  }

  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
  @keyframes popIn { from { transform: scale(0.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }
</style>