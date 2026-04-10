<script>
  import Wheel from '$lib/Wheel.svelte';
  import TodoList  from '$lib/TodoList.svelte';

  const COLORS = [
    '#B66AD9','#D96A95','#D96AD7','#916AD9','#D9716A','#D9B2D8'
  ];

  /** @type {string[]} */
  let todos = $state([]);

  /** @type {Wheel} */
  let wheel;

  let result = $state('');
  let showResult = $state(false);

  /** @param {string} winner */
  function handleResult(winner) {
    result = winner;
    showResult = true;
  }
</script>

<main>
  <h1>WHEEL OF FORTUNATE DECISIONS</h1>

  <div class="container">
    <Wheel bind:this={wheel} items={todos} onResult={handleResult} colors={COLORS} />

    <TodoList bind:todos colors={COLORS} />
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