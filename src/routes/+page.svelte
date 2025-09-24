<script>
    import { onMount } from "svelte";
    import { getDistance, getGreatCircleBearing, isPointWithinRadius } from "geolib";

    const target = { latitude: 42.05913363079434, longitude: 19.51725368912721 };
    const radiusMeters = 5;

    let userLat = null;
    let userLon = null;
    let distance = null;
    let bearing = null;
    let insideCircle = false;

    let watchId;

    function updatePosition(pos) {
        userLat = pos.coords.latitude;
        userLon = pos.coords.longitude;

        distance = getDistance(
            { latitude: userLat, longitude: userLon },
            target
        );

        bearing = getGreatCircleBearing(
            { latitude: userLat, longitude: userLon },
            target
        );

        insideCircle = isPointWithinRadius(
            { latitude: userLat, longitude: userLon },
            target,
            radiusMeters
        );
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

    // 🔄 Reactive arrow rotation (Svelte 4 style)
    $: arrowRotation = (bearing ?? 0) % 360;
</script>

<main>
<div class="flex flex-col items-center justify-center min-h-screen bg-gray-50 dark:bg-gray-900 text-gray-800 dark:text-gray-100">
    <h1 class="text-2xl font-bold mb-4">Navigation to Target</h1>

    <div class="mb-6">
        {#if userLat && userLon}
            <p>Distance to target: {distance} m</p>
            <p>Bearing to target: {bearing}°</p>
            <p>Status:
                {insideCircle
                    ? "Inside 5 m radius"
                    : "Outside radius"}
            </p>
        {:else}
            <p>Waiting for GPS signal…</p>
        {/if}
    </div>

    <!-- ✅ Arrow that rotates -->
    <div class="mb-6">
        <div
            class="text-5xl transition-transform duration-200"
            style="transform: rotate({arrowRotation}deg);"
        >
            ➤
        </div>
        <p class="text-sm text-gray-500">(arrow points toward target)</p>
    </div>

    <!-- Circle indicator -->
    <div
        class="w-16 h-16 rounded-full"
        style="background-color: {insideCircle ? 'green' : 'red'};"
    ></div>
</div>
</main>
