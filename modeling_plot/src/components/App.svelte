<script>
  import { onMount } from "svelte";
  import * as d3 from "d3";

  let data = [];
  let keys = [];
  let teams = [];
  let values = [];
  let year = 1950;
  let interval;
  let playing = false;
  let minVal = 75;
  let maxVal = 125;
  let svg;
  let x;
  let y;
  let bars;
  let height;

  onMount(() => {
    d3.csv("src/DSC106_NBA.csv").then((csvData) => {
      data = csvData;
      renderBarChart(year);
    });
  });

  function renderBarChart(year) {
    teams = data.filter((d) => d.Year == year);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        return { team: key, score: value };
      });

    const margin = { top: 20, right: 20, bottom: 100, left: 40 };
    const width = 1000 - margin.left - margin.right;
    height = 600 - margin.top - margin.bottom;

    svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleBand().domain(keys).range([0, width]).padding(0.1);

    y = d3
      .scaleLinear()
      .domain([
        minVal,
        maxVal,
      ])
      .range([height, 0]);

    svg
      .append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x))
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end");

    svg.append("g").call(d3.axisLeft(y));

    svg
      .selectAll("rect")
      .data(values)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("y", (d) => y(d.score))
      .attr("width", x.bandwidth())
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "steelblue");
  }

  function updateBars(newyr) {
    teams = data.filter((d) => d.Year == newyr);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        return { team: key, score: value };
      });

    bars = svg.selectAll("rect").data(values);

    bars.enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("width", x.bandwidth())
      .merge(bars)
      .transition()
      .duration(800)
      .attr("y", (d) => y(d.score))
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "steelblue");

    bars.exit().remove();
}


  function updateYear(newYear) {
    year = newYear;
    updateBars(year);
  }

  function play() {
    playing = true;
    interval = setInterval(() => {
      if (year < 2022) {
        year++;
        updateBars(year);
      } else {
        stop(); 
      }
    }, 1000);
  }

  function stop() {
    clearInterval(interval);
    playing = false;
  }
</script>

<main>
  <div id="chart"></div>
  <div id="overlay">
    <label>{year}</label>
    <input
      type="range"
      min="1950"
      max="2022"
      value={year}
      on:input={(e) => updateYear(+e.target.value)}
    />
    {#if playing}
      <button on:click={stop}>Stop</button>
    {:else}
      <button on:click={play}>Play</button>
    {/if}
  </div>
</main>

<style>
#overlay {
	font-size: 0.9em;
	background-color: rgba(255, 255, 255, 0.4);
	position: absolute;
	min-width:250px;
	width: 15%;
	top: 10px;
	right: 10px;
	padding: 10px;
	z-index: 3;
 }
 input {
	display: inline-block;
	width: 100%;
	position: relative;
	margin: 0;
	cursor: pointer;
 }
 label {
	font-size: 1.5em;
	font-family: sans-serif;
	font-weight: bold;
 }
</style>
