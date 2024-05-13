<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';
  
    let data;
    let usMap;
  
    onMount(async () => {
      data = await d3.csv('/cleaned-gun-violence-data.csv');
      usMap = await d3.json('/gz_2010_us_040_00_500k.json'); // from https://eric.clst.org/tech/usgeojson/
      renderMap();
    });

    const colorScale = d3.scaleOrdinal()
    .domain(['F', 'D-', 'D', 'D+', 'C-', 'C', 'C+', 'B-', 'B', 'B+', 'A-', 'A'])
    .range([
        '#add8e6',  // Lightest blue for 'F'
        '#9ac7e6',
        '#87b6e6',
        '#74a5e6',
        '#6194e6',
        '#4e83e6',
        '#3b72e6',
        '#2861e6',
        '#1550e6',
        '#033fe6',
        '#002fde',
        '#001fce'   // Darkest blue for 'A'
    ]);
  
    function renderMap() {
        const projection = d3.geoAlbersUsa().scale(1300).translate([487.5, 305]);
        const pathGenerator = d3.geoPath().projection(projection);

        const svg = d3.select('#map').append('svg')
            .attr('width', 975)
            .attr('height', 610);

        // Drawing the states
        svg.selectAll('.state')
            .data(usMap.features)
            .enter().append('path')
            .attr('class', 'state')
            .attr('d', pathGenerator)
            .attr('fill', d => {
                const stateData = data.find(state => state.state === d.properties.NAME);
                return stateData ? colorScale(stateData.Gun_Law_Rating) : '#ddd';  // Fallback color
            });

        // Adding labels to each state
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
            return stateData ? stateData.Gun_Law_Rating : 'N/A';  // 'N/A' to identify unmatched entries
            })
            .attr('fill', 'black'); // Ensure labels are visible
    }
  </script>
  
  <div id="map"></div>
  
  <style>
    .state {
      stroke: #a9c90c84;
      stroke-linejoin: round;
    }
    .label {
      font-size: 10px;
      font-family: "Arial", sans-serif;
      fill: #a9c90c84;  /* White text may not be visible on lighter colors; adjust as needed */
    }
  </style>
  