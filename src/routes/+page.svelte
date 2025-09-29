<script>
  import { onMount } from "svelte";

  let number = null;
  let motionEnabled = false;

  // simple dice roll
  function rollDice() {
    number = Math.floor(Math.random() * 6) + 1;
  }

  function startListening() {
    function handleMotion(event) {
      const { x, y, z } = event.accelerationIncludingGravity;
      const now = Date.now();

      if (lastX === null && lastY === null && lastZ === null) {
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

    let lastX = null, lastY = null, lastZ = null;
    let lastTime = 0;

    window.addEventListener("devicemotion", handleMotion);

    return () => {
      window.removeEventListener("devicemotion", handleMotion);
    };
  }

  async function enableMotion() {
    try {
      if (typeof DeviceMotionEvent !== "undefined" &&
          typeof DeviceMotionEvent.requestPermission === "function") {
        // iOS needs explicit permission
        const resp = await DeviceMotionEvent.requestPermission();
        if (resp !== "granted") {
          alert("Motion permission denied");
          return;
        }
      }
      startListening();
      motionEnabled = true;
    } catch (err) {
      console.error("Enable motion error", err);
    }
  }

  onMount(() => {
    // On non-iOS devices, start listening immediately
    if (!(typeof DeviceMotionEvent !== "undefined" &&
          typeof DeviceMotionEvent.requestPermission === "function")) {
      enableMotion();
    }
  });
</script>

<main class="p-4 text-center">
  <h1 class="text-2xl font-bold">Shake to Roll 🎲</h1>

  {#if !motionEnabled}
    <button
      class="mt-4 px-4 py-2 bg-blue-500 text-white rounded-xl"
      on:click={enableMotion}
    >
      Enable Motion
    </button>
  {/if}

  {#if number}
    <p class="text-3xl mt-4">{number}</p>
  {/if}
</main>
