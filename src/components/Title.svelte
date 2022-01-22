<script>
	import TypeIt from "typeit";
    export let vizTheme;
    import { onMount } from "svelte";

    onMount( async () => {
        new TypeIt("#FigTitle", {
            speed: 150,
            strings: [
            `A <span class="rich${vizTheme}" style="background-color:#449d57;font-weight:900;color:#fffae7">Rich</span> vs. <span class="poor${vizTheme}" style="background-color:#c17036;font-weight:900;color:#fffae7">Poor</span> issue`,
            `An <span class="rich${vizTheme}" style="background-color:#449d57;font-weight:900;color:#fffae7">urban</span> vs. <span class="poor${vizTheme}" style="background-color:#c17036;font-weight:900;color:#fffae7">rural</span> issue`,
            ],
            breakLines: false,
            loop: true,
            pause: 1000,
        }).go();
    });

</script>

<div class="titleContainer">
    <div class="content">
        <div class="FigTitleBig">
            <span style="font-weight:900;">Work From Home</span> <br>
            <div id="FigTitle"></div>
        </div>
        <br>
        <div class="mapCredit">
            by <a href='https://www.leonardonicoletti.com/' style='text-decoration:underline; color:black' target='_blank'><b>LEONARDO NICOLETTI</b></a> and <a href='https://www.carolinecullinan.com/' style='text-decoration:underline; color:black' target='_blank'><b>CAROLINE CULLINAN</b></a>
        </div>
        <br>
        <br>
        <br>
        <div class="FigSubtitle">
            The March 11, 2020 official World Health Organization (WHO) announcement declaring the COVID-19 outbreak as a global pandemic fundamentally changed societies everywhere.  With government implemented mask mandates and social distancing practices, we no longer work nor play the same as before.  In order to comply with government regulations and craft a more pleasant “new normal”, many employers have adopted a “Work from Home” (WFH) lifestyle.  While WFH has been touted as the be-all and end-all of office productivity while simultaneously providing employees with more flexible work hours and improved overall wellbeing, not everyone has been able to benefit from the shift in work culture that has been brought upon us by the COVID-19 pandemic.  Who really benefits from this new WFH normal?  In investigating this question, we turned to data provided by Google, the UK Office for National Statistics, and the US Census regarding both mobility patterns and median household income in rural and urban areas for the United Kingdom (UK) and United States (US) over the course of the pandemic.
        </div>
        <br><br>
        <details>
            <summary class="mapCredit" style="opacity:0.6">What does this visualization show?</summary>
        
            <p class="FigSubtitle">In visualizing the relationship between mobility patterns and median household income, the UK's 374 local authorities and the US's 3,174 counties were explored on a national (i.e. US vs. UK) basis.  With a cartogram approach, each national spatial unit is represented by a <svg width=20 height=20><circle r=10 fill="lightgrey" style="transform:translate(10px, 10px)"></circle></svg>. The resulting color scheme of such circles is twofold.  Each circles' color tone is dependent on both 1) the median household income of the national spatial unit's residents and 2) the residents' percent change in mobility to workplaces relative to a pre-pandemic 2019 baseline. <br><br>

            As such, national spatial units colored in <span class="richtheNytimes">green</span> represent areas where residents are higher income and benefited from WFH policies (i.e., these residents were able to greatly reduce their mobility to the workplace).  Contrastingly, national spatial units colored in <span class="poortheNytimes">orange</span> represent areas where residents are lower income and did not benefit from WFH policies (i.e. these residents were not able to greatly reduce their mobility to the workplace).  National spatial units colored in <span class="richHightheNytimes">dark-green</span> represent areas where residents are higher income and continued to travel to the workplace during the COVID-19 pandemic.  Regardless of the wealthy status of these areas, residents did not benefit from WFH practices.  Finally, national spatial units colored in <span class="poorLowtheNytimes">light-grey</span> represent areas where residents are lower income and benefited from WFH policies.  Regardless of the poorer status of these areas, residents in these national spatial units were still able to benefit from WFH practices.<br><br>

            For both the UK and the US, this data highlights the existence of inequality.  In both national instances, the higher a resident's household income, the more likely that they are to benefit from WFH policies.  Similarly, in both the UK and the US, the lower a resident's household income, the less likely that they are to reduce their travel to the workplace and subsequently work from home.  However, while the WFH inequality seems to primarily be a rich vs. poor issue in the UK, in the US, this inequality is starkly compounded by an urban vs. rural divide.  While many wealthy US residents have been able to reduce their travel to the workplace and benefit from a WFH lifestyle during the pandemic, these individuals overwhelmingly reside in urban areas.  Contrastingly, many poorer US residents, who have been unable to reap the benefits of WFH policies, reside in rural areas.  In the US, the rich vs. poor divide seems to be somewhat one and the same with the urban vs. rural divide.  When and why did the rural US become so disenfranchised?  Have WFH policies forgotten middle-America?  How can rural American residents also experience a more pleasant “new normal” amidst this pandemic era?
            </p>
        </details>
        <br>
    </div>
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