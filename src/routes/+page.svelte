<script>
  import Wheel from '$lib/Wheel.svelte';
  import TodoList  from '$lib/TodoList.svelte';
  import Popup from '$lib/Popup.svelte';

  const COLORS = [
    '#B66AD9','#D96A95','#D96AD7','#916AD9','#D9716A','#D9B2D8'
  ];

  /** @type {string[]} */
  let todos = $state([]);

  let result = $state('');
  let showResult = $state(false);

  /** @param {string} winner */
  function handleResult(winner) {
    result = winner;
    showResult = true;
  }
</script>

<main>
  <div class="header">
    <h1>WHEEL OF FORTUNATE DECISIONS</h1>
    <p>Add your todos and let the wheel decide what you'll do next.</p>
  </div>

  <div class="container">
    <Wheel items={todos} onResult={handleResult} colors={COLORS} />

    <TodoList bind:todos={todos} colors={COLORS} />
  </div>

  <Popup bind:showPopup={showResult} smallHeading="AND THE WINNER IS..." mainContent={result} />
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
  }
  h1 {
    font-size: 2.5rem;
    text-shadow: 0 0 20px rgb(0, 255, 161, 0.8);
    color: #00ffa1;
    letter-spacing: 3px;
  }
  .header {
    margin-bottom: 30px;
     text-align: center;
  }
  .container {
    display: flex;
    flex-wrap: wrap;
    gap: 40px;
    align-items: center;
    justify-content: center;
  }
</style>