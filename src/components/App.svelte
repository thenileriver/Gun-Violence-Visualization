<script>
  import Map from './Map.svelte';
</script>

<main>
  <h1>US Gun Law Visualization</h1>
  <div class="map-container">
    <Map/>
  </div>
</main>

<style>
  /* Basic reset and styling */
  body {
    margin: 0;
    font-family: 'Arial', sans-serif;
    background: linear-gradient(to right, #2c3e50, #bdc3c7); /* Example gradient background */
    color: #fff;
  }

  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    padding: 20px;
  }

  h1 {
    font-size: 3rem; /* Larger font size */
    font-family: 'Georgia', serif; /* Change font family */
    margin: 20px 0;
    text-shadow: 2px 2px #000; /* Add a text shadow for effect */
  }

  .map-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 20px;
    width: 100%;
  }

  svg {
    max-width: 100%; /* Ensure the map is responsive */
    height: auto;
  }
</style>