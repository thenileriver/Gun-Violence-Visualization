<script>
  import Map from './Map.svelte';

  let gunTypes = ['All', 'Handgun', 'Shotgun', 'Rifle', 'Automatic Rifle', 'Semiautomatic Rifle', 'Unknown'];
  let selectedGunType = 'All';
  let mapInformation = `This map helps us visualize the strength of gun safety laws and their performance on gun violence. Each state is labeled with the rating of their gun laws, with 'A' being strong and 'F' being weak. You can filter through different types of guns, states being labeled in red being top 5 in that selected category by percentage, and hovering over a state will give you detailed information.`;
</script>

<main>
  <h1>US Gun Law Visualization (2013 - 2018)</h1>
  <div class="container">
    <div class="dropdown">
      <label for="gunType">Select Gun Type by % Used:</label>
      <select id="gunType" bind:value={selectedGunType}>
        {#each gunTypes as gunType}
          <option value="{gunType}">{gunType}</option>
        {/each}
      </select>
    </div>
    <div class="map-container">
      <Map {selectedGunType} />
    </div>
    <div class="sidebar">
      <label for="info">Map Information:</label>
      <textarea id="info" name="info" rows="6" cols="50" readonly style="resize: none;">{mapInformation}</textarea>
    </div>
  </div>
</main>

<style>
  /* Basic reset and styling */
  body {
    margin: 0;
    font-family: 'Arial', sans-serif;
    background: linear-gradient(to right, #2c3e50, #bdc3c7);
    color: #fff;
    height: 100vh;
    overflow: hidden; /* Prevent scrolling */
  }

  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;
    padding: 2vh 2vw;
    height: 100%;
    box-sizing: border-box;
  }

  h1 {
    font-size: 3vw;
    font-family: 'Georgia', serif;
    margin: 1vh 0;
  }

  h2 {
    font-size: 2vw;
    margin: 1vh 0;
  }

  .map-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: calc(100vh - 20vh); /* Increased height */
  }

  svg {
    width: 80%; /* Relative to the container size */
    height: auto; /* Maintain aspect ratio */
  }

  .container {
    position: relative;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }

  .dropdown {
    margin-top: 2vh;
    width: 100%;
    height: 100%;
    max-width: 300px;
  }

  label {
    display: block;
    margin-bottom: 1vh;
    font-size: 1.5vw;
    color: #000; /* Ensure label color is black */
  }

  select {
    width: 100%;
    padding: 1vh;
    font-size: 1.5vw;
    border-radius: 5px;
    border: 1px solid #ddd;
    background-color: #fff;
    color: #333;
  }

  @media (max-width: 768px) {
    h1 {
      font-size: 4vw;
    }

    h2 {
      font-size: 3vw;
    }

    .dropdown {
      width: 90%;
    }

    label {
      font-size: 2.5vw;
    }

    select {
      font-size: 2.5vw;
    }
  }

  @media (max-width: 480px) {
    h1 {
      font-size: 5vw;
    }

    h2 {
      font-size: 4vw;
    }

    .dropdown {
      width: 100%;
    }

    label {
      font-size: 3.5vw;
    }

    select {
      font-size: 3.5vw;
    }
  }
</style>