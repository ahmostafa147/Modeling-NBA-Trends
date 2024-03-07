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
  let svg1;
  let svg2;
  let svg3;
  let x;
  let y;
  let bars;
  let height;
  let logos;
  let mouseX;
  let mouseY;

  let tooltip;

  const teamLogos = {
    "Atlanta Hawks": "modeling_plot/src/components/logos/atlanta hawks.svg",
    "Boston Celtics":
      "https://cdn.nba.com/logos/nba/1610612738/primary/L/logo.svg",
    "Brooklyn Nets":
      "https://cdn.nba.com/logos/nba/1610612751/primary/L/logo.svg",
    "New York Knicks":
      "https://cdn.nba.com/logos/nba/1610612752/primary/L/logo.svg",
    "Philadelphia 76ers":
      "https://cdn.nba.com/logos/nba/1610612755/primary/L/logo.svg",
    "Toronto Raptors":
      "https://cdn.nba.com/logos/nba/1610612761/primary/L/logo.svg",
    "Chicago Bulls":
      "https://cdn.nba.com/logos/nba/1610612741/primary/L/logo.svg",
    "Cleveland Cavaliers":
      "https://cdn.nba.com/logos/nba/1610612739/primary/L/logo.svg",
    "Detroit Pistons":
      "https://cdn.nba.com/logos/nba/1610612765/primary/L/logo.svg",
    "Indiana Pacers":
      "https://cdn.nba.com/logos/nba/1610612754/primary/L/logo.svg",
    "Milwaukee Bucks":
      "https://cdn.nba.com/logos/nba/1610612749/primary/L/logo.svg",
    "Atlanta Hawks":
      "https://cdn.nba.com/logos/nba/1610612737/primary/L/logo.svg",
    "Charlotte Hornets":
      "https://cdn.nba.com/logos/nba/1610612766/primary/L/logo.svg",
    "Miami Heat": "https://cdn.nba.com/logos/nba/1610612748/primary/L/logo.svg",
    "Orlando Magic":
      "https://cdn.nba.com/logos/nba/1610612753/primary/L/logo.svg",
    "Washington Wizards":
      "https://cdn.nba.com/logos/nba/1610612764/primary/L/logo.svg",
    "Denver Nuggets":
      "https://cdn.nba.com/logos/nba/1610612743/primary/L/logo.svg",
    "Minnesota Timberwolves":
      "https://cdn.nba.com/logos/nba/1610612750/primary/L/logo.svg",
    "Oklahoma City Thunder":
      "https://cdn.nba.com/logos/nba/1610612760/primary/L/logo.svg",
    "Portland Trail Blazers":
      "https://cdn.nba.com/logos/nba/1610612757/primary/L/logo.svg",
    "Utah Jazz": "https://cdn.nba.com/logos/nba/1610612762/primary/L/logo.svg",
    "Golden State Warriors":
      "https://cdn.nba.com/logos/nba/1610612744/primary/L/logo.svg",
    "Los Angeles Clippers":
      "https://cdn.nba.com/logos/nba/1610612746/primary/L/logo.svg",
    "Los Angeles Lakers":
      "https://cdn.nba.com/logos/nba/1610612747/primary/L/logo.svg",
    "Phoenix Suns":
      "https://cdn.nba.com/logos/nba/1610612756/primary/L/logo.svg",
    "Sacramento Kings":
      "https://cdn.nba.com/logos/nba/1610612758/primary/L/logo.svg",
    "Dallas Mavericks":
      "https://cdn.nba.com/logos/nba/1610612742/primary/L/logo.svg",
    "Houston Rockets":
      "https://cdn.nba.com/logos/nba/1610612745/primary/L/logo.svg",
    "Memphis Grizzlies":
      "https://cdn.nba.com/logos/nba/1610612763/primary/L/logo.svg",
    "New Orleans Pelicans":
      "https://cdn.nba.com/logos/nba/1610612740/primary/L/logo.svg",
    "San Antonio Spurs":
      "https://cdn.nba.com/logos/nba/1610612759/primary/L/logo.svg",
  };

  onMount(() => {
    d3.csv("src/DSC106_NBA.csv").then((csvData) => {
      data = csvData;
      renderBarChart(year);
      renderBarChart2(1953);
      renderBarChart3(1990);
      renderBarChart4(2022)
    });
  });

  function renderBarChart(year) {
    teams = data.filter((d) => d.Year == year);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        const logo = teamLogos[key];
        return {
          team: key,
          score: value,
          logo:
            logo ||
            "https://cdn.freebiesupply.com/images/large/2x/nba-logo-transparent.png",
        }; // Provide a default logo URL if not found
      });

    const margin = { top: 40, right: 120, bottom: 150, left: 60 };
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
      .style("text-anchor", "end")
      .style("font-size", "12px");

    svg.append("g").call(d3.axisLeft(y));

    svg
      .selectAll(".logo")
      .data(values)
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .attr("y", (d) => y(d.score) - 40) // Adjust positioning as needed
      .attr("width", 30)
      .attr("height", 30)
      .attr("class", "logo");

    svg
      .selectAll("rect")
      .data(values)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("y", (d) => y(d.score))
      .attr("width", x.bandwidth())
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2)
      .on("mouseover", (event, d) => {
        const tooltipWidth = 120;
        const tooltipHeight = 60;

        const tooltip = svg
          .append("g")
          .attr("class", "tooltip")
          .attr(
            "transform",
            `translate(${x(d.team) + x.bandwidth()}, ${y(d.score)})`,
          );

        tooltip
          .append("rect")
          .attr("width", tooltipWidth)
          .attr("height", tooltipHeight)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`PPG: ${d.score}`);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg.select(".tooltip").remove();
      });
    svg
      .append("text")
      .attr("transform", `translate(${width / 2}, ${height + 120})`) // Adjust the position as needed
      .style("text-anchor", "middle")
      .text("Teams")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");

    svg
      .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left) // Adjust the position as needed
      .attr("x", 0 - height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Average Points Per Game Difference")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");
  }

  function updateTooltipPosition(event, d) {
    tooltip = svg.select(".tooltip");
    const tooltipWidth = 100;
    const tooltipHeight = 40;
    // const mouseX = event.pageX;
    // const mouseY = event.pageY;

    // tooltip.attr(
    //   "transform",
    //   `translate(${mouseX - 280}, ${mouseY - tooltipHeight - 170})`,
    // );
    // tooltip.attr(
    //   "transform",
    //   `translate(${0.5 * mouseX}, ${0.5 * mouseY})`,
    // );
    // console.log(event.toElement);
    let [mouseX, mouseY] = d3.pointer(event);
    console.log(mouseX, mouseY);
    tooltip.attr('transform', `translate(${mouseX + 10}, ${mouseY - tooltipHeight + 5})`);
    // tooltip.select('.text').text(`${event.srcElement.__data__.team}: ${event.srcElement.__data__.score} dsfjhsdk`);
    // console.log(tooltip.select('.text'))
    // console.log(event.srcElement.__data__);
    // console.log(tooltip.selectAll('text'));
    // const first = tooltip.select('text')
    // tooltip.select('text').text(`${d.team}: ${d.score}`);
    tooltip.select("text:nth-child(3)").text(`PPG: ${d.score}`);
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
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2);

    logos = svg.selectAll(".logo").data(values);
    logos
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .merge(logos)
      .transition()
      .duration(800)
      .attr("y", (d) => y(d.score) - 40)
      .attr("height", (d) => y(d.score) - 800)
      .attr("class", "logo")
      .on("mouseover", (event, d) => {
        tooltip = svg.append("g").attr("class", "tooltip");

        tooltip
          .append("rect")
          .attr("width", 130)
          .attr("height", 60)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
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














  function renderBarChart2(year) {
    teams = data.filter((d) => d.Year == year);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        const logo = teamLogos[key];
        return {
          team: key,
          score: value,
          logo:
            logo ||
            "https://cdn.freebiesupply.com/images/large/2x/nba-logo-transparent.png",
        }; // Provide a default logo URL if not found
      });

    const margin = { top: 40, right: 120, bottom: 150, left: 60 };
    const width = 1400 - margin.left - margin.right;
    height = 600 - margin.top - margin.bottom;

    svg1 = d3
      .select("#chart2")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleBand().domain(keys).range([0, width]).padding(0.1);

    y = d3.scaleLinear().domain([minVal, maxVal]).range([height, 0]);

    svg1
      .append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x))
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end")
      .style("font-size", "12px");

    svg1.append("g").call(d3.axisLeft(y));

    svg1
      .selectAll(".logo")
      .data(values)
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .attr("y", (d) => y(d.score) - 40) // Adjust positioning as needed
      .attr("width", 30)
      .attr("height", 30)
      .attr("class", "logo");

    svg1
      .selectAll("rect")
      .data(values)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("y", (d) => y(d.score))
      .attr("width", x.bandwidth())
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2)
      .on("mouseover", (event, d) => {
        const tooltipWidth = 120;
        const tooltipHeight = 60;

        const tooltip = svg1
          .append("g")
          .attr("class", "tooltip")
          .attr(
            "transform",
            `translate(${x(d.team) + x.bandwidth()}, ${y(d.score)})`,
          );

        tooltip
          .append("rect")
          .attr("width", tooltipWidth)
          .attr("height", tooltipHeight)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`PPG: ${d.score}`);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg1.select(".tooltip").remove();
      });
    svg1
      .append("text")
      .attr("transform", `translate(${width / 2}, ${height + 120})`) // Adjust the position as needed
      .style("text-anchor", "middle")
      .text("Teams")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");

    svg1
      .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left) // Adjust the position as needed
      .attr("x", 0 - height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Average Points Per Game Difference")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");
  }




  function renderBarChart3(year) {
    teams = data.filter((d) => d.Year == year);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        const logo = teamLogos[key];
        return {
          team: key,
          score: value,
          logo:
            logo ||
            "https://cdn.freebiesupply.com/images/large/2x/nba-logo-transparent.png",
        }; // Provide a default logo URL if not found
      });

    const margin = { top: 40, right: 120, bottom: 150, left: 60 };
    const width = 1400 - margin.left - margin.right;
    height = 600 - margin.top - margin.bottom;

    svg2 = d3
      .select("#chart3")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleBand().domain(keys).range([0, width]).padding(0.1);

    y = d3.scaleLinear().domain([minVal, maxVal]).range([height, 0]);

    svg2
      .append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x))
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end")
      .style("font-size", "12px");

    svg2.append("g").call(d3.axisLeft(y));

    svg2
      .selectAll(".logo")
      .data(values)
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .attr("y", (d) => y(d.score) - 40) // Adjust positioning as needed
      .attr("width", 30)
      .attr("height", 30)
      .attr("class", "logo");

    svg2
      .selectAll("rect")
      .data(values)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("y", (d) => y(d.score))
      .attr("width", x.bandwidth())
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2)
      .on("mouseover", (event, d) => {
        const tooltipWidth = 120;
        const tooltipHeight = 60;

        const tooltip = svg2
          .append("g")
          .attr("class", "tooltip")
          .attr(
            "transform",
            `translate(${x(d.team) + x.bandwidth()}, ${y(d.score)})`,
          );

        tooltip
          .append("rect")
          .attr("width", tooltipWidth)
          .attr("height", tooltipHeight)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`PPG: ${d.score}`);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg2.select(".tooltip").remove();
      });
    svg2
      .append("text")
      .attr("transform", `translate(${width / 2}, ${height + 120})`) // Adjust the position as needed
      .style("text-anchor", "middle")
      .text("Teams")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");

    svg2
      .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left) // Adjust the position as needed
      .attr("x", 0 - height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Average Points Per Game Difference")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");
  }

  function renderBarChart4(year) {
    teams = data.filter((d) => d.Year == year);
    keys = Object.keys(teams[0]).filter((key) => key !== "Year");
    values = Object.entries(teams[0])
      .filter((entry) => entry[0] !== "Year")
      .map(([key, value]) => {
        const logo = teamLogos[key];
        return {
          team: key,
          score: value,
          logo:
            logo ||
            "https://cdn.freebiesupply.com/images/large/2x/nba-logo-transparent.png",
        }; // Provide a default logo URL if not found
      });

    const margin = { top: 40, right: 120, bottom: 150, left: 60 };
    const width = 1400 - margin.left - margin.right;
    height = 600 - margin.top - margin.bottom;

    svg3 = d3
      .select("#chart4")
      .append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    x = d3.scaleBand().domain(keys).range([0, width]).padding(0.1);

    y = d3.scaleLinear().domain([minVal, maxVal]).range([height, 0]);

    svg3
      .append("g")
      .attr("transform", `translate(0,${height})`)
      .call(d3.axisBottom(x))
      .selectAll("text")
      .attr("transform", "rotate(-45)")
      .style("text-anchor", "end")
      .style("font-size", "12px");

    svg3.append("g").call(d3.axisLeft(y));

    svg3
      .selectAll(".logo")
      .data(values)
      .enter()
      .append("svg:image")
      .attr("xlink:href", (d) => d.logo)
      .attr("x", (d) => x(d.team) + x.bandwidth() / 2 - 15) // Adjust positioning as needed
      .attr("y", (d) => y(d.score) - 40) // Adjust positioning as needed
      .attr("width", 30)
      .attr("height", 30)
      .attr("class", "logo");

    svg3
      .selectAll("rect")
      .data(values)
      .enter()
      .append("rect")
      .attr("x", (d) => x(d.team))
      .attr("y", (d) => y(d.score))
      .attr("width", x.bandwidth())
      .attr("height", (d) => height - y(d.score))
      .attr("fill", "#FA8320")
      .attr("stroke", "black")
      .attr("stroke-width", 2)
      .on("mouseover", (event, d) => {
        const tooltipWidth = 120;
        const tooltipHeight = 60;

        const tooltip = svg3
          .append("g")
          .attr("class", "tooltip")
          .attr(
            "transform",
            `translate(${x(d.team) + x.bandwidth()}, ${y(d.score)})`,
          );

        tooltip
          .append("rect")
          .attr("width", tooltipWidth)
          .attr("height", tooltipHeight)
          .attr("fill", "rgba(255, 255, 255, 0.8)")
          .attr("stroke", "black")
          .attr("stroke-width", 1);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 - 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`${d.team}`);

        tooltip
          .append("text")
          .attr("x", tooltipWidth / 2)
          .attr("y", tooltipHeight / 2 + 10)
          .attr("dy", "0.35em")
          .attr("text-anchor", "middle")
          .style("font-size", "12px")
          .text(`PPG: ${d.score}`);
      })
      .on("mousemove", (event, d) => {
        updateTooltipPosition(event, d);
      })
      .on("mouseout", (event) => {
        svg3.select(".tooltip").remove();
      });
    svg3
      .append("text")
      .attr("transform", `translate(${width / 2}, ${height + 120})`) // Adjust the position as needed
      .style("text-anchor", "middle")
      .text("Teams")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");

    svg3
      .append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left) // Adjust the position as needed
      .attr("x", 0 - height / 2)
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Average Points Per Game Difference")
      .style(
        "font-family",
        "Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif",
      )
      .style("font-size", "20px");
  }


</script>

<main>
  <div id="chart" class="chart_class">
    <h1>
      Is Defense Dying in the NBA?<img
        src="https://images.ctfassets.net/h8q6lxmb5akt/5qXnOINbPrHKXWa42m6NOa/421ab176b501f5bdae71290a8002545c/nba-logo_2x.png"
        ,
        alt="NBA"
      />
    </h1>
    <h2 style="text-align: left;">NBA Teams Difference in Average Points per Game in {year} From All Time Lowest Average</h2>
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

  <div id = "chart2" class="chart_class">
    <h2 style="text-align: left;">NBA Teams Difference in Average Points per Game in 1953 From All Time Lowest Average</h2>
  </div>
  <div id = "chart3" class="chart_class">
    <h2 style="text-align: left;">NBA Teams Difference in Average Points per Game in 1990 From All Time Lowest Average</h2>
  </div>
  <div id = "chart4" class="chart_class">
    <h2 style="text-align: left;">NBA Teams Difference in Average Points per Game in 2022 From All Time Lowest Average</h2>
  </div>
  
  <div id="text">
    <h3 style="text-align: left;">Design Process and Decisions</h3>
    <p style="font-size: 24;">A common narrative and topic of debate amongst NBA (National Basketball 
      Association) fans and media is whether teams are scoring too many 
      points and defense is becoming a smaller part of the game. We decided 
      to explore this narrative by plotting the average points scored per 
      game per team over the history of the league, starting from 1950 when 
      the league only comprised of 8 teams.</p>
  <p>For our data, we decided to get it from Basketball Reference, a widely 
      trusted, although not official, source for basketball data amongst NBA 
      fans and media. We chose to scrape data from Basketball Reference as 
      opposed to NBA.com, the league’s official source for data, because 
      Basketball Reference was much easier to get data and the data was 
      cleaner. Cleaning the data involved filling in zeros for the years 
      when certain teams did not exist yet. Otherwise, the Basketball 
      Reference data was clean and easy to work with.</p>
  <p>Regarding design decisions, we knew we wanted to do a slider to show 
      data for different years and be able to play it to show the progression 
      of scoring over the years. As a result, we chose to use a bar chart as 
      it was an efficient way to display data for all 30 teams (which are 
      essentially 30 categories) for a single year. On top of each individual 
      bar, we decided to add the team’s logo. This is a quick and easy way 
      to identify a team’s bar, if one knows the team logos, without having 
      to look all the way down to the x-axis on top of helping differentiate 
      bars when many can have similar values. For each bar, we have a tooltip 
      that shows the exact points per game (PPG) values for the team being 
      hovered over. This is helpful as in a bar chart it can be hard to get 
      an exact value, especially in the wider chart that we have. The tooltip 
      updates to whatever team it hovers over and the value updates with the 
      incrementing year as long as one’s mouse is moving. For the font of the 
      graph and axes titles, we decided to use the impact font as it matches 
      the font commonly used in NBA graphics on social media and titles of 
      NBA articles. The color of the bars matches the color of a basketball 
      to give a bit of additional semantic encoding.</p>
  <p>Playing the slider and observing the changes over the years, we can see 
    that teams were scoring very few points in the early years of the league,
     likely as basketball was still being fleshed out as a serious sport. 
     Over the next three decades until around the 1990s, teams would score a 
     respectable amount of points, with many teams hovering around 110 points 
     per game. Starting in the 90s, teams started scoring significantly less 
     points and averages reached lows not seen since the inception of the 
     league in the 2000s, when some teams were averaging less than 90 points 
     per game. This trend of low scoring games stopped around the mid 2010s, 
     with team scoring averages increasing at an unprecedented rate in the 
     last few years. Our conclusion is that while there is reason for concern 
     with the quick uptick in scoring in not even the last decade. However, 
     this level of scoring has been seen before in the 70s and 80s. Therefore, 
     this fear is most likely exaggerated by older NBA fans who grew up 
     watching the lower scoring style of basketball of the 1990s and 2000s. </p>
  <p>To begin our development process, we had to decide on a dataset. We all 
    agreed that the datasets we had used for Project 2 were limiting, so we 
    decided to find some new, bigger data. Nathan and Walter were confident 
    in their domain knowledge of the NBA and basketball. Additionally, they 
    knew that there is an abundance of data available from sources such as 
    Basketball Reference and NBA.com. </p>
  <p>Approaching the project from the perspective of making an interactive 
    visualization, we decided from the start to do something that changes 
    with time, incorporating a slider like we did in Lab 6 and one of the 
    D3 examples in lecture. In using a slider, we decided to not do a line 
    chart as it would not make much sense, because line charts typically 
    show progression and trends over a period of time. Having a line for 
    each NBA team would have been too messy. With the bar chart, we decided 
    to limit the y-axis to a range from 0 to 60. The value represented by the 
    bar charts represents the difference in points per game for that team 
    that year from the all time lowest average (70.0 by the Atlanta Hawks 
    in 1953). This allows us to see how much teams have improved at scoring 
    since the worst team near the beginning of the league. This 
    transformation of the data is key as it makes small differences more 
    evident. A seemingly small difference of 5 PPG is extremely significant 
    in the context of basketball. </p>
  <p>With the slider, Ahmed made it so that it can be played automatically 
    and that it would increment at a rate of about a year per second. This 
    rate allows for the transitions to fully show and therefore the bar will 
    show the corect value before changing to the next year. Regarding the 
    tooltip, we decided to add it as we believed just having the slider has 
    not enough interactivity. The tooltip pops up when you hover over the 
    bar for a team and shows the team name being hovered over as wel as the 
    exact PPG value, as this can be hard to get on a bar chart. During the 
    development process, Walter initially had the tooltip follow the position 
    of the mouse. However, he later decided to attach the tooltip to the 
    associated bar, as there was a problem with the position of the tooltip 
    on different computers. Depending on one’s browser and the size of one’s 
    screen, the tooltip would sometimes not be close to the mouse, instead 
    being way off while still moving with the mouse in some manner. Nathan 
    decided that adding logos would make the bars easier to follow and add 
    helpful redundant encodings. As mentioned in the design decisions above, 
    this allows for easier differentiation and identification of the bars 
    without having to trace the bar all the way down to the x-axis. Nathan 
    also decided to use the links of the team logos from the official NBA 
    website so that when a team’s logo gets updated, this change would be 
    reflected in our visualization as well. </p>
  <p>We were initially planning on making each bar match the colors of the 
    associated team as well, however Nathan and Ahmed decided that this would 
    be too confusing and not as color blind friendly. As a result, Walter 
    decided on the orange color to match that of a basketball and the black 
    stroke to create contrast between the bars themselves as well as the 
    background and axes. Walter also decided on the title font based on what 
    he had seen before in NBA graphics and articles. As a finishing touch, 
    he also added the NBA logo in the top-left corner of the page to quickly 
    let readers know that they were dealing with a basketball visualization 
    if they had some prior knowledge about the professional league.</p>
  <p>Roles were more or less mentioned above in the development process. 
    Ahmed was responsible for setting up the Svelte framework and made the 
    slider as well the function to update the bar chart to match the year. 
    Nathan was responsible for creating the logos and having them match the 
    height of the bars to create a helpful visual encoding. Walter was 
    responsible for creating the tooltip and the remaining auxiliary design 
    decisions. Ahmed was also responsible for setting up the website. In 
    general, we worked together for the majority of the project and constantly 
    bounced ideas and feedback off of each other.</p>
  </div>
</main>

<style>
  #overlay {
    font-size: 0.9em;
    /* background-color: rgba(157, 152, 152, 0.2); */
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

  .chart_class {
    background-color: antiquewhite;
    margin: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }

  h2 {
    font-size: 24px;
    /* color: #FA8320; */
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
  }

  h1 {
    font-size: 36px;
    font-family: Impact, Haettenschweiler, "Arial Narrow Bold", sans-serif;
    /* color: #FA8320; */
  }
  img {
    position: absolute;
    top: 0;
    left: 0;
    height: 20%;
  }
  #text {
      font-size: 18px;
      margin-left: 40px;
      margin-right: 40px;
      text-align: justify;
      font-family:Arial, Helvetica, sans-serif
    }
</style>
