<script>
// @ts-nocheck
    import { countries, useData, countriesSumDiff } from "../assets/data";
    import { onMount, afterUpdate } from "svelte";
    import { slide } from 'svelte/transition';
    import * as d3 from 'd3'
    import {rgb} from 'd3'
    import 'd3-attrs';

    export let rankingBasis, ranges, dataLength, nations;
    let xScale = null;
    let yScale = null;
    let svgWidth = 0;
    let svgHeight = 0;
    let svgRight;
    let colors = [rgb(241,0,84), rgb(67,64,164), rgb(80,138,0)]

    const setSvgRightScale = function() {
        xScale = d3.scaleLinear().domain([ranges[0], ranges[1]]).range([0,svgWidth-10]);
        yScale = d3.scaleLinear().domain([0, dataLength-1]).range([0,svgHeight-35]);
        console.log('changes', xScale(10))
        $countriesSumDiff.sort(function(a,b){return b[`${rankingBasis}a`]-a[`${rankingBasis}a`]});
    }
    const setSvgRightBg =  function(el) {
        if(d3.select('g#bg')) d3.select('g#bg').remove()
        el.append('g').attr('id', 'bg').selectAll('rect').data($countriesSumDiff).join('rect').attrs({
        width: xScale(ranges[1]),
        height: svgHeight/dataLength-3,
        x: 0,
        y:(d,i)=>yScale(i)+5,
        fill: rgb(238,239,240),
        })
    }
    const setSvgRightBars = function(el) {
        if(d3.select('g#rightbar')) d3.select('g#rightbar').remove()
        el.append('g').attr('id', 'rightbar').style('filter','drop-shadow(1px 3px 3px rgb(0 0 0 / 0.2))').selectAll('rect').data($countriesSumDiff).join('rect').attrs({
        width: 0,
        height: svgHeight/dataLength-3,
        x:d=>{
            if(d[`${rankingBasis}a`]>0) return xScale(0)
            return xScale(d[`${rankingBasis}a`])},
        y:(d,i)=>yScale(i)+5,
        fill: d=>nations.includes(d.name)?colors[nations.indexOf(d.name)]:rgb(195,197,196) 
        }).styles({
            stroke: 'rgba(0,0,0,0.8)',
            'stroke-width': .5,
            }).transition().duration(400).delay(200).ease(d3.easeBounceOut).attr('width', d=>{
            if(d[`${rankingBasis}a`]>0) return xScale(d[`${rankingBasis}a`])-xScale(0)
            return xScale(0)-xScale(d[`${rankingBasis}a`])})
    }
    const setSvgRightValues = function(el) {
        if(d3.select('g#rankvalues')) d3.select('g#rankvalues').remove()
        el.append('g').attr('id', 'rankvalues').selectAll('text').data($countriesSumDiff).join('text').text(d=>`${Math.round(d[`${rankingBasis}a`])}%`).attrs({
        x: svgWidth-5,
        y: (d,i)=>yScale(i)+5+18
        }).styles({
        'fill': "black",
        'text-anchor': 'end',
        'font-size': '14px',
        'font-weight': 500
        })
    }

    onMount(()=>{
        console.log('mount')
        svgRight= d3.select('svg#right');
        svgWidth = svgRight.node().clientWidth;
        svgHeight = svgRight.node().clientHeight;
        //RANKING svg right
        setSvgRightScale()
        //RANKING bar background
        setSvgRightBg(svgRight)
        //RANKING bars
        setSvgRightBars(svgRight)
        //RANGING textvalues
        setSvgRightValues(svgRight)        
    })
    
    afterUpdate(()=>{
        console.log('update')
        svgRight= d3.select('svg#right');
        svgWidth = svgRight.node().clientWidth;
        svgHeight = svgRight.node().clientHeight;
        //RANKING svg right
        setSvgRightScale()
        //RANKING bar background
        setSvgRightBg(svgRight)
        //RANKING bars
        setSvgRightBars(svgRight)
        //RANGING textvalues
        setSvgRightValues(svgRight)        
    })

</script>


<svg id="right"></svg>

<style>
    svg#right {
        background-color: white;
        width: 100%;
        height: 60rem;
    }
</style>
