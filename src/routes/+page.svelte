<script>
  import { onMount } from "svelte";
  import { getDistance, getGreatCircleBearing, isPointWithinRadius } from "geolib";

  const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };
  const radiusMeters = 5;

  let userLat = null;
  let userLon = null;
  let watchId = null;

  // --- battery state
  let batteryLevel = null;
  let charging = null;

  // --- derived values
  $: distance = (userLat != null && userLon != null)
    ? getDistance({ latitude: userLat, longitude: userLon }, target)
    : null;

  $: bearing = (userLat != null && userLon != null)
    ? getGreatCircleBearing({ latitude: userLat, longitude: userLon }, target)
    : null;

  $: normBearing = bearing != null ? ((bearing % 360) + 360) % 360 : null;
  $: arrowRotation = normBearing ?? 0;

  $: insideCircle = (userLat != null && userLon != null)
    ? isPointWithinRadius({ latitude: userLat, longitude: userLon }, target, radiusMeters)
    : false;

  function updatePosition(pos) {
    userLat = pos.coords.latitude;
    userLon = pos.coords.longitude;
  }

  function geoError(err) {
    console.error("Geolocation error:", err);
  }

  onMount(() => {
    // --- watch geolocation
    if ("geolocation" in navigator) {
      watchId = navigator.geolocation.watchPosition(updatePosition, geoError, {
        enableHighAccuracy: true,
        maximumAge: 0,
        timeout: 10000
      });
    }

    // --- watch battery (Battery Status API)
    if ("getBattery" in navigator) {
      navigator.getBattery().then((battery) => {
        // set initial
        batteryLevel = battery.level;
        charging = battery.charging;

        // listen for changes
        battery.addEventListener("levelchange", () => {
          batteryLevel = battery.level;
        });
        battery.addEventListener("chargingchange", () => {
          charging = battery.charging;
        });
      });
    } else {
      console.warn("Battery API not supported on this browser.");
    }

    return () => {
      if (watchId) navigator.geolocation.clearWatch(watchId);
    };
  });
</script>

<main class="flex flex-col items-center justify-center min-h-screen p-6 bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="text-2xl font-bold mb-4">Navigation to Target</h1>

  <div class="mb-4">
    {#if userLat != null && userLon != null}
      <p>Distance: {distance} m</p>
      <p>Bearing: {bearing}°</p>
      <p>Status: {insideCircle ? "Inside 5 m" : "Outside 5 m"}</p>
    {:else}
      <p>Waiting for GPS…</p>
    {/if}
  </div>

  <!-- Arrow -->
  <div class="mb-4">
    <svg
      viewBox="0 0 64 64"
      width="96"
      height="96"
      class="transition-transform duration-300"
      style="transform: rotate({arrowRotation}deg); transform-origin: center; transform-box: fill-box;"
    >
      <path d="M32 6 L52 44 L36 44 L36 58 L28 58 L28 44 L12 44 Z" fill="currentColor" />
    </svg>
  </div>

  <!-- Battery info -->
  <div class="mt-4 p-2 rounded bg-gray-200 dark:bg-gray-800">
    {#if batteryLevel != null}
      <p>
        Battery: {Math.round(batteryLevel * 100)}% 
        {charging ? "⚡ (charging)" : ""}
      </p>
    {:else}
      <p>Battery status not supported.</p>
    {/if}
  </div>

  <!-- Circle indicator -->
  <div class="w-16 h-16 rounded-full mt-6" style="background-color: {insideCircle ? 'green' : 'red'};"></div>
</main>
