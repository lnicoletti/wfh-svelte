<script>
	import {csv, json, merge, autoType, selectAll} from "d3"
	import { onMount } from "svelte";
	import Title from "./components/Title.svelte";
  import * as animateScroll from "svelte-scrollto";
  // button
  // import Chart from "./components/Chart.svelte";
  // scrolly
	import Chart from "./components/ChartScrolly.svelte";
	import Footer from "./components/Footer.svelte";
  import Legend from "./components/Legend.svelte";
  import CountryScatter from "./components/CountryScatter.svelte";
  

  import {onlyUnique} from "./utils.js"
  // import * as animateScroll from "svelte-scrollto";
  

	let isLoading = true;

	// initialize data variables
	let data_UK = []
	let data_US = []
  let data_sm = []
  let nations = []

	// color scheme
	const vizTheme = "theNytimes"
	// const vizTheme = "original"
			// color settings
	const colorSchemes = new Map([{scheme:"original", colors:["#e8e8e8", "#e4d9ac", "#c8b35a", "#cbb8d7", "#c8ada0", "#af8e53", "#9972af", "#976b82", "#804d36"]}, 
                        {scheme:"theEconomist", colors:["#d3d3d3", "#a3a3a3", "#747474", "#db9381", "#a97264", "#795147", "#e83000", "#b32500", "#801b00"]}, 
                        {scheme:"theNytimes", colors:["#e6e6e6", "#99c5a2", "#47a45b", "#ddb08e", "#939765", "#457e38", "#d5742f", "#8d6421", "#425312"]}].map(obj => [obj.scheme, obj.colors]))

	let colors = colorSchemes.get(vizTheme)

  let countryOptions = [
    { id: 0, value: null, text: `Select a country`},
    { id: 1, value: `US`, text: `United States`},
		{ id: 2, value: `UK`, text: `United Kingdom`}
	];

  $: selectedCountry = countryOptions[0];

  
  function revertFrameOne() {
    animateScroll.scrollTo({element: '#start'})
    // selectAll(".laCircleUK").attr("opacity", 1).attr("fill", "#ccc")
    // selectAll(".laCircleUS").attr("opacity", 1).attr("fill", "#ccc")
    // selectAll(".mainChart").remove()
    setTimeout(() => {
      selectedCountry = countryOptions[0]
      document.getElementById('chartCountry').classList.add('animatedButton')
    }, 1000)
    
  }

	onMount( async () => {
		await Promise.all([
					// uk base data
					// json("https://raw.githubusercontent.com/odileeds/hexmaps/gh-pages/maps/uk-local-authority-districts-2021.hexjson"),
					// json("https://gist.githubusercontent.com/lnicoletti/2e08ca8357c8d4e4cf9bf869d890ab99/raw/a8135c1175715c9d8715be58de2ce8752fe236a4/ukUpd_tot_final_reduced.json"),
					// json("https://gist.githubusercontent.com/lnicoletti/8025f10314c9004f5c0d9e392bbf5b17/raw/f5fa0d5c86d1b8d41f9ed6624d6b23b0b1b2ec37/ukUpdGroupMonth"),
					// ////uk urb rural
					// csv("https://gist.githubusercontent.com/lnicoletti/9b5df10403fd89cc6af0ee8cd26b7925/raw/cacd508ed040a1d190401e9b16247b970181aac1/ukUrbRural_2class.csv", autoType),
					// csv("https://gist.githubusercontent.com/lnicoletti/1117ef6f526534506e96f981758a5602/raw/e0611a8a1ce43b9debb8c04465d8a67a054e6b2c/urbanRuralScotland_2class.csv", autoType),

					// // US data
					// json("https://gist.githubusercontent.com/lnicoletti/abbd577e047a3f501430869a2b6833ca/raw/2de81f38eaf396c5c2d1d75901660c9f86b10e64/usUpd_tot_fixed_reduced.json", autoType),
					// csv("https://gist.githubusercontent.com/lnicoletti/a01622da3e106830fe1c51ab96403234/raw/2b0d35c4efff50a65beb29759ae1a55eb99f73ac/usCountiesHex_tilemaps_albers.csv")

          // Already preprocessed data (drammatically faster)
          // commuter belt
          // csv("https://gist.githubusercontent.com/lnicoletti/8e926944a492e859ce8e2c3fe49b0311/raw/53d81aa1242db88f8e9f5bbf3f0049d6ab292dfc/hexes_clean_UK_le.csv", autoType),
          csv("https://gist.githubusercontent.com/lnicoletti/c40cd3aafeeb56d6429692246650fe0c/raw/c7f118a6830a865c4a356ee256d67ee36548fffe/hexes_clean_UK_luz.csv", autoType),
          csv("https://gist.githubusercontent.com/lnicoletti/206902d2e08bad7328ece8faee6921a1/raw/0945b85f8360e298bfdb277eb9169e559c6a3957/hexes_clean_US.csv", autoType),
          csv("https://gist.githubusercontent.com/lnicoletti/069a1138bf9f4c1953344f017c79c5d4/raw/0ebd8bb5cc45ca8de506adb3b020b4c01a57c9bc/countries_sm_12.csv", autoType)
          // csv("https://gist.githubusercontent.com/lnicoletti/4d96325e679440d9cb9ab13c5882af59/raw/6ae07d6919b5a42d1f882e1340b9ded21c7e67fa/countries_sm.csv", autoType)
					])
					.then((datasets)=>{
						data_UK = datasets[0]
						data_US = datasets[1]
            data_sm = datasets[2]
					})
		isLoading = !isLoading;
	})
  
$: nations = data_sm.map(d=>d.country).filter(onlyUnique)

$: console.log("small multiple", nations)
// resize view reactivity
$: view = "horizontal"
$: outerWidth = 0
$: innerWidth = 0
$: outerHeight = 0
$: innerHeight = 0

</script>

<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight />
<main>
	<!-- {#if isLoading}
	<h2>Loading...</h2>
	{:else } -->
	<!-- <h2>Data has been loaded!</h2> -->
	<body>
		<div class="content">
			<Title {vizTheme}/>
      <!-- <Legend {colors}></Legend> -->
      <!-- selectedCountry.value!==null?document.getElementById('chartCountry').classList.remove('animatedButton'):
      document.getElementById('chartCountry').classList.add('animatedButton') -->
      <div style="text-align:center" class="custom-select" id="start">
        <span class="mapCredit">START BY SELECTING A COUNTRY</span><br>
        <select id="chartCountry" class="animatedButton" bind:value={selectedCountry} on:click={() => 
          document.getElementById('chartCountry').classList.remove('animatedButton')}>
          {#each countryOptions as option}
            <option value={option}>
              {option.text}
            </option>
          {/each}
        </select>
        </div>
        <section>
        {#if selectedCountry.value==="US"}
        <Chart {colors} hexesClean = {data_US} country={"US"}/>
        {:else if selectedCountry.value==="UK"}
        <Chart {colors} hexesClean = {data_UK} country={"UK"}/>
        {/if}
      </section>
        <!-- <section>
          <Chart {colors} hexesClean = {data_US} country={"US"}/>
        </section>
        <section>
          <div class="FigSubtitle">
            Done exploring the US?  You may be wondering how the UK compares.  Let's take a look.
          </div>
        </section>
        <section>
          <Chart {colors} hexesClean = {data_UK} country={"UK"}/>
        </section> -->
        <!-- <section>
          <Chart {colors} hexesClean = {data_US} country={"US"}/>
        </section>
        <section>
          <div class="FigSubtitle">
            Done exploring the US?  You may be wondering how the UK compares.  Let's take a look.
          </div>
        </section>
        <section>
          <Chart {colors} hexesClean = {data_UK} country={"UK"}/>
        </section> -->
        <section>
          {#if selectedCountry.value!==null}
            <br>
            <br>  
            <div class="FigSubtitle">
              All in all, regardless of whether you live in the US or UK, one thing seems to be for sure: residing in an economic center of one of these nations has its benefits.<br><br>
              <i>But what about your country?  How does it compare?</i><br><br>
              We also analyzed data for {nations.length-2} other countries.  Click <button class="scrollButton" on:click={() => revertFrameOne()}>here</button> to take a dive into another country, or check out the results for all other countries below.
            </div>
            <br>
            <br>
            <br>
            <br>
            <Legend {colors} country="none"></Legend>
            <div class="smChart">
              {#each nations as nation, i}
                <CountryScatter {data_sm} {nation} {colors}/>
              {/each}
            </div>
            <Footer {vizTheme}/>
          {:else}
            <div id="emptySpacer"></div>
          {/if}
        </section>
        <!-- <section>
          <Legend {colors}></Legend>
          <div class="smChart">
            {#each nations as nation, i}
              <CountryScatter {data_sm} {nation} {colors}/>
            {/each}
          </div>
        </section> -->
		</div>
	</body>
	<!-- {/if} -->
</main>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Montserrat+Subrayada:wght@700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Montserrat+Subrayada:wght@700&family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Oswald:wght@700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Righteous&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Syncopate:wght@700&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=Anton&display=swap" rel="stylesheet">
<link href="https://fonts.googleapis.com/css2?family=DotGothic16&display=swap" rel="stylesheet">
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

  .scrollButton {
      background-color: #fafafa;
      /* width: 140px; */
      height:30px;
      color:black;
      font-weight:500;
      font-size: 12px;
      text-transform: uppercase;
      border-color: black;
      cursor: pointer;
  }

	@font-face {
    font-family: 'Roboto', sans-serif;
    src: url('https://fonts.googleapis.com/css2?family=Roboto&display=swap');
}

@font-face {
    font-family: 'Montserrat Subrayada', sans-serif;
    src: url('https://fonts.googleapis.com/css2?family=Montserrat+Subrayada:wght@700&display=swap');
}

@font-face {
    font-family:'Texturina', serif;
    src: url('https://fonts.googleapis.com/css2?family=Texturina:ital,wght@1,300&display=swap');
    font-weight: normal;
    font-style: normal;
}

body, main {
    /* background-color: #fffae7; */
    background-color: #fafafa;
}

.smChart {
  display:flex;
  flex-direction: row;
  flex-wrap: wrap;
}
.legendTitle {
  font-family:'Roboto', sans-serif;
  font-size: 10px;
  font-weight: 200;
  text-transform: None;
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

#emptySpacer {
  height: 85vh
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
    /* font-size: calc(7px + 0.4vw); */
    font-size: calc(14px - 0.15vw);
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
    /* font-size: 14px; */
    font-size: calc(18px - 0.2vw);
    font-weight: 400;
    text-transform: None;
    max-width: 400px;
    margin-left:auto;
    margin-right:auto
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
  

  .custom-select button {
    position: relative;
    font-family: 'Roboto', sans-serif;
    text-transform: uppercase;
  }

  #chartView, #chartCountry {
      background-color: #fafafa;
      width: 160px;
      height:30px;
      color:black;
      font-weight:500;
      font-size: 12px;
      text-transform: uppercase;
      border-color: black;
      margin-top: 10px;
    }

    .animatedButton {
      box-shadow: 0 0 0 0 rgba(0, 0, 0, 1);
      transform: scale(1);
      animation: pulse 2s infinite;
    }

      @keyframes pulse {
      0% {
        transform: scale(0.95);
        box-shadow: 0 0 0 0 rgba(0, 0, 0, 0.7);
      }
      
      70% {
        transform: scale(1);
        box-shadow: 0 0 0 10px rgba(0, 0, 0, 0);
      }
      
      100% {
        transform: scale(0.95);
        box-shadow: 0 0 0 0 rgba(0, 0, 0, 0);
      }
    }

  #chartView-selected {
    border-color: lightgrey
  }

  #staticTooltip {
      height: 30px;
      margin-top:10px;
      font-family: 'Roboto', sans-serif;
  }
/* 
  .annotation-group {
    font-family: 'Roboto', sans-serif;
    font-size: 15px;
  } */

  .annotation-note-label {
    font-family: 'Roboto', sans-serif;
    font-size: 10px;
  }
</style>