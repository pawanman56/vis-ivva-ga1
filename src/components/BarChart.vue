<script setup>
import ChartItem from './ChartItem.vue'
</script>

<template>
  <ChartItem>
    <template #heading>'File Name' vs. 'Total Line of Codes'</template>
    <svg></svg>
  </ChartItem>
</template>

<script>
  import * as d3 from 'd3'
  import sourceCodeData from '../assets/source_code_data.json'

  export default {
    data() {
      return {};
    },

    mounted() {
      const width = 600;
      const height = 400;
      const marginTop = 20;
      const marginRight = 20;
      const marginBottom = 20;
      const marginLeft = 40;

      const data = sourceCodeData.data;

      const svg = d3.select("svg")
        .attr("viewBox", [0, 0, width, height])
        .attr("style", `
          max-width: ${width}px;
          height: auto;
          font: 10px sans-serif;
          overflow: visible;
        `);
      const g = svg.append("g");

      const randomHexColorCode = () => {
        let n = (Math.random() * 0xfffff * 1000000).toString(16);
        return '#' + n.slice(0, 6);
      };

      data.forEach(d => d.fillColor = randomHexColorCode());
      
      const x = d3.scaleBand()
        .domain(data.map(d => d.filename))
        .range([marginLeft, width - marginRight])
        .padding(0.2);

      const xAxis = d3.axisBottom(x).tickSizeOuter(0);

      const y = d3.scaleLinear()
        .domain(
          d3.extent(data, d => d.lines.length)
        ).nice()
        .rangeRound([height - marginBottom, marginTop]);
      
      const yAxis = d3.axisLeft(y).tickSizeOuter(0);
      
      g.append("g")
        .attr("transform", `translate(0, ${height - marginBottom})`)
        .call(xAxis);

      g.append("g")
        .attr("transform", `translate(${marginLeft}, 0)`)
        .call(yAxis);

      g.append("g")
        .attr("fill", "steelblue")
        .selectAll("rect")
        .data(data)
        .join("rect")
          .style("mix-blend-mode", "multiply")
          .attr("x", d => x(d.filename))
          .attr("y", d => y(d.lines.length))
          .attr("height", d => y(0) - y(d.lines.length))
          .attr("width", x.bandwidth())
          .attr("fill", d => d.fillColor);
    }
  }
</script>
