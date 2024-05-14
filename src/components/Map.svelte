<script>
  import * as d3 from 'd3';
  import { onMount } from 'svelte';

  let data;
  let usMap;
  let deathData;
  export let selectedGunType;

  onMount(async () => {
    try {
      data = await d3.csv('/cleaned-gun-violence-data.csv');
      deathData = await d3.csv('/deaths_state_and_type.csv');
      usMap = await d3.json('/gz_2010_us_040_00_500k.json');

      deathData.forEach(d => {
        // Convert string to numbers
        d.Handgun = +d.Handgun;
        d.Shotgun = +d.Shotgun;
        d.Rifle = +d.Rifle;
        d["Automatic Rifle"] = +d["Automatic Rifle"];
        d["Semiautomatic Rifle"] = +d["Semiautomatic Rifle"];
        d.Unknown = +d.Unknown;

        // Calculate the total deaths in each state
        d.Total = d.Handgun + d.Shotgun + d.Rifle + d["Automatic Rifle"] + d["Semiautomatic Rifle"] + d.Unknown;
        d['All Percentage'] = d.Handgun + d.Shotgun + d.Rifle + d["Automatic Rifle"] + d["Semiautomatic Rifle"];

        // Calculate the percentage of each type relative to the total
        d['Handgun Percentage'] = (d.Handgun / d.Total * 100).toFixed(2);
        d['Shotgun Percentage'] = (d.Shotgun / d.Total * 100).toFixed(2);
        d['Rifle Percentage'] = (d.Rifle / d.Total * 100).toFixed(2);
        d['Automatic Rifle Percentage'] = (d["Automatic Rifle"] / d.Total * 100).toFixed(2);
        d['Semiautomatic Rifle Percentage'] = (d["Semiautomatic Rifle"] / d.Total * 100).toFixed(2);
        d['Unknown Percentage'] = (d.Unknown / d.All * 100).toFixed(2);
      });

      renderMap();
      updateMap(selectedGunType);
    } catch (error) {
      console.error("Failed to load data:", error);
    }
  });

  const colorScale = d3.scaleOrdinal()
    .domain(['F', 'D-', 'D', 'D+', 'C-', 'C', 'C+', 'B-', 'B', 'B+', 'A-', 'A'])
    .range([
      '#f2e6f4', // Very light purple
      '#e6cced',
      '#dbb2e6',
      '#cf99df',
      '#c480d8',
      '#b866d1',
      '#ac4dca',
      '#a033c3',
      '#9420bc',
      '#8818b5',
      '#7c10ae',
      '#7010a1'  // Deeper purple, still light enough for black text
    ]);

  function renderMap() {
    const projection = d3.geoAlbersUsa().scale(1400).translate([500, 300]);
    const pathGenerator = d3.geoPath().projection(projection);

    const svg = d3.select('#map').append('svg')
      .attr('width', '100%')
      .attr('height', '100%')
      .attr('viewBox', '0 0 975 610')
      .attr('preserveAspectRatio', 'xMidYMid meet');

    const tooltip = d3.select('#tooltip');

    svg.selectAll('.state')
      .data(usMap.features)
      .enter().append('path')
      .attr('class', 'state')
      .attr('d', pathGenerator)
      .attr('fill', d => {
        const stateData = data.find(state => state.state === d.properties.NAME);
        return stateData ? colorScale(stateData.Gun_Law_Rating) : '#ddd';
      })
      .attr('stroke', 'black')
      .attr('stroke-width', 1)
      .on('mouseover', function (event, d) {
        d3.select(this).attr('stroke', 'red').attr('stroke-width', 2);
        const stateData = data.find(state => state.state === d.properties.NAME);
        const deadData = deathData.find(state => state.state === d.properties.NAME);

        // Create a tooltip content string with additional data
        let tooltipContent = `<strong>${d.properties.NAME}</strong><br>`;
        tooltipContent += `Rating: ${stateData ? stateData.Gun_Law_Rating : 'N/A'}<br>`;

        // Add other data points if available
        if (deadData) {
          tooltipContent += `Handgun Deaths: ${deadData.Handgun || 0}<br>`;
          tooltipContent += `Rifle Deaths: ${deadData.Rifle || 0}<br>`;
          tooltipContent += `Automatic Rifle Deaths: ${deadData["Automatic Rifle"] || 0}<br>`;
          tooltipContent += `Semiautomatic Rifle Deaths: ${deadData["Semiautomatic Rifle"] || 0}<br>`;
          tooltipContent += `Unknown Gun Deaths: ${deadData.Unknown || 0}<br>`;
          tooltipContent += `Total Deaths: ${deadData.Total || 0}`;
        }

        // Display the tooltip
        tooltip
          .style('opacity', 1)
          .html(tooltipContent);
      })
      .on('mouseout', function () {
        d3.select(this).attr('stroke', 'black').attr('stroke-width', 1);
        tooltip.style('opacity', 0);
      });

    svg.selectAll('.label')
      .data(usMap.features)
      .enter().append('text')
      .attr('class', 'label')
      .attr('transform', d => {
        const centroid = pathGenerator.centroid(d);
        return `translate(${centroid})`;
      })
      .attr('dy', '0.35em')
      .attr('text-anchor', 'middle')
      .text(d => {
        const stateData = data.find(state => state.state === d.properties.NAME);
        return stateData ? stateData.Gun_Law_Rating : 'N/A';
      })
      .attr('fill', 'black');
  }

  function updateMap(gunType) {
    console.log(`Selected Gun Type: ${gunType}`);
    const normalizedGunType = gunType + ' Percentage'; // Adjust to use the percentage field

    const stateCounts = deathData.reduce((acc, d) => {
      const percentage = parseFloat(d[normalizedGunType]) || 0; // Use parseFloat to ensure numeric value
      acc[d.state] = (acc[d.state] || 0) + percentage;
      return acc;
    }, {});

    const topStates = Object.keys(stateCounts)
      .sort((a, b) => stateCounts[b] - stateCounts[a])
      .slice(0, 5);

    console.log('Top States by Percentage:', topStates);
    highlightStates(topStates);
  }

  function highlightStates(states) {
    const svg = d3.select('#map svg');
    svg.selectAll('.state')
      .style('fill', d => {
        const stateData = data.find(state => state.state === d.properties.NAME);
        return stateData ? colorScale(stateData.Gun_Law_Rating) : '#ddd';
      });

    svg.selectAll('.state')
      .filter(d => states.includes(d.properties.NAME))
      .style('fill', 'red');
  }

  $: if (deathData && usMap && data) {
    console.log(`Gun type changed to: ${selectedGunType}`);
    updateMap(selectedGunType);
  }
</script>

<div id="map" class="map-container"></div>
<div id="tooltip" class="tooltip"></div>

<style>
  .map-container {
    clip-path: polygon(20% 0, 100% 0, 100% 100%, 0 100%);
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%; 
  }

  svg {
    width: 100%; 
    height: 100%;
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
  .tooltip {
    position: absolute;
    top: 20px; 
    right: 20px;
    background-color: white;
    padding: 10px;
    border: 1px solid #ccc;
    box-shadow: 2px 2px 5px rgba(0,0,0,0.2);
    z-index: 1000;
    pointer-events: none;
    color: #333;
    font-size: 1rem;
  }
</style>
