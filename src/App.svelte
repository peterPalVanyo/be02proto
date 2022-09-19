<script>
// @ts-nocheck
  import { useData } from "./assets/data";
  import { onMount } from "svelte";
  import Ranking from './lib/Ranking.svelte'
  import Titles from './lib/Titles.svelte'
  import Trend from './lib/Trend.svelte'
  import * as d3 from 'd3'
  import {rgb} from 'd3'
  import 'd3-attrs';

  $: rankingBasis = 2024;
  $: ranges = useData.getRange(rankingBasis);
  let dataR = useData.getSumData();
  const dataLength = dataR.length;


  ////new
  $: indicators = ['res']
  $: nations = ['United Kingdom']
  $: trends = [useData.selectTrend('United Kingdom', 'res')];
  $: mergedNations = [] 

  $: console.log('merged nations:' , mergedNations);
  $: console.log('indicators length', indicators.length)
  $: console.log('check trends', trends)

  $: indicatorsDisplay = ['Residential']
  function add() {
    if (rankingBasis >= 2024) {
      rankingBasis = 2024;
    } else {
      rankingBasis += 1;
    }
  }
  function subtract() {
    if (rankingBasis <= 2022) {
      rankingBasis = 2022;
    } else {
      rankingBasis -= 1;
    }
  }
  const selectIndicators = function(e) {
    if(!indicatorsDisplay.includes(e.target.textContent)) {
      indicatorsDisplay = [...indicatorsDisplay, e.target.textContent];
      indicators = [...indicators, e.target.value];
    } else if(indicatorsDisplay.length<=1) {
      indicatorsDisplay = indicatorsDisplay;
      indicators = indicators;
    } else {
      indicatorsDisplay = indicatorsDisplay.filter(i=>i !== e.target.textContent);
      indicators = indicators.filter(i=> i !== e.target.value);
    }
    let newTrends = []
    nations.forEach(nat=>{
      indicators.forEach(ind=>{
        newTrends.push(useData.selectTrend(nat, ind))
      })
    })
    trends = [...newTrends]
  } 
  /////new end
  console.log("finally", dataR);


  let xScale = null;
  let yScale = null;
  let svgWidth = 0;
  let svgHeight = 0;
  let svgRight;
  //colors?

  
  //selected trend missing
  //selectcountry missing
  $: rangeTrend = useData.getRanges(indicators)
  //$: trendsCou = useData.selectNations(nations)
  $: trendsInd = useData.selectTrends(nations, indicators)
  
  
  const handleNewNations = function(e){
    nations = e.detail
    let newTrends = []
    nations.forEach(nat=>{
      indicators.forEach(ind=>{
        newTrends.push(useData.selectTrend(nat, ind))
      })
    })
    trends = [...newTrends]
  }

  const handleMerge = function(){
    let nats = []
    nations.forEach(n=>{
      let first = trends.find(t=>t[0].name == n);
      nats.push([...first]);
    })
    nats.forEach(nat=>{
      trends.forEach(t=>{
        if(t[0].name == nat[0].name && t[0].ind !== nat[0].ind) {
          t.forEach(ty=>{
            nat[t.indexOf(ty)].val = nat[t.indexOf(ty)].val+ty.val})
        }
      })
    })
    mergedNations = [...nats]
    indicators = ['res']
    let newTrends = []
    nations.forEach(nat=>{
      indicators.forEach(ind=>{
        newTrends.push(useData.selectTrend(nat, ind))
      })
    })
    trends = [...newTrends]
  }

  onMount(()=>{
})

const handleZeroMerged = function(){
  mergedNations = [];
  indicators = ['res'];
  indicatorsDisplay = ["Residential"]
}
function isMergedAllowed(indicators){
  return indicators.length<2;
}

</script>

<main>
  <div class="grid" id="item1">
    <div>
      <button disabled="{isMergedAllowed(indicators)}" on:click={handleMerge}>Aggregate</button>
      INDICATORS: {indicatorsDisplay}
    </div>
    <div>
      RANKING
      <button on:click={add}>+</button>
      {rankingBasis}
      <button on:click={subtract}>-</button>
    </div>
  </div>
  <div class="grid" id="item2">
    <div>
      <button class:selected={indicatorsDisplay.includes("Residential")} value="res" on:click={selectIndicators}>Residential</button>
      <button class:selected={indicatorsDisplay.includes("Non-residential")} value="nonres" on:click={selectIndicators}>Non-residential</button>
      <button class:selected={indicatorsDisplay.includes("Civil")} value="civil" on:click={selectIndicators}>Civil</button>
    </div>
  </div>
  <div class="grid" id="item3">
    <Trend {trends} {rangeTrend} {nations} {indicators} {rankingBasis} {mergedNations} on:newMerged={handleZeroMerged}></Trend>
  </div>
  <div class="grid" id="item4">
    <Titles {rankingBasis} {dataLength} {nations} on:newNations={handleNewNations}></Titles>
  </div>
  <div class="grid" id="item5">
    <Ranking {rankingBasis} {ranges} {dataLength} {nations}></Ranking>
  </div>
</main>

<style>
  .grid {
    font-size: 2.5rem;
    text-align: center;
  }
  main {
    width: 100vw;
    height: 100vh;
    padding: 0 1rem;
    background-color: rgb(79,79,79);
    display: grid;
    gap: 1px;
    grid: max-content 1fr / max-content 2fr max-content 1fr;
  }
  main .grid {
    display: flex;
    align-items: center;
  }
  main .grid#item1 {
    background-color: rgb(79,79,79);
    color: white;
    grid-column: 2 / span 3;
    grid-row: 1 / span 1;
    padding: 1rem;
    display: flex;
    justify-content: space-between;
  }
  main .grid#item1 div {
    display: flex;
    align-items: center;
    justify-content: space-around;
    min-width: 40rem;
  }
  main .grid#item1 div button {
    margin-right: 1rem;
  }
  main .grid#item2 {
    grid-column: 1 / span 1;
    grid-row: 1 / span 2;
    background-color: rrgb(79,79,79);
  }
  main .grid#item2 div {
    display: flex;
    height: 30%;
    justify-content: space-around;
    flex-direction: column;
  }
  main .grid#item2 div button {
    padding: 1rem 1.5rem;
    margin: 0 1rem;
    font-weight: 300;    
  }
  main .grid#item2 div button.selected {
    background: #222;
    color: #fff;
  }
  main .grid#item2 div button.selected:nth-child(2) {
    color: rgba(255, 255, 255, .5);
  }
  main .grid#item2 div button.selected:nth-child(3) {
    background-color: #222;
    background-image: radial-gradient(#fff 5%, transparent 15%),
      radial-gradient(#fff 10%, transparent 10%);
    background-size: 10px 10px;
    background-position: 0 0, 10px 10px;
    background-repeat: repeat;
  }
  main .grid#item3 {
    background-color: rgb(125,125,125);
  }
  main .grid#item4 {
    background-color: rgb(125,125,125);
    grid-column: 3 / span 1;
    grid-row: 2 / span 1;
  }
  main .grid#item5 {
    grid-column: 4 / span 1;
    background-color: rgb(125,125,125);
  }

  button {
    display: inline-block;
    outline: none;
    cursor: pointer;
    font-size: 1.6rem;
    line-height: 2rem;
    font-weight: 600;
    border-radius: 8px;
    padding: 1.3rem 2.3rem;
    border: 1px solid #222222;
    transition: box-shadow 0.2s ease 0s, -ms-transform 0.1s ease 0s,
      -webkit-transform 0.1s ease 0s, transform 0.1s ease 0s;
    background: #fff;
    color: #222222;
  }
  button:hover {
    border-color: #000000;
    background: #f7f7f7;
  }
  button:disabled {
    background-color: rgb(125,125,125);
  }
</style>
