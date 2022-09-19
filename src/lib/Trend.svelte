<script>
    import { onMount, afterUpdate, createEventDispatcher } from "svelte";


    import { fade } from 'svelte/transition';
    import * as d3 from "d3";
    import "d3-attrs";
    import { rgb } from "d3";

    // @ts-ignore
    const dispatch = createEventDispatcher();
    export let nations, trends, rangeTrend, rankingBasis, mergedNations;
    //range reflects only the selected, trendsInd handles the nations as well
    const colors = [
        ["rgb(241,0,84)", "rgb(249,153,187)", "rgb(241,0,84)"],
        ["rgb(67,64,164)", "rgb(180,179,219)", "rgb(67,64,164)"],
        ["rgb(80,138,0)", "rgb(185,208,153)", "rgb(80,138,0)"],
    ];

    let svg,
        svgWidth,
        svgHeight,
        xScale,
        xAxis,
        xAxisG,
        yScale,
        yAxis,
        yAxisG,
        trendG;
    let buffer = 30;
    const rKeys = [
        "2005",
        "2006",
        "2007",
        "2008",
        "2009",
        "2010",
        "2011",
        "2012",
        "2013",
        "2014",
        "2015",
        "2016",
        "2017",
        "2018",
        "2019",
        "2020",
        "2021",
        "2022",
        "2023",
        "2024",
    ];

    const drawLine = function () {
        if (d3.select("g#trend")) d3.select("g#trend").remove();
        trendG = svg.append("g").attr("id", "trend");
        trends.forEach((trend) => {
        trendG
            .append("g")
            .attr("id", `trend${trend[0].iso}.${trend[0].ind}`)
            .selectAll("path")
            .data([trend])
            .join("path")
            .attr(
            "d",
            d3
                .line()
                .x((d) => xScale(d.year))
                .y((d) => yScale(d.val))
                .curve(d3.curveLinear)
            )
            .attr("class", "trendPathLine")
            .styles({
            fill: "none",
            "stroke-width": 5,
            stroke: (d) => {
                let second = d[0].ind=='nonres'?1:0;
                return colors[nations.indexOf(d[0].name)][
                second
                ];
            },
            "stroke-dasharray": d=>d[0].ind=='civil'?'5 4':0,
            filter:'drop-shadow(1px 8px 3px rgb(0 0 0 / 0.2))'
            });
        });
    };
    const drawMerged = function(){
            if (d3.select("g#trend")) d3.select("g#trend").remove();
            trendG = svg.append("g").attr("id", "trend");
            mergedNations.forEach((trend) => {
            trendG
                .append("g")
                .attr("id", `trend${trend[0].iso}`)
                .selectAll("path")
                .data([trend])
                .join("path")
                .attr(
                "d",
                d3
                    .line()
                    .x((d) => xScale(d.year))
                    .y((d) => yScale(d.val))
                    .curve(d3.curveLinear)
                )
                .attr("class", "trendPathLine")
                .styles({
                fill: "none",
                "stroke-width": 5,
                stroke: (d) => {
                    return colors[nations.indexOf(d[0].name)][0];
                },
                filter:'drop-shadow(1px 8px 3px rgb(0 0 0 / 0.2))'
                });
            });
            dispatch('newMerged')
    };
    const drawRect = function(){
        if (d3.select('#bgrect')) d3.select('#bgrect').remove() 
        svg
        .append("g")
        .attr("id", "bgrect")
        .selectAll("rect")
        .data([2021])
        .join("rect")
        .attrs({
            width: (d) => xScale(rankingBasis) - xScale(d),
            height: yScale(svgHeight),
            x: xScale(2021),
            y: 0,
            fill: rgb(195,197,196),
        });
    }

    onMount(() => {
        svg = d3.select("svg");
        svgWidth = svg.node().clientWidth - buffer;
        svgHeight = svg.node().clientHeight - buffer;

        xScale = d3
        .scaleLinear()
        .domain([trends[0][0].year - 1, +trends[0][trends[0].length - 1].year])
        .range([buffer, svgWidth])
        .nice();
        xAxis = d3.axisBottom(xScale).ticks(trends[0].length);
        xAxisG = svg.append("g").attrs({
        id: "xAxisG",
        transform: `translate(0,${svgHeight})`,
        });
        xAxis(xAxisG);
        //titles dont work
        yScale = d3
        .scaleLinear()
        .domain(rangeTrend)
        .range([buffer, svgHeight])
        .nice();
        yAxis = d3.axisLeft(yScale).ticks(10);
        yAxisG = svg.append("g").attrs({
        id: "yAxisG",
        transform: `translate(${buffer},0)`,
        });
        yAxis(yAxisG);
        console.log("und", yScale(86029));


        drawRect();
        drawLine();
    });
    afterUpdate(() => {
        drawRect();
        if(mergedNations.length) {
            console.log('trends inside 1', trends)
            drawMerged();
        } else {
            console.log('trends inside 2', trends)
            drawLine();
        }
    });
</script>

    <svg></svg>

<style>
    svg {
        background-color: white;
        width: 100%;
        height: 60rem;
    }
</style>
