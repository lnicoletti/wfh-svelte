<script>
    import * as d3 from "d3";
    import { extent } from "d3";``
    export let data_sm;
    export let nation;
    export let colors;

    let width = 250
    let height = 200
    let margin = ({ top: 0.05*height, right: 0.04*width, bottom: 0.2*height, left: 0.2*width})
    let numTicksX = 4
    let numTicksY = 4
    let radius = 3

    $: topQuantileX = d3.quantile(data_sm.filter(d=>d.country===nation&&d.income!==null&&d.income!==0).map(d=>d.income), 0.98)
    $: topQuantileY = d3.quantile(data_sm.filter(d=>d.country===nation&&d.income!==null&&d.income!==0).map(d=>d.mobilityWork), 0.02)
    // $: console.log("95% quantile income", topQuantileX)
    // $: console.log("5% quantile mob", topQuantileY)
    $: data = data_sm.filter(d=>d.country===nation&&d.income!==null&&d.income!==0)//&&d.income<topQuantileX&&d.mobilityWork>topQuantileY)
    // bivar settings
    $: dataBivar = Object.assign(new Map(data.map(d=>[d.province, [d.mobilityWork, d.income]])), {title: ["Travel to Work", "Med. Income"]})

    // $: console.log("bivar", dataBivar)
                                   
    $: n = Math.floor(Math.sqrt(colors.length))
    $: yBivar = d3.scaleQuantile(Array.from(dataBivar.values(), d => d[1]), d3.range(n))
    $: xBivar = d3.scaleQuantile(Array.from(dataBivar.values(), d => d[0]), d3.range(n))
    // let colorBivar = function(value) {
    //         if (!value) return "lightgrey";
    //           let [a, b] = value;
    //           return colors[yBivar(b) + xBivar(a) * n];
    //     }

    let colorBivar = function(value) {
        if (!value) return "lightgrey";
        let [a, b] = value;
        if (!b) return "lightgrey";
        if (!b) return "lightgrey";
        return colors[yBivar(b) + xBivar(a) * n];
    }

    // $: console.log("scatter data for", nation, data)

    $: extentX = d3.extent(data, d => d.income)
    // $: console.log(extentX)
    $: extentY = d3.extent(data, d => d.mobilityWork)
    $: intervalX = (extentX[1] - extentX[0])/numTicksX
    $: intervalY = (extentY[1] - extentY[0])/numTicksY

    $: xScale = d3.scaleLinear()
                        .domain(extentX)
                        .range([margin.left, width - margin.right]).nice()
      
    $: yScale = d3.scaleLinear()
                        .domain(extentY)
                        .range([height - margin.bottom, margin.top]).nice()

    $: xTicksIncome = d3.range(extentX[0], extentX[1], intervalX);
    $: yTicksMob = d3.range(extentY[0], extentY[1], intervalY);


    $: outerWidth = 0
    $: innerWidth = 0
    $: outerHeight = 0
    $: innerHeight = 0

    $: if (innerWidth<450) {
        d3.selectAll(".smScatterChild").style("flex", "1 0 50%")
    } else if (innerWidth>450&&innerWidth<750) {
        d3.selectAll(".smScatterChild").style("flex", "1 0 33%")
    } else {
        d3.selectAll(".smScatterChild").style("flex", "1 0 25%")
    }

</script>
<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight />
<svg class="smScatterChild">
    <g class='axis y-axis'>
        {#each yTicksMob as tick}
          <g class='tick tick-{tick}' transform='translate(0, {yScale(tick)})'>
            <line x1='{margin.left}' x2='{width-margin.right}'/>
            <text x='{margin.left - 8}' y='+4' font-size={"12px"}>{tick.toFixed()}</text>
          </g>
        {/each}
      </g>
      <g class='axisTitle' text-anchor=start transform='translate({margin.left*0.4}, {height}) rotate(-90)'>
        <text x={margin.left} y="+0" font-size={"12px"}>{nation==="United Kingdom"?"TRAVEL TO WORK (%)":""}</text>
      </g>
      <g class='axisTitle' text-anchor=start transform='translate(0, {10})'>
        <text x={margin.left} y="+0" font-size={"12px"}>{nation}</text>
      </g>
      <!-- x axis -->
      <g class='axis x-axis'>
        {#each xTicksIncome as tick}
          <g class='tick' transform='translate({xScale(tick)},0)'>
            <line y1='{yScale(0)}' y2='{yScale(d3.min(yTicksMob))}'/>
            <text y='{height - margin.bottom + 16}' font-size={"12px"}>{(tick/1000).toFixed()+"K"}</text>
          </g>
        {/each}
      </g>
      <g class='axisTitle' text-anchor=end transform='translate({xScale(extentX[1])}, {height - margin.bottom})'>
        <text x={0} y="+35" font-size={"12px"}>{nation==="Sweden"?"MED. HOUSEHOLD INCOME (USD)":""}</text>
      </g>
    <g>
    {#each data as d, i}
    <circle
    cx={xScale(d.income)}
    cy={yScale(d.mobilityWork)}
    r={radius}
    fill={colorBivar([d.mobilityWork, d.income])}
    stroke="#fafafa"
    stroke-width=0.1
    ></circle>
    {/each}
    </g>
</svg>

<style>
.smScatterChild {
  flex: 1 0 25%;
  margin: 5px;
  height: 200px;
  width: 200px;
  /* background-color: blue; */
}


.legendContainer {
    /* width:100vw; */
    height:45px;
    margin:auto;
    text-align: center;
}

.searchDropdown {
  max-width: 250px;
  text-align: center;
  margin:auto;
}

.spacer {
    height: 40vh;
  }

  .richtheNytimes {
    background-color:#449d57;
    font-weight:900;
    color:#fafafa
  }

  .poortheNytimes {
    background-color:#c17036;
    font-weight:900;
    color:#fafafa
  }

/* .sv-control {
  width:20vw;
} */

.chart {
    /* background: whitesmoke; */
    /* width: 400px;
    height: 400px; */
    /* box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2); */
    position: sticky;
    top: 5vh;
    margin: auto;
    height:100%
    /* display:table; */
    /* position: relative; */
    /* height: 100vh */
    /* bottom: 50%; */
  }

  /* .titleElements {
    display:table-cell;
    vertical-align:middle
  } */

  /* Scrollytelling CSS */
  .step {
    height: 150vh;
    display: flex;
    place-items: center;
    justify-content: center;
  }

  .step-content {
    background: #eeeeee;
    color: #ccc;
    border-radius: 5px;
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2);
    z-index: 10;
    font-family: 'Roboto', sans-serif;
    font-weight: 400;
  }

  .step.active .step-content {
    background: rgb(255, 255, 255, 0.9);
    color: black;
  }

  /* input {
    max-width: 100px
  } */

  /* SCROLLY END */

.svg-container {
    /* max-width:1000px; */
    max-width:620px;
    /* min-width:100px; */
    /* max-height:600px; */
    /* max-width: 80vw;
    max-height:80vh; */
}

/* .sv-control.svelte-v9xikc {
  background-color: #fafafa;
  border: 1px solid #445312;
  border-radius: 4px;
  min-height: 38px;
  width: 20vw;
  margin: auto;
} */

.svelecte.svelte-1lvkhl0 {
    position: relative;
    flex: 1 1 auto;
    width: 20vw;
    margin: auto;
    color: #fafafa;
}

/* .vizElement {
  max-width:400px;
} */

body, main {
    background-color: #fafafa;
}

.tick line {
		stroke: #445312;
    opacity:0.3;
		stroke-dasharray: 2;
    z-index: -100
	}

  .stateAbbrv {
    font-family: "Roboto", sans-serif;
  }


  .tick text {
    font-family: "Roboto", sans-serif;
		/* font-size: 10px; */
    /* font-size:11px; */
		fill: #445312;
	}

	.x-axis text {
		text-anchor: middle;
	}

	.y-axis text {
		text-anchor: end;
	}

  .axisTitle text {
    font-family:'Roboto', sans-serif;
    /* font-size: 10px; */
    /* font-size:11px; */
		/* fill: #445312; */
    fill: black;
    font-weight: 700;
    text-transform: uppercase
	}

  /* .axisText {
    background: #445312;
    color: #fafafa;
  } */


.legendTitle {
  font-family:'Roboto', sans-serif;
  font-size: 10px;
  font-weight: 200;
  text-transform: None;
  /* fill: #445312; */
  fill: black;

}

.regionAnnot {
font-family:'Roboto', sans-serif;
  font-size: 12px;
  font-weight: 900;
  text-transform: None;

}

.LStext {
  font-family:'Roboto', sans-serif;
  font-size:12px;
  font-weight: bold;
  text-transform: capitalize;
}

.LStextUK {
  font-family:'Roboto', sans-serif;
  text-transform: capitalize;
}

.legx-axis line, .legx-axis path { stroke: #fff; }

.Axis {
  font-family:'Roboto', sans-serif;
  font-size:6px;
  text-transform: capitalize;
}

.AxisBig {
  font-family:'Roboto', sans-serif;
  font-size:6px;
  text-transform: capitalize;
}

.AxisLAN{
  font-family:'Roboto', sans-serif;
  font-size:10px;
  text-transform: capitalize;
}

.AxisMonth {
  font-family:'Roboto', sans-serif;
  font-weight: bold;
  font-size: 12px
}

.AxisWide {
  font-family:'Roboto', sans-serif;
  text-transform: capitalize;
  font-size:12px;
}

.axisGrid {
  font-family:'Roboto', sans-serif;
  text-transform: capitalize;
  font-size:8px;
}

.Axis path,
.Axis line,
.axisGrid path,
.axisGrid line,
.AxisWide path,
.AxisWide line,
.AxisBig path,
.AxisBig line {
  stroke: lightgrey;
  stroke-width: 0.7px;
}

.annotation {
  font-family:'Roboto', sans-serif;
  font-size:6px; 
}

.hoverAnnotation {
  font-family:'Roboto', sans-serif;
  font-size:6px; 
  font-weight: 700;
}

.policyAnnotation {
  font-family:'Roboto', sans-serif;
  font-size:11.5px; 
}

.cityAnnot {
  font-family:'Roboto', sans-serif;
  font-size:12px;
  font-weight: 700;
}

.yAxisAnnot {
  text-anchor: start;
  font-weight: 700;
  font-size: 6;
  text-transform: None;
}

.sourceMeta {
  font-family:'Roboto', sans-serif;
  font-size: 11px;
  text-transform: None;
}

.subtitle {
  font-family:'Roboto', sans-serif;
  text-align:center;
  font-size:20px;
  font-weight:700;
  transform: translate(100px, 0px);
}

.subsubtitle {
  font-family:'Roboto', sans-serif;
  text-align:center;
  font-size:17px;
  font-weight:700;
  transform: translate(100px, 0px);
}

.title {
  font-family:'Roboto', sans-serif;
  text-align:center;
  transform: translate(100px, 0px);
}

.paragraph {
 font-family:'Georgia';
 text-align: justify;
 transform: translate(100px, 0px);
}

.legend {
    font-family: 'Roboto', sans-serif;
    font-weight: 600;
    /* font-size: 9.5px; */
    font-size: calc(6px + 0.2vw);
    text-transform:uppercase;
    /* transform: translate(30px, calc(100px + 5vw)) */
}

.titleContainer {
    /* max-width:850px; */
    position: relative;
    /* align-items: center; */
}

.content {
    max-width: 840px; /* Can be in percentage also. */
    height: auto;
    margin: 0 auto;
    /* padding: 10px; */
    position: relative;
    /* text-align: justify; */
}

.svg-container {
    display: inline-block;
    position: relative;
    width: 100%;
    /* padding-bottom: 100%; */
    vertical-align: top;
    overflow: hidden;
}
.svg-content {
    display: inline-block;
    position: absolute;
    top: 0;
    left: 0;
}

.mapCredit {

    font-family: 'Roboto', sans-serif;
    font-weight: 700;
    font-size: calc(6px + 0.4vw);
    text-align: center;
    /* transform: translate(10px,93px) */
    /* transform: translate(10px, calc(35px + 3.5vw)) */
    /* font-size: 12px; */
}

.FigTitleBig {
    font-family:'Roboto', sans-serif;
    font-size: 50px;
    font-weight: 900;
    /* height:130px; */
    text-transform: uppercase;
    text-align: center;
    /* margin-bottom: 10px; */
  }

  .frameTitle {
    font-family: 'DotGothic16', sans-serif;
    font-size: calc(8px + 1.2vw);
    font-weight: 500;
    /* height:53px;
    vertical-align:text-bottom; */
    /* text-transform: uppercase; */
    text-transform: lowercase;
    text-align: center;
    /* line-height: 1em; */
    /* margin-top: 0.8em; */
  }

  #FigTitle {
    font-family:'Roboto', sans-serif;
    font-size: 40px;
    font-weight: 500;
    height:53px;
    vertical-align:text-bottom;
    text-transform: uppercase;
    text-align: center;
  }
  
  .FigSubtitle {
    font-family:'Roboto', sans-serif;
    font-size: 14px;
    font-weight: 200;
    text-transform: None;
  }

  .richoriginal {
    background-color:#cbb35c;
    font-weight:900;
    color:#fafafa
  }

  .richtheEconomist {
    background-color:#747474;
    font-weight:900;
    color:#fafafa
  }

  .richtheNytimes {
    background-color:#449d57;
    font-weight:900;
    color:#fafafa
  }

  .poororiginal {
    background-color:#9c75b4;
    font-weight:900;
    color:#fafafa
  }

  .poortheEconomist {
    background-color:#e83000;
    font-weight:900;
    color:#fafafa
  }

  .poortheNytimes {
    background-color:#c17036;
    font-weight:900;
    color:#fafafa
  }

  .richHigh {
    background-color:#804d36;
    font-weight:900;
    color:#fafafa
  }

  .richHightheNytimes {
    background-color:rgb(2, 83, 2);
    font-weight:900;
    color:#fafafa
  }

  .poorLow {
    background-color:#e8e8e8;
    font-weight:900;
    color:#fafafa
  }

  .poorLowtheNytimes {
    background-color:#e8e8e8;
    font-weight:900;
    color:#fafafa
  }
  

  .custom-select {
    position: relative;
    font-family: 'Roboto', sans-serif;
    text-transform: uppercase;
  }

  #chartView, #chartCountry {
      background-color: #fafafa;
      width: 120px;
      height:30px;
      color:black;
      font-weight:500;
      font-size: 12px;
      text-transform: uppercase;
      border-color: black
  }

  #chartView-selected {
    border-color: lightgrey
  }

  #staticTooltip {
      height: 30px;
      margin-top:10px;
      font-family: 'Roboto', sans-serif;
  }

  .annotation-group {
    font-family: 'Roboto', sans-serif;
    /* font-size: 15px; */
    /* opacity:0.5 */
  }

  .annotation-note-label {
    font-family: 'Roboto', sans-serif;
    font-size: 10px;
  }


  /* .annotation-connector {

  } */
</style>