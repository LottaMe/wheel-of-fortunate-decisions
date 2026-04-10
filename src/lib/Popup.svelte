<script>
  /**
   * @typedef {Object} Props
   * @property {boolean} showPopup
   * @property {string} smallHeading
   * @property {string} mainContent
   */

  /** @type {Props} */
  let { showPopup = $bindable(), smallHeading, mainContent } = $props();

</script>

{#if showPopup}
  <div
    class="popup-banner"
    role="presentation"
    onclick={(e) => { if (e.target === e.currentTarget) showPopup = false; }}
    onkeydown={(e) => { if ((e.key === 'Escape' || e.key === 'Enter') && e.target === e.currentTarget) showPopup = false; }}
  >
    <div class="popup-card">
      <p>{smallHeading}</p>
      <h2>{mainContent}</h2>
      <button onclick={() => (showPopup = false)}>Close</button>
    </div>
  </div>
{/if}

<style>
  .popup-banner {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    animation: fadeIn 0.3s ease;
  }

  .popup-card {
    background: linear-gradient(135deg, #1a1a2e, #0f3460);
    border: 2px solid #00ffa1;
    border-radius: 20px;
    padding: 40px 60px;
    text-align: center;
    box-shadow: 0 0 60px rgba(0, 162, 103, 0.557);
    animation: popIn 0.4s cubic-bezier(0.34,1.56,0.64,1);
  }

  .popup-card p { font-size: 1rem; color: rgba(255,255,255,0.7); margin-bottom: 10px; }

  .popup-card h2 {
    font-size: 2rem;
    color: #00ffa1;
    margin-bottom: 24px;
    text-shadow: 0 0 20px rgba(0, 162, 103, 0.557);
  }

  .popup-card button {
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