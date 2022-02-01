<script>
	import {csv, json, merge, autoType} from "d3"
	import { onMount } from "svelte";
	import Title from "./components/Title.svelte";
  // button
  // import Chart from "./components/Chart.svelte";
  // scrolly
	import Chart from "./components/ChartScrolly.svelte";
	import Footer from "./components/Footer.svelte";
  import Legend from "./components/Legend.svelte";

	let isLoading = true;

	// initialize data variables
	let hex_la = []
	let ukUpd_tot = []
	let ukUpd_time = []
	let engUrbRural = []
	let scotUrbRural = []
	let ukUrbRural = []
	let uSuPd_tot = []
	let hex_us = []

	// color scheme
	const vizTheme = "theNytimes"
	// const vizTheme = "original"
			// color settings
	const colorSchemes = new Map([{scheme:"original", colors:["#e8e8e8", "#e4d9ac", "#c8b35a", "#cbb8d7", "#c8ada0", "#af8e53", "#9972af", "#976b82", "#804d36"]}, 
                        {scheme:"theEconomist", colors:["#d3d3d3", "#a3a3a3", "#747474", "#db9381", "#a97264", "#795147", "#e83000", "#b32500", "#801b00"]}, 
                        {scheme:"theNytimes", colors:["#e6e6e6", "#99c5a2", "#47a45b", "#ddb08e", "#939765", "#457e38", "#d5742f", "#8d6421", "#425312"]}].map(obj => [obj.scheme, obj.colors]))

	let colors = colorSchemes.get(vizTheme)

  let countryOptions = [
		{ id: 1, value: `UK`, text: `United Kingdom`},
		{ id: 2, value: `US`, text: `United States`}
	];

  $: selectedCountry = countryOptions[0];

	onMount( async () => {
		await Promise.all([
					// uk base data
					json("https://raw.githubusercontent.com/odileeds/hexmaps/gh-pages/maps/uk-local-authority-districts-2021.hexjson"),
					json("https://gist.githubusercontent.com/lnicoletti/2e08ca8357c8d4e4cf9bf869d890ab99/raw/a8135c1175715c9d8715be58de2ce8752fe236a4/ukUpd_tot_final_reduced.json"),
					json("https://gist.githubusercontent.com/lnicoletti/8025f10314c9004f5c0d9e392bbf5b17/raw/f5fa0d5c86d1b8d41f9ed6624d6b23b0b1b2ec37/ukUpdGroupMonth"),
					////uk urb rural
					csv("https://gist.githubusercontent.com/lnicoletti/9b5df10403fd89cc6af0ee8cd26b7925/raw/cacd508ed040a1d190401e9b16247b970181aac1/ukUrbRural_2class.csv", autoType),
					csv("https://gist.githubusercontent.com/lnicoletti/1117ef6f526534506e96f981758a5602/raw/e0611a8a1ce43b9debb8c04465d8a67a054e6b2c/urbanRuralScotland_2class.csv", autoType),

					// US data
					json("https://gist.githubusercontent.com/lnicoletti/abbd577e047a3f501430869a2b6833ca/raw/2de81f38eaf396c5c2d1d75901660c9f86b10e64/usUpd_tot_fixed_reduced.json", autoType),
					csv("https://gist.githubusercontent.com/lnicoletti/a01622da3e106830fe1c51ab96403234/raw/2b0d35c4efff50a65beb29759ae1a55eb99f73ac/usCountiesHex_tilemaps_albers.csv")
					])
					.then((datasets)=>{
						hex_la = datasets[0]
						ukUpd_tot = datasets[1]
						ukUpd_time = datasets[2]
						engUrbRural = datasets[3]
						scotUrbRural = datasets[4]
						uSuPd_tot = datasets[5]
						hex_us = datasets[6]

						ukUrbRural = merge([engUrbRural, scotUrbRural])
					})
		isLoading = !isLoading;
	})


// resize view reactivity
$: view = "horizontal"
$: outerWidth = 0
$: innerWidth = 0
$: outerHeight = 0
$: innerHeight = 0

</script>

<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight />
<main>
	{#if isLoading}
	<h2>Loading...</h2>
	{:else }
	<!-- <h2>Data has been loaded!</h2> -->
	<body>
		<div class="content">
			<Title {vizTheme}/>
			<br>
      <Legend {colors}></Legend>
      <!-- <div style="text-align:center" class="custom-select">
        <span class="mapCredit">SELECT COUNTRY AND VIEW</span><br>
        <select id="chartCountry" bind:value={selectedCountry}>
          {#each countryOptions as option}
            <option value={option}>
              {option.text}
            </option>
          {/each}
        </select>
        {#if selectedCountry.value==="UK"}
        <Chart {colors} {hex_la} {ukUpd_tot} {ukUpd_time} {ukUrbRural} {uSuPd_tot} {hex_us} country={"UK"}/>
        {:else if selectedCountry.value==="US"}
        <Chart {colors} {hex_la} {ukUpd_tot} {ukUpd_time} {ukUrbRural} {uSuPd_tot} {hex_us} country={"US"}/>
        {/if}
        </div> -->
        <section>
          <Chart {colors} {hex_la} {ukUpd_tot} {ukUpd_time} {ukUrbRural} {uSuPd_tot} {hex_us} country={"UK"}/>
        </section>
        <section>
          <div>
            <h2>Transition</h2>
          </div>
        </section>
        <section>
          <Chart {colors} {hex_la} {ukUpd_tot} {ukUpd_time} {ukUrbRural} {uSuPd_tot} {hex_us} country={"US"}/>
        </section>
        <section>
          <div>
            <h2>Transition</h2>
          </div>
        </section>
			<Footer {vizTheme}/>
		</div>
	</body>
	{/if}
</main>

<style>
	/* main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}


	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	} */

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

body, main {
    background-color: #fffae7;
}


.legendTitle {
  font-family:'Lato', sans-serif;
  font-size: 10px;
  font-weight: 200;
  text-transform: None;
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

.legend {
    font-family: 'Lato', sans-serif;
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
    color:#fffae7
  }

  .richtheEconomist {
    background-color:#747474;
    font-weight:900;
    color:#fffae7
  }

  .richtheNytimes {
    background-color:#449d57;
    font-weight:900;
    color:#fffae7
  }

  .poororiginal {
    background-color:#9c75b4;
    font-weight:900;
    color:#fffae7
  }

  .poortheEconomist {
    background-color:#e83000;
    font-weight:900;
    color:#fffae7
  }

  .poortheNytimes {
    background-color:#c17036;
    font-weight:900;
    color:#fffae7
  }

  .richHigh {
    background-color:#804d36;
    font-weight:900;
    color:#fffae7
  }

  .richHightheNytimes {
    background-color:rgb(2, 83, 2);
    font-weight:900;
    color:#fffae7
  }

  .poorLow {
    background-color:#e8e8e8;
    font-weight:900;
    color:#fffae7
  }

  .poorLowtheNytimes {
    background-color:#e8e8e8;
    font-weight:900;
    color:#fffae7
  }
  

  .custom-select button {
    position: relative;
    font-family: 'Lato', sans-serif;
    text-transform: uppercase;
  }

  #chartView, #chartCountry {
      background-color: #fffae7;
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
/* 
  .annotation-group {
    font-family: 'Lato', sans-serif;
    font-size: 15px;
  } */

  .annotation-note-label {
    font-family: 'Lato', sans-serif;
    font-size: 10px;
  }
</style>