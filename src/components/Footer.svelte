<script>
import * as animateScroll from "svelte-scrollto";
import {selectAll} from "d3";
export let vizTheme;

function revertFrameOne() {
  animateScroll.scrollTo({element: '#start'})
  selectAll(".laCircleUK").attr("opacity", 1).attr("fill", "#ccc")
  selectAll(".laCircleUS").attr("opacity", 1).attr("fill", "#ccc")
}
</script>

<div class="content">
    <!-- <div class="FigSubtitle">
      All in all, regardless of whether you live in the US or UK, one thing seems to be for sure: residing in an economic center of one of these nations has its benefits.  But what about your country?  How does it compare?  We also analyzed data for X nations.  Click <button class="scrollButton" on:click={() => revertFrameOne()}>here</button> to take a dive into another country, or check out the results for all other countries below.
    </div>
    <br>
    <br> -->
    <!-- <details> -->
        <!-- <summary class="mapCredit" style="opacity:0.6">Learn more about how these visuals were created</summary> -->
        <hr style="height:1.5px;border-width:0;color:gray;background-color:lightgrey">
        <div class="mapCredit">METHODS</div>
        <p class="methodSection">
            This investigatigation was conducted using aggregated mobility data for 12 countries (i.e. the United Kingdom, the United States, Italy, France, Denmark, the Netherlands, Sweden, Canada and Australia) from <a href="https://www.google.com/covid19/mobility/?hl=en" target="__blank">Google's COVID-19 Community Mobility Reports</a>, household income data from the <a href="https://www.ons.gov.uk/employmentandlabourmarket/peopleinwork/earningsandworkinghours/datasets/placeofresidencebylocalauthorityashetable8" target="__blank">Office for National Statistics</a> and the <a href="https://www.census.gov/data.html" target="__blank">U.S. Census</a>, and rural population data from the United States's <a href="https://www.census.gov/programs-surveys/geography/guidance/geo-areas/urban-rural.html" target="__blank">2020 Census Urban and Rural Classification</a> and the United Kingdom's <a href="https://www.ons.gov.uk/methodology/geography/geographicalproducts/ruralurbanclassifications/2011ruralurbanclassification" target="__blank">2011 Rural Urban Classification</a>.<br><br>
        
            For each administrative area in each country, temporal data on mobility to work-places was aggregated spatially by computing the median change in mobility over the course of the pandemic (February 2020 - January 2022) relative to Google's 2019 baseline. For the United Kingdom and the United States and by using rural population data, each area with a rural population greater than 50% was classified as "Rural" and each area with a rural population lower than 50% was classified as "Urban".<br><br>

            Raw data was cleaned and processed with Javascript inside this <a href="https://observablehq.com" target="__blank">Observable</a> notebook. The visualizations themselves were created using <a href="https://d3js.org/" target="__blank">D3.js</a> and the <a href="https://svelte.dev/" target="__blank">Svelte</a> Javascript framework. U.K. level tiles were created using the <a href="https://github.com/olihawkins/d3-hexjson" target="__blank">d3-hexjson plugin</a> developed by <a href="https://olihawkins.com/" target="__blank">Oli Hawkins</a> and the Leeds <a href="https://open-innovations.org/about/" target="__blank">Open Innovations</a> team, and U.S. level tiles were created using the <a href="https://github.com/kaerosen/tilemaps">tilemaps</a> R library developed by <a href="https://github.com/kaerosen">Kaelyn Rosenberg</a>. Finally, the scrolling animation was created using the <a href="https://github.com/russellgoldenberg/scrollama">Scrollama</a> Javascript library developed by <a href="https://twitter.com/codenberg?lang=en" target=__blank>Russel Goldenberg</a>.
    </p>
    <!-- </details> -->
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

body, main {
    background-color: #fffae7;
}

.scrollButton {
      background-color: #fafafa;
      /* width: 140px; */
      height:30px;
      color:black;
      font-weight:500;
      font-size: 12px;
      text-transform: uppercase;
      border-color: black
  }

.methodSection {
    font-family:'Roboto', sans-serif;
    /* font-size: 14px; */
    font-size: calc(14px - 0.2vw);
    font-weight: 400;
    text-transform: None;
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

    /* font-family: 'Lato', sans-serif; */
    font-family: 'DotGothic16', sans-serif;
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
    font-family:'Roboto', sans-serif;
    /* font-size: 14px; */
    font-size: calc(18px - 0.2vw);
    font-weight: 400;
    text-transform: None;
    margin-top: 80px;
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
  

  .custom-select {
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