<script>
  import { onMount } from "svelte";
  import { getDistance, getGreatCircleBearing, isPointWithinRadius } from "geolib";

  const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };
  const radiusMeters = 5;

  let userLat = null;
  let userLon = null;
  let watchId = null;

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
    } else {
      console.error("Geolocation not available");
    }

    return () => {
      if (watchId) navigator.geolocation.clearWatch(watchId);
    };
  });

  // --- reactive derived values (Svelte 4 style)
  $: distance = (userLat != null && userLon != null)
    ? getDistance({ latitude: userLat, longitude: userLon }, target)
    : null;

  $: bearing = (userLat != null && userLon != null)
    ? getGreatCircleBearing({ latitude: userLat, longitude: userLon }, target)
    : null;

  // normalize bearing to 0..359.999
  $: normBearing = bearing != null ? ((bearing % 360) + 360) % 360 : null;

  // we use an SVG arrow that points UP (north) by default,
  // so rotation = bearing (north-based) works directly
  $: arrowRotation = normBearing ?? 0;

  $: insideCircle = (userLat != null && userLon != null)
    ? isPointWithinRadius({ latitude: userLat, longitude: userLon }, target, radiusMeters)
    : false;
</script>

<main class="flex flex-col items-center justify-center min-h-screen p-6 bg-gray-50 dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <h1 class="text-2xl font-bold mb-4">Navigation to Target</h1>

  <div class="mb-4">
    {#if userLat != null && userLon != null}
      <p>Distance: {distance} m</p>
      <p>Bearing (to target): {bearing}°</p>
      <p>Normalized bearing: {normBearing}°</p>
      <p>Status: {insideCircle ? "Inside 5 m" : "Outside 5 m"}</p>
    {:else}
      <p>Waiting for GPS… (allow location permission)</p>
    {/if}
  </div>

  <!-- SVG arrow points UP by default. We rotate it by arrowRotation (degrees from north). -->
  <div class="mb-4">
    <svg
      viewBox="0 0 64 64"
      width="96"
      height="96"
      class="transition-transform duration-300"
      style="transform: rotate({arrowRotation}deg); transform-origin: center; transform-box: fill-box;"
      aria-hidden="true"
    >
      <!-- simple arrow pointing up -->
      <path d="M32 6 L52 44 L36 44 L36 58 L28 58 L28 44 L12 44 Z" fill="currentColor" />
    </svg>
    <div class="text-sm text-gray-500 mt-2">Arrow points toward the target (north = 0°)</div>
  </div>

  <div class="w-16 h-16 rounded-full" style="background-color: {insideCircle ? 'green' : 'red'};"></div>
</main>

<style>
  main { font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
</style>
