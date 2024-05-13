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
        
        const tooltip = d3.select('body').append('div')
            .attr('class', 'tooltip');

        // Drawing the states
        svg.selectAll('.state')
            .data(usMap.features)
            .enter().append('path')
            .attr('class', 'state')
            .attr('d', pathGenerator)
            .attr('fill', d => {
                const stateData = data.find(state => state.state === d.properties.NAME);
                return stateData ? colorScale(stateData.Gun_Law_Rating) : '#ddd';  // Fallback color
            }).on('mouseover', function(event, d) {
                d3.select(this).attr('stroke', 'red').attr('stroke-width', 2);
                const stateData = data.find(state => state.state === d.properties.NAME);
                tooltip
                    .style('opacity', 1)
                    .html(`<strong>${d.properties.NAME}</strong><br>Rating: ${stateData ? stateData.Gun_Law_Rating : 'N/A'}`)
                    .style('left', (event.pageX + 10) + 'px')
                    .style('top', (event.pageY - 28) + 'px');
            })
            .on('mouseout', function() {
            d3.select(this).attr('stroke', 'black').attr('stroke-width', 1);
            tooltip.style('opacity', 0);
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
    .STATE {
        stroke: #000; /* Default black border */
        stroke-linejoin: round;
        stroke-width: 1;
        fill-opacity: 1; /* Ensure full opacity */
        transition: stroke 0.2s; /* Smooth transition */
    }

    .STATE:hover {
        stroke: red; /* Color when hovered */
        stroke-width: 2; /* Thicker border on hover */
    }

    .tooltip {
        position: absolute;
        text-align: center;
        padding: 8px;
        background: white;
        border: 1px solid #ccc;
        border-radius: 4px;
        pointer-events: none; /* Ensures it doesn't capture mouse events */
        opacity: 0;
        transition: opacity 0.2s; /* Smooth transition */
    }
    .label {
      font-size: 10px;
      font-family: "Arial", sans-serif;
      fill: #a9c90c84;  
    }
  </style>
  