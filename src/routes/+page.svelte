<script>
  import { onMount } from "svelte";
  import { getDistance, getGreatCircleBearing, isPointWithinRadius } from "geolib";

  const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };
  const radiusMeters = 5;

  let userLat = null;
  let userLon = null;
  let watchId = null;

  let insideCircle = false; // current state
  let lastInsideCircle = false; // track previous state

  function updatePosition(pos) {
    userLat = pos.coords.latitude;
    userLon = pos.coords.longitude;
  }

  function geoError(err) {
    console.error("Geolocation error:", err);
  }

  onMount(() => {
    if ("geolocation" in navigator) {
      watchId = navigator.geolocation.watchPosition(updatePosition, geoError, {
        enableHighAccuracy: true,
        maximumAge: 0,
        timeout: 10000
      });
    }

    return () => {
      if (watchId) navigator.geolocation.clearWatch(watchId);
    };
  });

  // --- derived values
  $: distance = (userLat != null && userLon != null)
    ? getDistance({ latitude: userLat, longitude: userLon }, target)
    : null;

  $: bearing = (userLat != null && userLon != null)
    ? getGreatCircleBearing({ latitude: userLat, longitude: userLon }, target)
    : null;

  // Adjust for ➤ emoji (points right by default)
  $: arrowRotation = bearing != null ? (bearing - 90 + 360) % 360 : 0;

  // insideCircle detection
  $: {
    if (userLat != null && userLon != null) {
      insideCircle = isPointWithinRadius({ latitude: userLat, longitude: userLon }, target, radiusMeters);

      // 🔔 vibrate when status changes
      if (insideCircle !== lastInsideCircle) {
        if ("vibrate" in navigator) {
          if (insideCircle) {
            navigator.vibrate([200, 100, 200]); // short buzz pattern when entering
          } else {
            navigator.vibrate(500); // long buzz when leaving
          }
        }
        lastInsideCircle = insideCircle;
      }
    }
  }

  // test button vibration
  function testVibrate() {
    if ("vibrate" in navigator) {
      navigator.vibrate([100, 50, 100, 50, 200]);
    } else {
      alert("Vibration API not supported on this device/browser.");
    }
  }
</script>

<main class="flex flex-col items-center justify-center min-h-screen p-6 bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="text-2xl font-bold mb-4">Navigation to Target</h1>

  <div class="mb-4 text-center">
    {#if userLat != null && userLon != null}
      <p>Latitude: {userLat}</p>
      <p>Longitude: {userLon}</p>
      <p>Distance: {distance} m</p>
      <p>Bearing: {bearing}°</p>
      <p>Status: {insideCircle ? "Inside 5 m radius" : "Outside radius"}</p>
    {:else}
      <p>Waiting for GPS… (allow location permission)</p>
    {/if}
  </div>

  <!-- Arrow (emoji) -->
  <div class="mb-6">
    <div
      class="text-6xl transition-transform duration-300"
      style="transform: rotate({arrowRotation}deg);"
    >
      ➤
    </div>
    <p class="text-sm text-gray-500 mt-2">(arrow points toward target)</p>
  </div>

  <!-- Test button for vibration -->
  <button
    on:click={testVibrate}
    class="px-4 py-2 rounded bg-blue-600 text-white hover:bg-blue-700 mb-4"
  >
    Test Vibration
  </button>

  <!-- Circle indicator -->
  <div class="w-16 h-16 rounded-full" style="background-color: {insideCircle ? 'green' : 'red'};"></div>
</main>
