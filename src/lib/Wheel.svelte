<!-- lib/Wheel.svelte -->
<script>
  import { onMount } from 'svelte';

  /**
   * @typedef {Object} Props
   * @property {string[]} items
   * @property {(result: string) => void} onResult
   * @property {string[]} colors
   */

  /** @type {Props} */
  let { items, onResult, colors } = $props();

  /** @type {HTMLCanvasElement} */
  let canvas;

  let rotation = $state(0);
  let spinning = $state(false);

  onMount(() => drawWheel());

  $effect(() => {
    // Redraw whenever items change
    items;
    if (canvas) drawWheel();
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
      ctx.fillStyle = colors[i % colors.length];
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
    onResult(items[index]);
  }
</script>

<div class="wheel-section">
  <div class="wheel-wrapper" onclick={spin} onkeydown={spin} role="button" tabindex="0">
    <canvas bind:this={canvas} width="420" height="420"></canvas>
    <div class="pointer"></div>
    <div class="center-circle"></div>
  </div>
</div>


<style>
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
</style>
