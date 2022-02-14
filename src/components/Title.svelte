<script>
	  import TypeIt from "typeit";
    export let vizTheme;
    import { onMount } from "svelte";
    import { select } from "d3";

    let typedSubtitle;
    let typedTitle;
    let titleTyped=false;

    onMount( async () => {

      typedTitle = new TypeIt(".FigTitleBig", {
            speed: 250,
            strings: [
            `<span>Work<br>From<br>Home</span>`
            ],
            breakLines: false,
            loop: false,
            pause: 1000,
            afterComplete: function (instance) {
            instance.destroy();
          }
        })

      typedSubtitle = new TypeIt("#FigTitle", {
            speed: 150,
            strings: [
            `A <span class="rich${vizTheme}" style="background-color:#449d57;font-weight:900;color:#fafafa">Rich</span> vs. <span class="poor${vizTheme}" style="background-color:#c17036;font-weight:900;color:#fafafa">Poor</span> issue`,
            `An <span class="rich${vizTheme}" style="background-color:#449d57;font-weight:900;color:#fafafa">urban</span> vs. <span class="poor${vizTheme}" style="background-color:#c17036;font-weight:900;color:#fafafa">rural</span> issue`,
            ],
            breakLines: false,
            loop: true,
            pause: 10000,
        })

        function typeTitle() {
          return new Promise(function() {
            typedTitle.go()
          })
        }

        typeTitle()
        // .then(function() {
        //   titleTyped=true
        // })

        setTimeout(() => {typedSubtitle.go()}, 4500)

        setTimeout(() => {
          select(".mapCredit").transition().duration(1500).style("opacity", "1")
        }, 14000);
        
    });

</script>

<!-- <div > -->
    <div class="content">
        <div class="titleContainer">
          <div class="titleElements">
            <div class="FigTitleBig"></div><!--<span>Work<br>From<br>Home</span></div>-->
            <div id="FigTitle"></div>
            <div class="mapCredit">
                by <a href='https://www.leonardonicoletti.com/' style='text-decoration:underline; color:black' target='_blank'><b>LEONARDO NICOLETTI</b></a><br> and <a href='https://www.carolinecullinan.com/' style='text-decoration:underline; color:black' target='_blank'><b>CAROLINE CULLINAN</b></a>
            </div>
          </div>
        </div>
        <div class="FigSubtitle" id="opener">
          The March 11, 2020 <a class="inTextLink" href="https://www.who.int/director-general/speeches/detail/who-director-general-s-opening-remarks-at-the-media-briefing-on-covid-19---11-march-2020" target=__blank>official World Health Organization (WHO) announcement</a> declaring the COVID-19 outbreak as a global pandemic fundamentally changed societies everywhere.  With government implemented mask mandates and social distancing practices, we no longer work nor play the same as before.  In order to comply with government regulations and craft a more pleasant “new normal”, many employers have adopted a “Work from Home” (WFH) lifestyle.  While WFH has been touted as the be-all and end-all of office productivity while simultaneously providing employees with more flexible work hours and improved overall wellbeing, not everyone has been able to benefit from the shift in work culture.  <i>Who really benefits from this new WFH normal?</i>  In investigating this question, we turned to data provided by <a class="inTextLink" href="https://www.google.com/covid19/mobility/?hl=en" target=__blank>Google</a> and open government portals (see the methods section for a complete list) regarding both mobility patterns and median household income within the administrative units of 12 countries: the United Kingdom, the United States, Italy, Spain, Portugal, France, Denmark, the Netherlands, Sweden, Norway, Canada and Australia.<br><br>

          For two of those countries, the United States (US) and United Kingdom (UK), we took a deeper dive into the social and spatial disparities that have emerged over the course of the pandemic. We first asked ourselves, <i>“what constitutes each of these anglophone nations?”</i>
        </div>
        <br>
        <!-- <details>
            <summary class="mapCredit" style="opacity:0.6">What does this visualization show?</summary>
        
            <p class="methodSection">In visualizing the relationship between mobility patterns and median household income, the UK's 374 local authorities and the US's 3,174 counties were explored on a national (i.e. US vs. UK) basis.  With a cartogram approach, each national spatial unit is represented by a <svg width=20 height=20><circle r=10 fill="lightgrey" style="transform:translate(10px, 10px)"></circle></svg>. The resulting color scheme of such circles is twofold.  Each circles' color tone is dependent on both 1) the median household income of the national spatial unit's residents and 2) the residents' percent change in mobility to workplaces relative to a pre-pandemic 2019 baseline. <br><br>

            As such, national spatial units colored in <span class="richtheNytimes">green</span> represent areas where residents are higher income and benefited from WFH policies (i.e., these residents were able to greatly reduce their mobility to the workplace).  Contrastingly, national spatial units colored in <span class="poortheNytimes">orange</span> represent areas where residents are lower income and did not benefit from WFH policies (i.e. these residents were not able to greatly reduce their mobility to the workplace).  National spatial units colored in <span class="richHightheNytimes">dark-green</span> represent areas where residents are higher income and continued to travel to the workplace during the COVID-19 pandemic.  Regardless of the wealthy status of these areas, residents did not benefit from WFH practices.  Finally, national spatial units colored in <span class="poorLowtheNytimes">light-grey</span> represent areas where residents are lower income and benefited from WFH policies.  Regardless of the poorer status of these areas, residents in these national spatial units were still able to benefit from WFH practices.<br><br>

            For both the UK and the US, this data highlights the existence of inequality.  In both national instances, the higher a resident's household income, the more likely that they are to benefit from WFH policies.  Similarly, in both the UK and the US, the lower a resident's household income, the less likely that they are to reduce their travel to the workplace and subsequently work from home.  However, while the WFH inequality seems to primarily be a rich vs. poor issue in the UK, in the US, this inequality is starkly compounded by an urban vs. rural divide.  While many wealthy US residents have been able to reduce their travel to the workplace and benefit from a WFH lifestyle during the pandemic, these individuals overwhelmingly reside in urban areas.  Contrastingly, many poorer US residents, who have been unable to reap the benefits of WFH policies, reside in rural areas.  In the US, the rich vs. poor divide seems to be somewhat one and the same with the urban vs. rural divide.  When and why did the rural US become so disenfranchised?  Have WFH policies forgotten middle-America?  How can rural American residents also experience a more pleasant “new normal” amidst this pandemic era?
            </p>
        </details> -->
        <br>
    </div>
<!-- </div> -->

<style>
    @font-face {
    font-family: 'Roboto', sans-serif;
    src: url('https://fonts.googleapis.com/css2?family=Roboto&display=swap');
}

@font-face {
    font-family:'Texturina', serif;
    src: url('https://fonts.googleapis.com/css2?family=Texturina:ital,wght@1,300&display=swap');
    font-weight: normal;
    font-style: normal;
}

/* body, main {
    background-color: #fafafa;
} */


.inTextLink {
  color:rgb(75, 75, 75);
  text-decoration: underline;
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



.content {
    max-width: 840px; /* Can be in percentage also. */
    height: auto;
    margin: 0 auto;
    padding: 10px;
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

    /* font-family: 'Roboto', sans-serif; */
    font-family: 'DotGothic16', sans-serif;
    font-weight: 700;
    font-size: calc(7px + 0.4vw);
    text-align: center;
    line-height: 1.6em;
    margin-top: 1.5em;
    letter-spacing: 0.1em;
    opacity: 0;
    /* visibility:hidden; */
    /* transform: translate(10px,93px) */
    /* transform: translate(10px, calc(35px + 3.5vw)) */
    /* font-size: 12px; */
}

.FigTitleBig {
    /* font-family:'Roboto', sans-serif; */
    /* font-family: 'Montserrat Subrayada', sans-serif; */
    /* font-family: 'Righteous', cursive; */
    /* font-family: 'Oswald', sans-serif; */
    /* font-family: 'Syncopate', sans-serif; */
    /* font-family: 'Anton', sans-serif; */
    font-family: 'DotGothic16', sans-serif;
    font-size: calc(60px + 8vw);
    font-weight: 900;
    /* line-height: 0.8em; */
    line-height: 1em;
    letter-spacing: 0.2em;
    /* min-height: 40vh; */
    /* height:130px; */
    text-transform: uppercase;
    text-align: center;
    transform: translate(15px, 0);
  }

  .titleElements {
    display:table-cell;
    vertical-align:middle
  }

  .titleContainer {
    height: 92vh;
    line-height: 100vh;
    width:100%;
    /* height:130px; */
    margin-bottom: 20px;
    /* border: 7px solid #449d57; */
    display:table;
    position: relative;
    /* align-items: center;
    display: flex;
    flex-direction: column; */
  }

  /* .titleContainer:before {
  content: " ";
  position: absolute;
  z-index: 1;
  top: 5px;
  left: 5px;
  right: 5px;
  bottom: 5px;
  border: 7px solid #c17036;
} */

  #FigTitle {
    /* font-family:'Roboto', sans-serif; */
    font-family: 'DotGothic16', sans-serif;
    font-size: calc(10px + 1.7vw);
    font-weight: 500;
    height:53px;
    vertical-align:text-bottom;
    /* text-transform: uppercase; */
    text-transform: lowercase;
    text-align: center;
    line-height: 1em;
    margin-top: 0.8em;
  }
  
  .FigSubtitle {
    font-family:'Roboto', sans-serif;
    /* font-size: 14px; */
    font-size: calc(18px - 0.2vw);
    font-weight: 400;
    text-transform: None;
    margin-top: 80px;
  }

  .methodSection {
    font-family:'Roboto', sans-serif;
    /* font-size: 14px; */
    font-size: calc(14px - 0.2vw);
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