<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    let data = [];
  
    onMount(() => {
      d3.csv('src/DSC106_NBA.csv').then((csvData) => {
        data = csvData;
        console.log('huh')
  
        // Call the function to render the bar chart
        renderBarChart();
      });
    });
  
    function renderBarChart() {
      const margin = { top: 20, right: 20, bottom: 30, left: 40 };
      const width = 1000 - margin.left - margin.right;
      const height = 600 - margin.top - margin.bottom;
  
      const svg = d3.select('#chart')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`);
  
      const x = d3.scaleBand()
        .range([0, width])
        .padding(0.1)
        .domain(data.map(d => d.Year));
  
      const y = d3.scaleLinear()
        .range([height, 0])
        .domain([0, d3.max(data, d => +d['Boston Celtics'])]);
  
      svg.append('g')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x));
  
      svg.append('g')
        .call(d3.axisLeft(y));
  
      svg.selectAll('rect')
        .data(data)
        .enter().append('rect')
        .attr('x', d => x(d.Year))
        .attr('y', d => y(+d['Boston Celtics']))
        .attr('width', x.bandwidth())
        .attr('height', d => height - y(+d['Boston Celtics']))
        .attr('fill', 'green');
    }
  </script>
  
  <style>
    /* Add your styles here */
  </style>
  
  <main>
    <div id="chart"></div>
  </main>
  