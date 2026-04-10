<script>
    import { browser } from "$app/environment";
    import { onMount } from "svelte";

    const STORAGE_KEY = 'todos';
  /**
   * @typedef {Object} Props
   * @property {string[]} todos
   * @property {string[]} colors
   */

  /** @type {Props} */
  let { todos = $bindable(), colors } = $props();

  onMount(() => {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) todos = JSON.parse(saved);
  });

  $effect(() => {
    if (browser) localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  });
  
  let newTodo = $state('')

  function addTodo() {
    const todo = newTodo.trim()
    if (!todo) return
    todos = [...todos, todo]
    newTodo = ''
  }

  /** @param {number} i */
  function removeTodo(i) {
    todos = todos.filter((_, idx) => idx !== i);
  }

  /** @param {KeyboardEvent} e */
  function handleKeydown(e) {
    if (e.key === 'Enter') addTodo();
  }
</script>

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
      <div class="todo-tag" style="background: {colors[i % colors.length]}">
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

<style>
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
</style>