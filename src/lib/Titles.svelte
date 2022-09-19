<script>
    // @ts-nocheck
    import { countries, useData, countriesSumDiff } from "../assets/data";
    import { onMount, afterUpdate, createEventDispatcher } from "svelte";
    import * as d3 from "d3";
    import { rgb } from "d3";
    import "d3-attrs";

    const dispatch = createEventDispatcher();
    export let rankingBasis, dataLength;
    let svgtext;
    let yScale = null;
    let svgWidth = 0;
    let svgHeight = 0;
    export let nations;
    let colors = [rgb(252, 204, 221), rgb(173, 196, 247), rgb(214, 239, 179)];

    const selectCountries = function (e, d) {
        let countryIndex = $countriesSumDiff.findIndex((c) => c.name === d.name);
        if (!nations.includes(d.name) && nations.length < 3) {
        nations = [...nations, d.name];
        d3.select(this).style("fill", "white");
        d3.select(`#bgT rect:nth-child(${countryIndex + 1})`).style(
            "fill",
            "black"
        );
        } else if (nations.length > 1 && nations.includes(d.name)) {
        nations = nations.filter((c) => c !== d.name);
        d3.select(this).style("fill", "black");
        d3.select(`#bgT rect:nth-child(${countryIndex + 1})`).style(
            "fill",
            "white"
        );
        } else {
        return;
        }
        dispatch("newNations", nations);
    };

    const setSvgTextScale = function () {
        yScale = d3
        .scaleLinear()
        .domain([0, dataLength - 1])
        .range([0, svgHeight - 35]);
        $countriesSumDiff.sort(function (a, b) {
        return b[`${rankingBasis}a`] - a[`${rankingBasis}a`];
        });
    };
    const setSvgTextBg = function (el) {
        if (d3.select("g#bgT")) d3.select("g#bgT").remove();
        if (d3.select("g#bgT")) d3.select("g#bgT").remove();
        el.append("g")
        .attr("id", "bgT")
        .selectAll("rect")
        .data($countriesSumDiff)
        .join("rect")
        .attrs({
            width: svgWidth,
            height: svgHeight / dataLength - 3,
            x: 0,
            y: (d, i) => yScale(i) + 5,
            fill: (d) => (nations.includes(d.name) ? "black" : "white"),
        });
    };
    const setSvgTextTitles = function (el) {
        if (d3.select("g#textG")) d3.select("g#textG").remove();
        el.append("g")
        .attr("id", "textG")
        .selectAll("text")
        .data($countriesSumDiff)
        .join("text")
        .text((d) => d.name)
        .attrs({
            id: (d) => d.iso,
            x: svgWidth - 5,
            y: (d, i) => yScale(i) + 5 + 18,
        })
        .styles({
            fill: (d) =>
            nations.includes(d.name) ? colors[nations.indexOf(d.name)] : "black",
            "text-anchor": "end",
            "font-size": "14px",
            "font-weight": 600,
            cursor: "pointer",
        })
        .on("click", selectCountries);
    };

    onMount(() => {
        svgtext = d3.select("svg#text");
        svgWidth = svgtext.node().clientWidth;
        svgHeight = svgtext.node().clientHeight;
        setSvgTextScale();
        setSvgTextBg(svgtext);
        setSvgTextTitles(svgtext);
    });
    afterUpdate(() => {
        svgtext = d3.select("svg#text");
        svgWidth = svgtext.node().clientWidth;
        svgHeight = svgtext.node().clientHeight;
        setSvgTextScale();
        setSvgTextBg(svgtext);
        setSvgTextTitles(svgtext);
    });
</script>

<svg id="text" />

<style>
    svg#text {
        background-color: rgb(238, 239, 240);
        width: 12rem;
        height: 60rem;
    }
</style>
