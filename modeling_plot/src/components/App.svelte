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
  let minVal = 70;
  let maxVal = 130;
  let svg;
  let x;
  let y;
  let bars;
  let height;

  let tooltip;

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

    const margin = { top: 20, right: 120, bottom: 100, left: 60 };
    const width = 1400 - margin.left - margin.right;
    height = 600 - margin.top - margin.bottom;

    svg = d3
      .select("#chart")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleBand().domain(keys).range([0, width]).padding(0.1);

    y = d3.scaleLinear().domain([minVal, maxVal]).range([height, 0]);

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
      .attr("fill", "steelblue")
    //   .on("mouseover", (event, d) => {
    //     tooltip = svg.append("g").attr("class", "tooltip");

    //     tooltip
    //       .append("rect")
    //       .attr("width", 130)
    //       .attr("height", 60)
    //       .attr("fill", "rgba(255, 255, 255, 0.8)")
    //       .attr("stroke", "black")
    //       .attr("stroke-width", 1);

    //     tooltip
    //       .append("text")
    //       .attr("x", 15)
    //       .attr("y", 20)
    //       .attr("dy", "0.35em")
    //       .attr("text-anchor", "start")
    //       .style("font-size", "12px")
    //       .text(`${d.team}`);

    //     tooltip
    //       .append("text")
    //       .attr("x", 15)
    //       .attr("y", 40)
    //       .attr("dy", "0.35em")
    //       .attr("text-anchor", "start")
    //       .style("font-size", "12px")
    //       .text(`PPG: ${d.score}`);

    //     updateTooltipPosition(event);
    //   })
    //   .on("mousemove", (event, d) => {
    //     updateTooltipPosition(event, d);
    //   })
    //   .on("mouseout", (event) => {
    //     svg.select(".tooltip").remove();
    //   });
    .on('mouseover', (event, d) => {
        const tooltipWidth = 120;
        const tooltipHeight = 60;

        const tooltip = svg.append('g')
          .attr('class', 'tooltip')
          .attr('transform', `translate(${x(d.team) + x.bandwidth()}, ${y(d.score)})`);

        tooltip.append('rect')
          .attr('width', tooltipWidth)
          .attr('height', tooltipHeight)
          .attr('fill', 'rgba(255, 255, 255, 0.8)')
          .attr('stroke', 'black')
          .attr('stroke-width', 1);

        tooltip.append('text')
          .attr('x', tooltipWidth / 2)
          .attr('y', tooltipHeight / 2 - 10)
          .attr('dy', '0.35em')
          .attr('text-anchor', 'middle')
          .style('font-size', '12px')
          .text(`${d.team}`);
        
          tooltip.append('text')
          .attr('x', tooltipWidth / 2)
          .attr('y', tooltipHeight / 2 + 10)
          .attr('dy', '0.35em')
          .attr('text-anchor', 'middle')
          .style('font-size', '12px')
          .text(`PPG: ${d.score}`);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on('mouseout', (event) => {
        svg.select('.tooltip').remove();
      });
  }

  function updateTooltipPosition(event, d) {
    tooltip = svg.select(".tooltip");
    const tooltipWidth = 100;
    const tooltipHeight = 40;
    const mouseX = event.pageX;
    const mouseY = event.pageY;

    // tooltip.attr(
    //   "transform",
    //   `translate(${mouseX - 280}, ${mouseY - tooltipHeight - 170})`,
    // );
    // tooltip.attr(
    //   "transform",
    //   `translate(${0.5 * mouseX}, ${0.5 * mouseY})`,
    // );

    // tooltip.attr('transform', `translate(${mouseX-40}, ${mouseY - tooltipHeight - 10})`);
    // tooltip.select('.text').text(`${event.srcElement.__data__.team}: ${event.srcElement.__data__.score} dsfjhsdk`);
    // console.log(tooltip.select('.text'))
    // console.log(event.srcElement.__data__);
    // console.log(tooltip.selectAll('text'));
    // const first = tooltip.select('text')
    // tooltip.select('text').text(`${d.team}: ${d.score}`);
    tooltip.select('text:nth-child(3)').text(`PPG: ${d.score}`);
    // tooltip.selectAll('text').text(`${d.team}: ${d.score}`);
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

    bars
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("width", x.bandwidth())
      .merge(bars)
      .transition()
      .duration(800)
      .attr("y", (d) => y(d.score))
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "steelblue")
      .on("mouseover", (event, d) => {
        tooltip = svg.append("g").attr("class", "tooltip");

        tooltip
          .append("rect")
          .attr("width", 130)
          .attr("height", 60)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        // tooltip
        //   .append("text")
        //   .attr("x", 15)
        //   .attr("y", 20)
        //   .attr("dy", "0.35em")
        //   .attr("text-anchor", "start")
        //   .style("font-size", "12px")
        //   .text(`${d.team}`);

        // tooltip
        //   .append("text")
        //   .attr("x", 15)
        //   .attr("y", 40)
        //   .attr("dy", "0.35em")
        //   .attr("text-anchor", "start")
        //   .style("font-size", "12px")
        //   .text(`PPG: ${d.score}`);
        tooltip.append('text')
          .attr('x', tooltipWidth / 2)
          .attr('y', tooltipHeight / 2 - 10)
          .attr('dy', '0.35em')
          .attr('text-anchor', 'middle')
          .style('font-size', '12px')
          .text(`${d.team}`);
        
          tooltip.append('text')
          .attr('x', tooltipWidth / 2)
          .attr('y', tooltipHeight / 2 + 10)
          .attr('dy', '0.35em')
          .attr('text-anchor', 'middle')
          .style('font-size', '12px')
          .text(`PPG: ${d.score}`);

        updateTooltipPosition(event, d);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg.select(".tooltip").remove();
      });
    

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
  <div id="chart">
    <h1>Is Defense Dying in the NBA?</h1>
    <h2 style="text-align: left;">NBA Average Points per Game in {year}</h2>
    <div id="overlay">
      <!-- svelte-ignore a11y-label-has-associated-control -->
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
  </div>
</main>

<style>
  #overlay {
    font-size: 0.9em;
    background-color: rgba(255, 255, 255, 0.4);
    position: absolute;
    min-width: 250px;
    width: 15%;
    top: 180px;
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
  #chart {
    margin: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }

  h2 {
    font-size: 24px;
    color: #FA8320;
    font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
  }

  h1 {
    font-size: 36px;
    font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
  }
</style>
