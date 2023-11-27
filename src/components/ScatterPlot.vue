<script setup>
import ChartItem from './ChartItem.vue'
</script>

<template>
  <ChartItem>
		<template #heading>'Last Changed Date' vs. 'Total Line of Codes'</template>
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
        `)
				.property("value", []);
      const g = svg.append("g");

			const parseTime = d3.utcParse("%d-%b-%Y");

      const randomHexColorCode = () => {
        let n = (Math.random() * 0xfffff * 1000000).toString(16);
        return '#' + n.slice(0, 6);
      };

      data.forEach(d => d.fillColor = randomHexColorCode());

			const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
			const parseDate = (dateString) => {
				let [d, m, y] = dateString.split("-");
				return `${d}-${months[m-1]}-${y}`;
			};

			const x = d3.scaleTime()
				.domain(
					d3.extent(data, d => {
						return parseTime(parseDate(d.lastedit));
					})
				).nice()
				.range([marginLeft, width - marginRight]);
			
			const xAxis = d3.axisBottom(x).tickSizeOuter(0);

			const y = d3.scaleLinear()
        .domain(
          d3.extent(data, d => d.lines.length)
        ).nice()
        .rangeRound([height - marginBottom, marginTop]);
      
      const yAxis = d3.axisLeft(y).tickSizeOuter(0);

			// Append the axes.
			g.append("g")
				.attr("transform", `translate(0,${height - marginBottom})`)
				.call(xAxis)
				.call(g => g.append("text")
					.attr("x", width - marginRight)
					.attr("y", -4)
					.attr("fill", "#000")
					.attr("font-weight", "bold")
					.attr("text-anchor", "end")
					.text("Last Changed"));

			g.append("g")
				.attr("transform", `translate(${marginLeft},0)`)
				.call(yAxis)
				.call(g => g.select(".tick:last-of-type text").clone()
					.attr("x", 10)
					.attr("y", 10)
					.attr("text-anchor", "end")
					.attr("font-weight", "bold")
					.text("Lines of Code")
					.attr("transform", "rotate(-90)")
					);

			g.append("g")
				.attr("fill", "none")
				.attr("stroke", "steelblue")
				.attr("stroke-width", 1.5)
				.selectAll("circle")
				.data(data)
				.join("circle")
					.attr("transform", d => `translate(${x(parseTime(parseDate(d.lastedit)))}, ${y(d.lines.length)})`)
					.attr("r", 3);
		}
	}
</script>

