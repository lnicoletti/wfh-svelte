<script>
import {cross, range} from "d3";
export let colors;
export let country;

$: outerWidth = 0
$: innerWidth = 0
$: outerHeight = 0
$: innerHeight = 0

let title = ["Travel to Work", "Med. Income"]
let n = Math.floor(Math.sqrt(colors.length))
let legendData = cross(range(n), range(n))
const k = 63/n
let labels = ["low", "medium", "high"]

</script>
<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight /> 
<div class="legendContainer"> <!-- width={100} height={100} display=flex justify-content=center-->
    <svg width={100} height={100} display=block margin=auto transform="translate({innerWidth>850&&country==="UK"?innerWidth*0.3:0},{innerWidth>850?0:-50})"> <!-- transform="translate({innerWidth/2-50},{0})" -->
      <g class="legend" transform="translate({15},{15})">  <!-- transform="translate({innerWidth/2},{margin.top/2})"-->
          <marker id="arrow" markerHeight=10 markerWidth=10 refX=3 refY=3 orient=auto>
              <path d="M0,0L6,3L0,6Z" fill=#445312/>
          </marker>
          {#each legendData as [i, j], idx}
              <circle 
              r={10} 
              cx={(i * k)+k/2} 
              cy={((n - 1 - j) * k)+k/2} 
              fill={colors[j * n + i]} 
              class="legendCircle" 
              value={colors[j * n + i]}
              >
                  <title>
                      {title[0]}{labels[j] && ` (${labels[j]})`}
                      {title[1]}{labels[i] && ` (${labels[i]})`}
                  </title>
              </circle>
          {/each}
          <line marker-end="url(#arrow)" x1=0 x2={n * k} y1={n * k} y2={n * k} stroke=#445312 stroke-width=1.5 />
          <line marker-end="url(#arrow)" y2=0 y1={n * k} stroke=#445312 stroke-width=1.5 />
          <text class="legendTicks" dy="0.71em" transform="rotate(90) translate({n / 2 * k},6)" text-anchor="middle">{title[0]}</text>
          <text class="legendTicks" dy="0.71em" transform="translate({n / 2 * k},{n * k + 6})" text-anchor="middle">{title[1]}</text>
      </g>
    </svg>
  </div>

  <style>
    @font-face {
    font-family: 'Lato', sans-serif;
    src: url('https://fonts.googleapis.com/css2?family=Lato&display=swap');
}

@font-face {
    font-family:'Texturina', serif;
    src: url('https://fonts.googleapis.com/css2?family=Texturina:ital,wght@1,300&display=swap');
    font-weight: normal;
    font-style: normal;
}

.svg-container {
    /* max-width:1000px; */
    max-width:620px;
    /* min-width:100px; */
    /* max-height:600px; */
    /* max-width: 80vw;
    max-height:80vh; */
}

.legendTicks {
        font-family: 'Roboto', sans-serif;
        /* font-family: 'DotGothic16', sans-serif; */
        /* font-size: 8px; */
        font-size: calc(8px + 0.15vw);
        font-weight: 400;
        fill: black;
		    text-transform: uppercase;
    }

.legendContainer {
    width:100px;
    height:100px;
    margin:auto;
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


  .tick text {
    font-family:'Lato', sans-serif;
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
    font-family:'Lato', sans-serif;
    /* font-size: 10px; */
    /* font-size:11px; */
		fill: #445312;
    font-weight: 700;
    text-transform: uppercase
	}


.legendTitle {
  font-family:'Lato', sans-serif;
  font-size: 10px;
  font-weight: 200;
  text-transform: None;
  fill: #445312;
}

.regionAnnot {
font-family:'Lato', sans-serif;
  font-size: 12px;
  font-weight: 900;
  text-transform: None;

}

.LStext {
  font-family:'Lato', sans-serif;
  font-size:12px;
  font-weight: bold;
  text-transform: capitalize;
}

.LStextUK {
  font-family:'Lato', sans-serif;
  text-transform: capitalize;
}

.legx-axis line, .legx-axis path { stroke: #fff; }

.Axis {
  font-family:'Lato', sans-serif;
  font-size:6px;
  text-transform: capitalize;
}

.AxisBig {
  font-family:'Lato', sans-serif;
  font-size:6px;
  text-transform: capitalize;
}

.AxisLAN{
  font-family:'Lato', sans-serif;
  font-size:10px;
  text-transform: capitalize;
}

.AxisMonth {
  font-family:'Lato', sans-serif;
  font-weight: bold;
  font-size: 12px
}

.AxisWide {
  font-family:'Lato', sans-serif;
  text-transform: capitalize;
  font-size:12px;
}

.axisGrid {
  font-family:'Lato', sans-serif;
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
  font-family:'Lato', sans-serif;
  font-size:6px; 
}

.hoverAnnotation {
  font-family:'Lato', sans-serif;
  font-size:6px; 
  font-weight: 700;
}

.policyAnnotation {
  font-family:'Lato', sans-serif;
  font-size:11.5px; 
}

.cityAnnot {
  font-family:'Lato', sans-serif;
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
  font-family:'Lato', sans-serif;
  font-size: 11px;
  text-transform: None;
}

.subtitle {
  font-family:'Lato', sans-serif;
  text-align:center;
  font-size:20px;
  font-weight:700;
  transform: translate(100px, 0px);
}

.subsubtitle {
  font-family:'Lato', sans-serif;
  text-align:center;
  font-size:17px;
  font-weight:700;
  transform: translate(100px, 0px);
}

.title {
  font-family:'Lato', sans-serif;
  text-align:center;
  transform: translate(100px, 0px);
}

.paragraph {
 font-family:'Georgia';
 text-align: justify;
 transform: translate(100px, 0px);
}

/* .legend {
    font-family: 'Lato', sans-serif;
    font-weight: 600;
    font-size: calc(6px + 0.2vw);
    text-transform:uppercase;
} */

.titleContainer {
    /* max-width:850px; */
    position: relative;
    /* align-items: center; */
}

.content {
    max-width: 840px; /* Can be in percentage also. */
    height: auto;
    margin: 0 auto;
    padding: 10px;
    position: relative;
    text-align: justify;
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

    font-family: 'Lato', sans-serif;
    font-weight: 700;
    font-size: calc(6px + 0.4vw);
    text-align: center;
    /* transform: translate(10px,93px) */
    /* transform: translate(10px, calc(35px + 3.5vw)) */
    /* font-size: 12px; */
}

.FigTitleBig {
    font-family:'Lato', sans-serif;
    font-size: 50px;
    font-weight: 900;
    /* height:130px; */
    text-transform: uppercase;
    text-align: center;
    /* margin-bottom: 10px; */
  }

  #FigTitle {
    font-family:'Lato', sans-serif;
    font-size: 40px;
    font-weight: 500;
    height:53px;
    vertical-align:text-bottom;
    text-transform: uppercase;
    text-align: center;
  }
  
  .FigSubtitle {
    font-family:'Lato', sans-serif;
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
    font-family: 'Lato', sans-serif;
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
      font-family: 'Lato', sans-serif;
  }

  .annotation-group {
    font-family: 'Lato', sans-serif;
    /* font-size: 15px; */
    opacity:0.5
  }

  .annotation-note-label {
    font-family: 'Lato', sans-serif;
    font-size: 10px;
  }


  /* .annotation-connector {

  } */
</style>