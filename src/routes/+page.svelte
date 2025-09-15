<script>
    
  let latitude = "";
  let longitude = "";
  let accuracy = "";
  let errorMessage = "";

  function showPosition(position) {
    latitude = position.coords.latitude;
    longitude = position.coords.longitude;
    accuracy = position.coords.accuracy;
    errorMessage = "";
  }

  function showError(error) {
    errorMessage = error.message;
  }

  function findPosition() {
    if ("geolocation" in navigator) {
      navigator.geolocation.getCurrentPosition(showPosition, showError);
    }else{
        error: "try again later."
    }
    
  }

</script>

<div class="p-4">
  <button on:click={findPosition} class="px-4 py-2 bg-blue-500 text-white rounded">
    find position
  </button>

  {#if latitude}
    <p>Latitude: {latitude}</p>
    <p>Longitude: {longitude}</p>
    <p>Accuracy: {accuracy}</p>
  {/if}

  {#if errorMessage}
    <p class="text-red-500">Fehler: {errorMessage}</p>
  {/if}
</div>
