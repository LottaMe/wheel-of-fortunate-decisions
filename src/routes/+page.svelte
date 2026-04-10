<script>
  import Wheel from '$lib/Wheel.svelte';

  const COLORS = [
    '#B66AD9','#D96A95','#D96AD7','#916AD9','#D9716A','#D9B2D8'
  ];

  /** @type {string[]} */
  let todos = $state([]);

  /** @type {Wheel} */
  let wheel;

  let newTodo = $state('');
  let result = $state('');
  let showResult = $state(false);

  function addTodo() {
    const val = newTodo.trim();
    if (!val) return;
    todos = [...todos, val];
    newTodo = '';
  }

  /** @param {number} i */
  function removeTodo(i) {
    todos = todos.filter((_, idx) => idx !== i);
  }

  /** @param {KeyboardEvent} e */
  function handleKeydown(e) {
    if (e.key === 'Enter') addTodo();
  }

  /** @param {string} winner */
  function handleResult(winner) {
    result = winner;
    showResult = true;
  }
</script>

<main>
  <h1>WHEEL OF FORTUNATE DECISIONS</h1>

  <div class="container">
    <div class="wheel-section">
      <Wheel bind:this={wheel} items={todos} onResult={handleResult} colors={COLORS} />
    </div>

    <div class="controls">
      <h2>MANAGE TODOS</h2>
      <div class="input-row">
        <input
          type="text"
          bind:value={newTodo}
          onkeydown={handleKeydown}
          placeholder="Add a todo..."
          maxlength="30"
        />
        <button onclick={addTodo}>+</button>
      </div>
      <div class="todos-list">
        {#each todos as todo, i}
          <div class="todo-tag" style="background: {COLORS[i % COLORS.length]}">
            <span>{todo}</span>
            <button onclick={() => removeTodo(i)} title="Remove">✕</button>
          </div>
        {/each}
        {#if todos.length === 0}
          <div class="empty-note">No todos yet. Add some above!</div>
        {:else if todos.length < 2}
          <div class="empty-note">Not enough todos yet, add at least 2 to spin!</div>
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

  .todos-list {
    max-height: 300px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .todos-list::-webkit-scrollbar { width: 4px; }
  .todos-list::-webkit-scrollbar-track { background: transparent; }
  .todos-list::-webkit-scrollbar-thumb { background: rgba(255,255,255,0.4); border-radius: 2px; }

  .todo-tag {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 8px 12px;
    border-radius: 8px;
    font-size: 0.9rem;
    color: #1a1a2e;
    font-weight: 600;
  }

  .todo-tag button {
    background: none;
    border: none;
    color: rgba(0,0,0,0.5);
    cursor: pointer;
    font-size: 1rem;
    line-height: 1;
    transition: color 0.2s;
  }

  .todo-tag button:hover { color: #050505; }

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