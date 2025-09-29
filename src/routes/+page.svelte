<script>
  import { onMount } from "svelte";

  let number = null;

  // simple dice roll
  function rollDice() {
    number = Math.floor(Math.random() * 6) + 1;
  }

  onMount(() => {
    let lastX, lastY, lastZ;
    let lastTime = 0;

    function handleMotion(event) {
      const { x, y, z } = event.accelerationIncludingGravity;
      const now = Date.now();

      if (!lastX && !lastY && !lastZ) {
        lastX = x; lastY = y; lastZ = z;
        return;
      }

      const delta = Math.abs(x - lastX) + Math.abs(y - lastY) + Math.abs(z - lastZ);

      if (delta > 15 && now - lastTime > 1000) { 
        rollDice();
        lastTime = now;
      }

      lastX = x; lastY = y; lastZ = z;
    }

    window.addEventListener("devicemotion", handleMotion);

    return () => {
      window.removeEventListener("devicemotion", handleMotion);
    };
  });
</script>

<main class="p-4 text-center">
  <h1 class="text-2xl font-bold">Shake to Roll 🎲</h1>
  {#if number}
    <p class="text-3xl mt-4">{number}</p>
  {/if}
</main>
