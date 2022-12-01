<template>
    <div id="parallel"></div>
</template>

<script>
import * as d3 from "d3";
import rawdata from "../../assets/data/filtered.json"
export default {
    name: 'ParallelChart',
    data() {
        return {
            process_data: undefined,
            keys: ["price", "metacritic_score", "positive", "achievements", "average_playtime_forever"],
        }
    },
    props: {
        myParallelChartData: Array,
    },
    mounted() {
        this.processData(rawdata)
        this.drawParallelChart(this.process_data, '#parallel')
    },
    methods: {
        processData(data) {
            console.log("parallel chart process data")
            let count = 0
            //console.log(data)
            const preData = []
            for (const prop in data) {
                //console.log(data[prop])
                count  = count + 1
                preData.push(data[prop])
            }
            console.log("total games: ", count)
            console.log(preData)
            this.process_data = preData
        },
        drawParallelChart(data, id) {
            const margin = { top: 30, right: 10, bottom: 30, left: 10 };
            const height = this.keys.length * 120;
            const width = 700;
            const brushHeight = 50;

            const svg = d3.select(id).append("svg")
                            .attr("width", width + margin.left + margin.right)
                            .attr("height", height + margin.top + margin.bottom)
                            .append("g")
                            .attr("transform", `translate(${margin.left}, ${margin.top})`);

            const selections = new Map();
            
            function brushed({selection}, key) {
                if (selection === null) selections.delete(key);
                else selections.set(key, selection.map(x.get(key).invert));
                const selected = [];
                path.each(function(d) {
                  const active = Array.from(selections).every(([key, [min, max]]) => d[key] >= min && d[key] <= max);
                  d3.select(this).style("stroke", active ? "#69b3a2" : "#ddd");
                  if (active) {
                    d3.select(this).raise();
                    selected.push(d);
                  }
                });
                console.log(selected)
                svg.property("value", selected).dispatch("input");
            }

            const brush = d3.brushX()
                            .extent([
                            [margin.left, -(brushHeight / 2)],
                            [width - margin.right, brushHeight / 2]
                            ])
                            .on("start brush end", brushed);
            
            const x = new Map(Array.from(this.keys, key => [key, d3.scaleLinear(d3.extent(data, d => d[key]), [margin.left, width - margin.right])]))
            const y = d3.scalePoint(this.keys, [margin.top, height - margin.bottom])

            // axis
            svg.append("g")
                .selectAll("g")
                .data(this.keys)
                .join("g")
                .attr("transform", d => `translate(0, ${y(d)})`)
                .each(function(d) { d3.select(this).call(d3.axisBottom(x.get(d))); })
                .call(g => g.append("text")
                    .attr("x", margin.left)
                    .attr("y", -6)
                    .attr("text-anchor", "start")
                    .attr("fill", "currentColor")
                    .text(d => d))
                .call(g => g.append("text")
                    .clone(true).lower()
                    .attr("fill", "none")
                    .attr("stroke-width", 5)
                    .attr("stroke-linejoin", "round")
                    .attr("stroke", "white"))
                .call(brush)
            
            const line = d3.line().defined(([, value]) => value != null)
                            .x(([key, value]) => x.get(key)(value))
                            .y(([key]) => y(key))

            // line
            const path = svg.append("g")
                .attr("fill", "none")
                .attr("stroke-width", 1.5)
                .attr("stroke-opacity", 0.4)
                .selectAll("path")
                .data(data)
                .join("path")
                .attr("stroke", "#69b3a2")
                .attr("d", d => line(d3.cross(this.keys, [d], (key, d) => [key, d[key]])))

            
        }
    }
}
</script>