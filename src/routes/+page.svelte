<script>
  import { onMount } from 'svelte';

  let number = null;
  let enabled = false;
  let status = 'idle'; // 'idle' | 'listening' | 'permission-denied' | 'error'
  let last = { x: null, y: null, z: null };
  let lastTime = 0;

  const THRESHOLD = 18;   // increase to make shake harder, decrease to make it more sensitive
  const COOLDOWN = 1000;  // milliseconds

  function rollDice() {
    number = Math.floor(Math.random() * 6) + 1;
  }

  function handleMotion(event) {
    const acc = event.accelerationIncludingGravity || event.acceleration;
    if (!acc) return;

    const x = acc.x ?? 0;
    const y = acc.y ?? 0;
    const z = acc.z ?? 0;
    const now = Date.now();

    // initialize on first non-null reading
    if (last.x === null) {
      last = { x, y, z };
      return;
    }

    const delta = Math.abs(x - last.x) + Math.abs(y - last.y) + Math.abs(z - last.z);

    // uncomment for debugging:
    // console.log('delta', delta);

    if (delta > THRESHOLD && now - lastTime > COOLDOWN) {
      rollDice();
      lastTime = now;
    }

    last = { x, y, z };
  }

  async function enableMotion() {
    try {
      // iOS requires explicit request
      if (typeof DeviceMotionEvent !== 'undefined' &&
          typeof DeviceMotionEvent.requestPermission === 'function') {
        const resp = await DeviceMotionEvent.requestPermission();
        if (resp !== 'granted') {
          status = 'permission-denied';
          return;
        }
      }

      window.addEventListener('devicemotion', handleMotion);
      enabled = true;
      status = 'listening';
    } catch (err) {
      console.error('enableMotion error', err);
      status = 'error';
    }
  }

  onMount(() => {
    // cleanup on destroy
    return () => {
      window.removeEventListener('devicemotion', handleMotion);
    };
  });
</script>

<main style="font-family: system-ui, sans-serif; padding: 1.25rem; text-align:center;">
  <h1 style="font-size:1.5rem; margin-bottom:0.5rem;">Shake to Roll 🎲</h1>

  <div style="margin:1rem 0;">
    <button on:click={enableMotion} disabled={enabled} style="padding:0.5rem 1rem;">
      {enabled ? 'Motion enabled' : 'Enable Motion (required on iOS)'}
    </button>

    <button on:click={rollDice} style="padding:0.5rem 1rem; margin-left:0.5rem;">
      Roll (desktop/test)
    </button>
  </div>

  <div style="font-size: 2.5rem; margin-top: 1rem;">
    {#if number}
      <strong>{number}</strong>
    {:else}
      <span style="opacity:0.6;">—</span>
    {/if}
  </div>

  <p style="margin-top:1rem; color:#666;">
    Status: {status} — Try shaking your phone. (iOS: tap *Enable Motion* then shake)
  </p>
</main>
