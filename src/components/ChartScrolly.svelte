<script>
    import * as d3 from "d3";
    // import {fade, blur, fly, slide, scale, draw} from "svelte/transition";
    import { tweened } from "svelte/motion";
    import * as easings from 'svelte/easing';
    import {renderHexJSON} from "d3-hexjson"
	  import {annotationLabel, annotation, annotationCalloutCircle} from "d3-svg-annotation"
    import {onlyUnique} from "../utils.js"
    import { onMount } from "svelte";
    import Scroll from "./Scrolly.svelte";
    import Svelecte from 'svelecte';
    import UnivariateLegend from "./UnivariateLegend.svelte";
    import Legend from "./Legend.svelte";

    // view reactivity
    $: view = "horizontal"  
    $: outerWidth = 0
    $: innerWidth = 0
    $: outerHeight = 0
    $: innerHeight = 0

    // export let vizTheme;
    export let colors;
    export let hexesClean;
    export let country;

    console.log(hexesClean)

  //   let countryOptions = [
	// 	{ id: 1, value: `UK`, text: `United Kingdom`},
	// 	{ id: 2, value: `US`, text: `United States`}
	// ];

    let viewOptions = [
		{ id: 1, value: `map`, text: `Map`},
		{ id: 2, value: `chart`, text: `Scatter-plot`},
    { id: 3, value: `bars`, text: `Bars`},
    { id: 4, value: `barsUrban`, text: `Bars Urban`}
	];

	  // let selectedCountry = countryOptions[0];
    $: selectedView = viewOptions[0];
    
    let svg, hexmap, circles, annot, stateAnnot;
    let scale = 1
    let width = 800
    let height = 600
    // responsive margins
    let margin = ({ top: 0.05*height, right: 0.04*width, bottom: 0.1*height, left: 0.1*width})

    let radiusUK = width/80
    let radiusUS = width/220//9.8
    let radiusHoverUK = radiusUK*2
    let radiusHoverUS = radiusUS*2
    $: boundWidth = 4
    $: fontSize = 6.5
    let yVar = "mobilityWork"
    // const xVar = "mobilityRes"
    let xVar = "income"
    $: xVarLabel = "Median Household Income"
    // const xVarLabel = "Average Change in Work From Home Households Since Feb. 2020"
    $: yVarLabel = "Average Change in People Going to Workplaces Since Feb. 2020 (%)"


    // color settings
    let categoriesX = country==="UK"?[colors[6], colors[2], colors[4], colors[7], colors[5], colors[3], colors[1], colors[0], colors[8]]:
                                     [colors[2], colors[6], colors[7], colors[4], colors[3], colors[5], colors[8], colors[1], colors[0]] ;
    let categoryLabels = country==="UK"?["Low Income, High Travel", "High Income, Low Travel", "Mid. Income, Mid. Travel", "Mid. Income, High Travel", 
                                         "High Income, Mid. Travel", "Low Income, Mid. Travel", "Mid. Income, Low Travel", "Low Income, Low Travel", "High Income, High Travel"]:
                                        ["High Income, Low Travel", "Low Income, High Travel", "Mid. Income, High Travel", "Mid. Income, Mid. Travel", 
                                         "Low Income, Mid. Travel", "High Income, Mid. Travel", "High Income, High Travel", "Mid. Income, Low Travel", "Low Income, Low Travel"];

    let sortOrder = country==="UK"?[categoriesX[1], categoriesX[0], categoriesX[4], categoriesX[2], categoriesX[7], categoriesX[5], categoriesX[6], categoriesX[3], categoriesX[8]]:
                                   [categoriesX[0], categoriesX[1], categoriesX[5], categoriesX[3], categoriesX[8], categoriesX[4], categoriesX[7], categoriesX[2], categoriesX[6]];

    // bivar settings
    let dataBivar = country==="UK"?Object.assign(new Map(hexesClean.map(d=>[d.key, [d[yVar], d[xVar]]])), {title: ["Travel to Work", "Med. Income"]}):
                                   Object.assign(new Map(hexesClean.map(d=>[d.GEOID, [d[yVar], d[xVar]]])), {title: ["Travel to Work", "Med. Income"]})

    console.log("bivar", dataBivar)
                                   
    let n = Math.floor(Math.sqrt(colors.length))
    let yBivar = d3.scaleQuantile(Array.from(dataBivar.values(), d => d[1]), d3.range(n))
    let xBivar = d3.scaleQuantile(Array.from(dataBivar.values(), d => d[0]), d3.range(n))
    let labels = ["low", "medium", "high"]
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
    // legend data
    let legendData = d3.cross(d3.range(n), d3.range(n))
    const k = 63/n


        // console.log("urb rural", ruralData)
        let urbCategoriesX = ["Urban", "Rural"]//ruralData.sort((a,b)=>b.data.length-a.data.length).map(d=>d.urbCategory)   
        let urbCategoryLabels = ["Urban", "Rural"]
        let leCatX = ["London Commuter Belt", "Not London"]
        let leCatLabels = ["London's Larger Urban Zone", "Elsewhere in the U.K."]

        var featureCollection = country==="US"?
        { type:"FeatureCollection", features: hexesClean.map(function(d) {
                    return {     
                      "type": "Feature",
                      "geometry": {
                         "type": "Point",
                         "coordinates": [d.x, d.y]
                      },
                      "properties": { "name":d.county }
                    }
                }) }: {}

        $: projection = country==="US"?
                        d3.geoAlbersUsa()
                          .fitSize([width-margin.right/4-margin.left, height-margin.bottom/4], featureCollection):null;
                
        $: normScaleXInc = d3.scaleLinear()
                        .domain(d3.extent(hexesClean, d => d.income))
                        .range([margin.left, width - margin.right])
      
        $: normScaleYMob = d3.scaleLinear()
                        .domain(d3.extent(hexesClean, d => d.mobilityWork))
                        .range([height - margin.bottom, margin.top])

        $: normScaleXhex = d3.scaleLinear()
                        .domain(d3.extent(hexesClean, d => d.x))
                        .range([margin.left, width - margin.right])
      
        $: normScaleYhex = d3.scaleLinear()
                        .domain(d3.extent(hexesClean, d => d.y))
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatRow = d3.scaleLinear()
                        .domain(country==="UK"?[0, 76]:[0, 520])
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatX = d3.scaleBand()
                        .domain(categoriesX)
                        .range([margin.left, width - margin.right])


        $: normScaleUrbRow = d3.scaleLinear()
                        .domain(country==="UK"?[0, 255]:[0, 1550])
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatXUrb = d3.scaleBand()
                        .domain(urbCategoryLabels)
                        .range([margin.left*1.5, width - margin.right])

        $: normScaleLeRow = d3.scaleLinear()
                        .domain(d3.extent(hexesClean, d => d.leRow))
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatXLe = d3.scaleBand()
                        .domain(leCatX)
                        .range([margin.left*1.5, width - margin.right])

        // color scales
        $: incomeColor = d3.scaleQuantize()
                        .domain(country==="UK"?d3.extent(hexesClean, d => d[xVar]):[d3.min(hexesClean, d => d[xVar]), 80000])
                        .range(d3.schemeGreens[9])

        $: mobilityColor = d3.scaleQuantize()
                        .domain(country==="UK"?d3.extent(hexesClean, d => d[yVar]):[-40, -5])
                        .range(d3.schemeOranges[9])
                        


        // ticks
        let xTicksIncome;
        $: if (country==="UK"){
          xTicksIncome = innerWidth > 500 ?
          d3.range(15000, 70000, 5000) :
          d3.range(15000, 70000, 10000);
        } 
        
        $: if (country==="US") {
          xTicksIncome = innerWidth > 500 ?
          d3.range(20000, 140000, 10000) :
          d3.range(20000, 140000, 20000);
          
        }

        let yTicksMob;
        $: if (country==="UK"){
          yTicksMob = innerHeight > 500 ?
          d3.range(-60, -10, 5) :
          d3.range(-60, -10, 15) ;
        } 
        
        $: if (country==="US") {
          yTicksMob = innerHeight > 500 ?
          d3.range(-60, 5, 5) :
          d3.range(-60, 5, 15) ;
        }

        console.log("clean", hexesClean)    
        
        let firstStep;
        let comingFromMap;

        // % of green areas in london commuter belt
        let pctLe = country==="UK"?hexesClean.filter(d=>d.leCat==="London Commuter Belt"&&d.category==="#47a45b").length/hexesClean.filter(d=>d.category==="#47a45b").length:null

        // console.log("%percent green in commuter belt", pctLe)

        $: stateCentroids = country==="US"?d3.rollups(
          hexesClean.map(d=>{return{GEOID:d.GEOID, abbrv:d.abbrv.replaceAll(" ", ""), x:d.x, y:+d.y}}), 
            v=>[d3.mean(v, d=>d.x), d3.mean(v, d=>d.y)], 
              d=>d.abbrv).map(d=>{return{abbrv:d[0], x:d[1][0], y:d[1][1]}})
              :null
        
        $: console.log("centroids", stateCentroids)

        onMount(() => {

          // console.log("innerwidth", innerWidth)
         
          hexmap = d3.select(svg)
              .selectAll("g")
              .data(hexesClean)
              .join("g")
              .attr("id", d=> country==="UK"?d.key + "Group":d.fullName.replaceAll(",", "").replaceAll(" ", "")+"Group")
              // .attr("transform", `translate(${margin.left},0)`)

          circles = hexmap
                .append("circle")
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("r", country==="UK"?radiusUK:radiusUS)//innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius)
                .attr("stroke", "#fafafa")
                .attr("stroke-width", "0.5")
                // .attr("fill", d => d.category)
                .attr("fill", "#ccc")
                .attr("class", "laCircle"+country)
                .attr("id", d=> country==="UK"?d.key:d.fullName.replaceAll(",", "").replaceAll(" ", ""))
                .attr("cursor", "pointer")
                // .style("z-index", 100)
                .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                                   country==="US"&&selectedView.value==="map"?`translate(${margin.left/2},0)`:`translate(0,0)`)
                .on("mouseover", (event,d)=>console.log(d))//[normScaleXInc(d[xVar]), normScaleYMob(d[yVar])], event.clientX)
                // .on("mouseover", (event, d)=>console.log(normScaleYhex(+d.y)))


          annot = hexmap
                .append("text")
                .attr("x", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("y", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("text-anchor", "middle")
                .attr('class', 'LStext'+country)
                .attr("id", d=> country==="UK"?"label"+d.key:"label"+d.fullName.replaceAll(",", "").replaceAll(" ", ""))
                .attr('font-size', fontSize)
                .attr('fill', 'rgb(255,255,255)')
                .attr("z-index", 10)
                .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .text(d=>country==="UK"?d.n.slice(0,3):"")
                .on("mouseover", (event, d)=>console.log(innerWidth))
                .attr("cursor", "pointer")


          if (country==="US") {

            stateAnnot = 
              d3.select(svg).append("g")
                
              stateAnnot.selectAll(".stateAbbrv")
                .data(stateCentroids)
                .join("text")
                .attr('class', 'stateAbbrv')
                .attr("x", d=>d.abbrv==="FL"?projection([d.x, d.y])[0]+30:projection([d.x, d.y])[0])
                .attr("y", d=>projection([d.x, d.y])[1])
                .attr("text-anchor", "middle")
                // .attr('fill', 'rgb(255,255,255)')
                .attr('fill', 'black')
                .attr("font-weight", "bold")
                .attr('font-size', 12)
                .text(d=>d.abbrv)
                .attr("transform",`translate(${margin.left/2},0)`)

          }

              
          firstStep = true
          comingFromMap = true
          // annot = true

                // resize()
        })

        $: console.log("normalized", selectedView.value)

        $: if (country==="UK" && hexmap && circles && annot && currentStep===0 && firstStep===false){

              d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", "#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)



              firstStep=true
              comingFromMap = true
              valueUK = null

            } else if (country==="UK" && hexmap && circles && annot && currentStep===1) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity", 1)
                .attr("fill", d => d.category!=="#ccc"?incomeColor(d[xVar]):"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUK = null

            } else if (country==="UK" && hexmap && circles && annot && currentStep===2) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity", 1)
                .attr("fill", d => d.category!=="#ccc"?mobilityColor(d[yVar]):"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUK = null

            } else if (country==="UK" && hexmap && circles && annot && currentStep===3) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity", 1)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUK = null

              } else if (country==="UK" && hexmap && circles && annot && currentStep===4) { 

                // highlight islington
                valueUK = "E09000019"
                valueUS = null

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity", 1)
                // .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                // } else if (hexmap && circles && annot && selectedView.value==="bars") {
              } else if (country==="UK" && hexmap && circles && annot && currentStep===5) { 

              // d3.select(".chart").style("position", "sticky")
              d3.selectAll(".annotation-group").remove()

                circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annot.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.category!=="#ccc")
                .transition()
                  .delay((d, i) => {
                    return i * Math.random() * 1.5;
                    })
                  .duration(800)
                .ease(d3.easeLinear)
                .attr("cx", d => normScaleXInc(d[xVar]))
                .attr("cy", d=> normScaleYMob(d[yVar]))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                          country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                        country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot.filter(d=>d.category!=="#ccc")
                .transition()
                  .delay((d, i) => {
                    return i * Math.random() * 1.5;
                    })
                  .duration(800)
                .ease(d3.easeLinear)
                .attr("x", d => normScaleXInc(d[xVar]))
                .attr("y", d=> normScaleYMob(d[yVar]))
                .attr("opacity", d=>d.income!==null?1:0)
                // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUK = null
                // } else if (hexmap && circles && annot && selectedView.value==="map") {
        
              } else if (country==="UK" && hexmap && circles && annot && currentStep===6) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight london
                  valueUK = "E09000001"
                  valueUS = null

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.key===valueUK)[0].n, 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                0, 
                                -250, 
                                -10)

              } else if (country==="UK" && hexmap && circles && annot && currentStep===7) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight Blaneau gwent
                  valueUK = "W06000019"
                  valueUS = null

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.key===valueUK)[0].n, 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                0, 
                                0, 
                                320)

              } else if (country==="UK" && hexmap && circles && annot && currentStep===8) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight glasgow
                  valueUK = "S12000049"
                  valueUS = null

                  // circles.filter(d=>d.income===null).attr("opacity", 0)
                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.key===valueUK)[0].n, 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                0, 
                                60, 
                                0)

              } else if (country==="UK" && hexmap && circles && annot && currentStep===9) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight East Hampshire
                  valueUK = "E07000085"
                  valueUS = null

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annot.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.category!=="#ccc")
                .transition()
                  .delay((d, i) => {
                    return i * Math.random() * 1.5;
                    })
                  .duration(800)
                .ease(d3.easeLinear)
                .attr("cx", d => normScaleXInc(d[xVar]))
                .attr("cy", d=> normScaleYMob(d[yVar]))
                // .attr("opacity", d=>d.income!==null?1:0)
                // .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                          country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                        country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot.filter(d=>d.category!=="#ccc")
                .transition()
                  .delay((d, i) => {
                    return i * Math.random() * 1.5;
                    })
                  .duration(800)
                .ease(d3.easeLinear)
                .attr("x", d => normScaleXInc(d[xVar]))
                .attr("y", d=> normScaleYMob(d[yVar]))
                // .attr("opacity", d=>d.income!==null?1:0)
                // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false

                annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.key===valueUK)[0].n, 
                                hexesClean.filter(d=>d.key===valueUK)[0][yVar], 
                                hexesClean.filter(d=>d.key===valueUK)[0][xVar], 
                                0, 
                                0, 
                                50)

              } else if (country==="UK" && hexmap && circles && annot && currentStep===10) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annot.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d => normScaleCatX(d.category)+d.paddingCatX)
                .attr("cy", d=> normScaleCatRow(d.catRow+d.paddingCatY))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)
                
                annot.filter(d=>d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .ease(d3.easeLinear)
                .attr("x", d => normScaleCatX(d.category)+d.paddingCatX)
                .attr("y", d=> normScaleCatRow(d.catRow+d.paddingCatY))
                .attr("opacity", d=>d.income!==null?1:0)
                // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUK = null

                if (country === "US") {
                  annotateBars(normScaleCatX, 
                  normScaleCatRow, 
                  categoriesX[0], 
                  460, 
                  svg, 
                  "30% of localities are either low income, high travel or high income, low travel", 
                  normScaleCatX.bandwidth()*1.75,
                  15,
                  170)
                } else if (country==="UK") {
                  annotateBars(normScaleCatX, 
                  normScaleCatRow, 
                  categoriesX[0], 
                  78, 
                  svg, 
                  "50% of localities are either low income, high travel or high income, low travel", 
                  normScaleCatX.bandwidth()*1.3,
                  30,
                  162)
                }

              // } else if (hexmap && circles && annot && selectedView.value==="barsUrban") {

              } else if (country==="UK" && hexmap && circles && annot && currentStep===11) {

                d3.selectAll(".annotation-group").remove()

                circles.filter(d=>d.urbCategory===null||d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)
          
                annot.filter(d=>d.urbCategory===null||d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrbX)
                .attr("cy", d=> normScaleUrbRow(d.urbRow+d.paddingUrbY))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d=>[categoriesX[1], categoriesX[0]].includes(d.category)?d.category:"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                         country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrbX)
                .attr("y", d=> normScaleUrbRow(d.urbRow+d.paddingUrbY))
                .attr("opacity", d=>d.income!==null?1:0)
                // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUK = null

                if (country==="US") {
                  annotateBars(normScaleCatXUrb, 
                    normScaleUrbRow, 
                    "Rural", 
                    450, 
                    svg, 
                    "Unlike the U.K., in the U.S. the majority of low income, high travel areas are rural...", 
                    -15,
                    -15,
                    -60)

                  annotateBars(normScaleCatXUrb, 
                    normScaleUrbRow, 
                    "Urban", 
                    363, 
                    svg, 
                    "...while the majority of high income, low travel localities are urban", 
                    normScaleCatXUrb.bandwidth()*0.615,
                    15,
                    60)
                } else if (country==="UK") {
                  annotateBars(normScaleCatXUrb, 
                    normScaleUrbRow, 
                    "Urban", 
                    78, 
                    svg, 
                    "The majority of low income, high travel and high income, low travel localities are located in urban areas", 
                    normScaleCatXUrb.bandwidth()*0.6,
                    -40,
                    60)

                  // make chart not sticky
                  // d3.select(".chart").style("position", "static")
                }

            } else if (country==="UK" && hexmap && circles && annot && currentStep===12) {

                d3.selectAll(".annotation-group").remove()

                circles.filter(d=>d.leCat===null||d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)
          
                annot.filter(d=>d.leCat===null||d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.leCat!==null&&d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d => normScaleCatXLe(d.leCat)+d.paddingLeX)
                .attr("cy", d=> normScaleLeRow(d.leRow+d.paddingLeY))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d=>[categoriesX[1], categoriesX[0]].includes(d.category)?d.category:"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                         country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot.filter(d=>d.leCat!==null&&d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d => normScaleCatXLe(d.leCat)+d.paddingLeX)
                .attr("y", d=> normScaleLeRow(d.leRow+d.paddingLeY))
                .attr("opacity", d=>d.income!==null?1:0)
                // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUK = null


                if (country==="UK") {
                  annotateBars(normScaleCatXLe, 
                  normScaleLeRow, 
                    "London Commuter Belt", 
                    68, 
                    svg, 
                    (pctLe*100).toFixed()+"% of all high income, low travel localities are located in London's Larger Urban Zone", 
                    normScaleCatXLe.bandwidth()*0.52,
                    -40,
                    60)
                }

            } else if (country==="UK" && hexmap && circles && annot && currentStep===13) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", d=>d.leCat!=="Not London"?1:0.3)
                .attr("fill", d=>[categoriesX[1], categoriesX[0]].includes(d.category)?d.category:"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                annotateBars(null, 
                    null, 
                    413.6, 
                    500.9, 
                    svg, 
                    "London's Larger Urban Zone", 
                    200,
                    -100,
                    110)

                firstStep=false
                comingFromMap = true
                valueUK = null

          } else if (country==="UK" && hexmap && circles && annot && currentStep===14) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity", 1)
                .attr("fill", d=>d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", country==="UK"&&[0,1,2,3,4,13,14].includes(currentStep)?`translate(${margin.left*2},0)`:
                                      country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

                annot
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13,14].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                // valueUK = null

          }


        
        $: if (country==="US" && hexmap && circles && annot && currentStep===0 && firstStep===false){

              d3.selectAll(".annotation-group").remove()
              d3.selectAll(".stateAbbrv").attr('visibility', 'visible').attr('fill', 'black')

                circles
                .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                .duration(200)
                .ease(d3.easeLinear)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", "#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)

                // annot
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d=>d.x)
                // .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                // .attr("transform",currentStep<6?`translate(${margin.left*2},0)`:`translate(0,0)`)

              firstStep=true
              comingFromMap = true
              valueUS = null

            } else if (country==="US" && hexmap && circles && annot && currentStep===1) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()
                d3.selectAll(".stateAbbrv").attr('visibility', 'hidden')//.attr('fill', 'rgb(255,255,255)')

                circles
                .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                .duration(200)
                .ease(d3.easeLinear)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", d => d.category!=="#ccc"?incomeColor(d[xVar]):"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)

                // annot
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d=>d.x)
                // .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                // .attr("transform",currentStep<6?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUS = null

            } else if (country==="US" && hexmap && circles && annot && currentStep===2) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                .duration(200)
                .ease(d3.easeLinear)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", d => d.category!=="#ccc"?mobilityColor(d[yVar]):"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)

                // annot
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d=>d.x)
                // .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                // .attr("transform",currentStep<6?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUS = null

            } else if (country==="US" && hexmap && circles && annot && currentStep===3) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles
                .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                .duration(200)
                .ease(d3.easeLinear)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)

                // annot
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d=>d.x)
                // .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                // .attr("transform",currentStep<6?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUS = null

              } else if (country==="US" && hexmap && circles && annot && currentStep===4) { 

                // highlight islington
                d3.selectAll(".annotation-group").remove()
                // d3.selectAll(".stateAbbrv").attr('visibility', 'visible')

                valueUK = null
                valueUS = "SpokaneCountyWashington"

                circles
                .transition()
                .duration(100)
                .ease(d3.easeLinear)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity", 1)
                // .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,4,12].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)


                // } else if (hexmap && circles && annot && selectedView.value==="bars") {
              } else if (country==="US" && hexmap && circles && annot && currentStep===5) { 

              // d3.select(".chart").style("position", "sticky")
              d3.selectAll(".annotation-group").remove()
              // d3.selectAll(".stateAbbrv").attr('visibility', 'hidden')

                circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annot.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.category!=="#ccc")
                .transition()
                  .delay((d, i) => {
                    return i * Math.random();
                    })
                  .duration(500)
                .ease(d3.easeLinear)
                .attr("cx", d => normScaleXInc(d[xVar]))
                .attr("cy", d=> normScaleYMob(d[yVar]))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                          country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

                // annot.filter(d=>d.category!=="#ccc")
                // .transition()
                //   .delay((d, i) => {
                //     return i * Math.random() * 1.5;
                //     })
                //   .duration(800)
                // .ease(d3.easeLinear)
                // .attr("x", d => normScaleXInc(d[xVar]))
                // .attr("y", d=> normScaleYMob(d[yVar]))
                // .attr("opacity", d=>d.income!==null?1:0)
                // // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                // .attr("transform",currentStep===0?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUS = null
                // } else if (hexmap && circles && annot && selectedView.value==="map") {
        
              } else if (country==="US" && hexmap && circles && annot && currentStep===6) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight islington
                  valueUK = null
                  valueUS = "SanFranciscoCountyCalifornia"
                  GEOID = 6075

                  
                annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0].fullName, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar],
                                0, 
                                -250, 
                                10)


                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

              } else if (country==="US" && hexmap && circles && annot && currentStep===7) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight islington
                  valueUK = null
                  valueUS = "DouglasCountyMissouri"
                  GEOID = 29067

                  annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0].fullName, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar],
                                0, 
                                0, 
                                300)

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

              } else if (country==="US" && hexmap && circles && annot && currentStep===8) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight islington
                  valueUK = null
                  valueUS = "LafayetteCountyArkansas"
                  GEOID = 5073

                  annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0].fullName, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar], 
                                0, 
                                -20, 
                                0)

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

               } else if (country==="US" && hexmap && circles && annot && currentStep===9) { 

                d3.selectAll(".annotation-group").remove()
                  // highlight islington
                  valueUK = null
                  valueUS = "NantucketCountyMassachusetts"
                  GEOID = 25019

                  annotateScatter(normScaleXInc, 
                                normScaleYMob, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                svg, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0].fullName, 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][yVar], 
                                hexesClean.filter(d=>d.GEOID===GEOID)[0][xVar], 
                                0, 
                                30, 
                                0)

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annot.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.category!=="#ccc")
                .transition()
                  .delay((d, i) => {
                    return i * Math.random();
                    })
                  .duration(500)
                .ease(d3.easeLinear)
                .attr("cx", d => normScaleXInc(d[xVar]))
                .attr("cy", d=> normScaleYMob(d[yVar]))
                // .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                          country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

              } else if (country==="US" && hexmap && circles && annot && currentStep===10) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

                circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

                annot.filter(d=>d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random();
                  })
                .duration(700)
                .attr("cx", d => normScaleCatX(d.category)+d.paddingCatX)
                .attr("cy", d=> normScaleCatRow(d.catRow+d.paddingCatY))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)
                
                // annot.filter(d=>d.category!=="#ccc")
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .ease(d3.easeLinear)
                // .attr("x", d => normScaleCatX(d.category)+d.paddingCatX)
                // .attr("y", d=> normScaleCatRow(d.catRow+d.paddingCatY))
                // .attr("opacity", d=>d.income!==null?1:0)
                // // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                // .attr("transform",currentStep===0?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUS = null

                if (country === "US") {
                  annotateBars(normScaleCatX, 
                  normScaleCatRow, 
                  categoriesX[0], 
                  460, 
                  svg, 
                  "30% of localities are either low income, high travel or high income, low travel", 
                  normScaleCatX.bandwidth()*1.75,
                  15,
                  170)
                } else if (country==="UK") {
                  annotateBars(normScaleCatX, 
                  normScaleCatRow, 
                  categoriesX[0], 
                  78, 
                  svg, 
                  "50% of localities are either low income, high travel or high income, low travel", 
                  normScaleCatX.bandwidth()*1.3,
                  30,
                  162)
                }

              // } else if (hexmap && circles && annot && selectedView.value==="barsUrban") {

              } else if (country==="US" && hexmap && circles && annot && currentStep===11) {

                d3.selectAll(".annotation-group").remove()
                // d3.selectAll(".stateAbbrv").attr('visibility', 'hidden')

                circles.filter(d=>d.urbCategory===null||d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)
          
                annot.filter(d=>d.urbCategory===null||d.category==="#ccc")
                .transition()
                .duration(750)
                .ease(d3.easeLinear)
                // .attr("x", 200)
                // .attr("y", 1000)
                .attr("opacity", 0)
                
                circles.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
                .transition()
                .delay((d, i) => {
                  return i * Math.random();
                  })
                .duration(700)
                .attr("cx", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrbX)
                .attr("cy", d=> normScaleUrbRow(d.urbRow+d.paddingUrbY))
                .attr("opacity", d=>d.income!==null?1:0)
                .attr("fill", d=>[categoriesX[1], categoriesX[0]].includes(d.category)?d.category:"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                         country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

                // annot.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrbX)
                // .attr("y", d=> normScaleUrbRow(d.urbRow+d.paddingUrbY))
                // .attr("opacity", d=>d.income!==null?1:0)
                // // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
                // .attr("transform",currentStep===0?`translate(${-margin.left},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = false
                valueUS = null

                if (country==="US") {
                  annotateBars(normScaleCatXUrb, 
                    normScaleUrbRow, 
                    "Rural", 
                    450, 
                    svg, 
                    "Unlike the U.K., in the U.S. the majority of low income, high travel areas are rural...", 
                    -10,
                    -15,
                    -48)

                  annotateBars(normScaleCatXUrb, 
                    normScaleUrbRow, 
                    "Urban", 
                    363, 
                    svg, 
                    "...while the majority of high income, low travel localities are urban", 
                    normScaleCatXUrb.bandwidth()*0.65,
                    15,
                    60)
                } else if (country==="UK") {
                  annotateBars(normScaleCatXUrb, 
                    normScaleUrbRow, 
                    "Urban", 
                    78, 
                    svg, 
                    "The majority of low income, high travel and high income, low travel localities are located in urban areas", 
                    normScaleCatXUrb.bandwidth()*0.6,
                    -40,
                    60)

                  // make chart not sticky
                  // d3.select(".chart").style("position", "static")
                }

            } else if (country==="US" && hexmap && circles && annot && currentStep===12) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()
                // d3.selectAll(".stateAbbrv").attr('visibility', 'visible')//.attr('fill', 'black')

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random();
                  })
                .duration(700)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", d=>d.urbCategory==="Urban"?1:0.3)
                .attr("fill", d=>[categoriesX[1], categoriesX[0]].includes(d.category)?d.category:"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)

                // annot
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d=>d.x)
                // .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                // .attr("transform",[6, 12].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                valueUS = null

          } else if (country==="US" && hexmap && circles && annot && currentStep===13) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()
                // d3.selectAll(".stateAbbrv").attr('fill', 'rgb(255,255,255)')

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random();
                  })
                .duration(700)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                // .attr("opacity",1)
                .attr("fill", d=>d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12,13].includes(currentStep)?`translate(${margin.left/2},0)`:`translate(0,0)`)

                // annot
                // .transition()
                // .delay((d, i) => {
                //   return i * Math.random() * 1.5;
                //   })
                // .duration(800)
                // .attr("x", d=>d.x)
                // .attr("y", d=>d.y)
                // .attr("opacity", 1)
                // // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                // .attr("transform",[6, 12].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                // valueUS = null
          }
  
        
        function resize() {
          // ({ width, height } = svg.getBoundingClientRect());

          scale = innerWidth/800
          console.log("scale", scale)
          // console.log(width, height)
          // console.log(margin)
          // console.log(normScaleXInc.range())

          // normScaleXInc.range([margin.left, width - margin.right])

          // return [width, height]
        }

        function annotateBars(x, y, category, row, svg, text, offset, yOffset, xOffset) {
            // console.log(y)
            const type = annotationLabel

            const annotations = [{
            note: {
                label: x===null?"":text,
                bgPadding: 10,
                title: x!==null?"":text
            },
            connector: {
              end: "dot",
              // type: "curve",
            },
            //can use x, y directly instead of data
            data: {category: category, row: row},
            className: "show-bg",
            dy: yOffset,
            dx: xOffset
            }]


            const makeAnnotations = annotation()
            // .editMode(true)
            //also can set and override in the note.padding property
            //of the annotation object
            .notePadding(15)
            .type(type)
            //accessors & accessorsInverse not needed
            //if using x, y in annotations JSON
            .accessors({
                x: d => x!==null?x(d.category)+offset:d.category+offset,
                y: d => y!==null?y(d.row):d.row
            })
            // .accessorsInverse({
            //     category: d => x.invert(d.x),
            //     row: d => y.invert(d.y)
            // })
            .annotations(annotations)

            // console.log(d3.annotation())

            // const annotationData = [{ category: category, row: row, text: text}]

            // // const annotations = 
            // d3.select("#chart")//.select("svg")
            // // .data(annotationData)
            d3.select(svg).append("g")
            .attr("class", "annotation-group")
            .style("font-size", "15px")
            // .append("text")
            // .attr("x", d=>x(d.category))
            // .attr("y", d=>y(d.row))
            // .text(text)
            .call(makeAnnotations)
            // .transition().duration(1000)
            // .attr("transform", `translate(${x.bandwidth()}, 0)`)

            d3.selectAll(".annotation-group").attr("opacity", 0).transition().duration(1000).attr("opacity", 0.6)
            // d3.selectAll(".annotation-note-label")//.attr("fill", "#445312")
            // d3.selectAll(".annotation-connector path").attr("stroke", "#445312").attr("fill", "#445312")
        }

        function annotateScatter(x, y, xValue, yValue, svg, area, mob, income, offset, yOffset, xOffset) {
            // console.log(y)
            const type = annotationCalloutCircle

            const annotations = [{
            note: {
                label: `Avg. Household income: ${(income/1000).toFixed()+"k"}, ${mob.toFixed()+"%"} in travel to workplaces`,
                bgPadding: 10,
                title: area
            },
            connector: {
              end: "dot",
              // type: "curve",
            },
            //can use x, y directly instead of data
            data: {xVar: xValue, yVar: yValue},
            className: "show-bg",
            subject: {
              radius: country==="UK"?30:15,
              radiusPadding: 5},
            dy: yOffset,
            dx: xOffset
            }]


            const makeAnnotations = annotation()
            // .editMode(true)
            //also can set and override in the note.padding property
            //of the annotation object
            .notePadding(15)
            .type(type)
            //accessors & accessorsInverse not needed
            //if using x, y in annotations JSON
            .accessors({
                x: d => x(d.xVar)+offset,
                y: d => y(d.yVar)
            })
            // .accessorsInverse({
            //     category: d => x.invert(d.x),
            //     row: d => y.invert(d.y)
            // })
            .annotations(annotations)

            // console.log(d3.annotation())

            // const annotationData = [{ category: category, row: row, text: text}]

            // // const annotations = 
            // d3.select("#chart")//.select("svg")
            // // .data(annotationData)
            d3.select(svg).append("g")
            .attr("class", "annotation-group")
            .style("font-size", "15px")
            // .append("text")
            // .attr("x", d=>x(d.category))
            // .attr("y", d=>y(d.row))
            // .text(text)
            .call(makeAnnotations)
            // .transition().duration(1000)
            // .attr("transform", `translate(${x.bandwidth()}, 0)`)

            d3.selectAll(".annotation-group").attr("opacity", 0).transition().duration(1000).attr("opacity", 0.6)
            // d3.selectAll(".annotation-note-label").attr("fill", "#445312")
            // d3.selectAll(".annotation-connector path").attr("stroke", "#445312").attr("fill", "#445312")
        }



        $: if (annot && innerWidth<550) {
          console.log(annot)
          d3.selectAll(".LStextUK").attr("visibility", "hidden")
          // d3.select(".tick").attr("font-size", "8")
          // d3.select(".axisTitle").attr("font-size", "8")
        } else if (annot && innerWidth>550) {
          d3.selectAll(".LStextUK").attr("visibility", "visible")
          // d3.select(".tick").attr("font-size", "11")
          // d3.select(".axisTitle").attr("font-size", "11")
          // annot.attr('font-size', fontSize*2)
        }

        // $: if (innerWidth<850) {
        //   d3.selectAll(".chart").style("top", "15%")
        // } else if (innerWidth<820) {
        //   d3.selectAll(".chart").style("top", "30%")
        // } else if (innerWidth<750) {
        //   d3.selectAll(".chart").style("top", "40%")
        // } else if (innerWidth<700) {
        //   d3.selectAll(".chart").style("top", "60%")
        // } else if (innerWidth<650) {
        //   d3.selectAll(".chart").style("top", "100%")
        // } else if (innerWidth<550) {
        //   d3.selectAll(".chart").style("top", "200%")
        // } else {
        //   d3.selectAll(".chart").style("top", "5%")
        // }

        $: widthHeightRatio = innerWidth/innerHeight*100

        $: if (innerWidth<innerHeight) {
          d3.selectAll(".chart").style("top", `30%`)
        // } else if (innerWidth<820) {
        //   d3.selectAll(".chart").style("top", "30%")
        // } else if (innerWidth<750) {
        //   d3.selectAll(".chart").style("top", "40%")
        // } else if (innerWidth<700) {
        //   d3.selectAll(".chart").style("top", "60%")
        // } else if (innerWidth<650) {
        //   d3.selectAll(".chart").style("top", "100%")
        // } else if (innerWidth<550) {
        //   d3.selectAll(".chart").style("top", "200%")
        } else {
          d3.selectAll(".chart").style("top", "5vh")
        }

        // legend interaction
        // d3.selectAll(".legendCircle")
        // .on("mouseover", function(event, d) { 
        //   const selectedCat = this.attributes.value.value
        //   d3.select(this).attr("cursor", "pointer").attr("stroke-width", "1.5")
        //   d3.select(this.parentNode).raise()
        //   d3.selectAll(".laCircle").filter(d=>d.category!==selectedCat).attr("opacity", 0.2)
        //   console.log(selectedCat)
        // })
        // .on("mouseout", function(event, d) { 
        //   const selectedCat = this.attributes.value.value
        //   d3.select(this).attr("stroke-width", "0.5").moveToBack()
        //   d3.selectAll(".laCircle").attr("opacity", 1) //.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
        //   console.log(selectedCat)
        // })

        // SCROLLYTELLING
        let currentStep;
        // const steps = country==="UK"?
        //               ["<p>UK Step 0!</p>", 
        //                "<p>UK Step 1?</p>",
        //                "<p>UK Step 2.</p>", 
        //                "<p>UK Step 3.</p>"]:
        //                ["<p>US Step 0!</p>", 
        //                "<p>US Step 1?</p>",
        //                "<p>US Step 2.</p>", 
        //                "<p>US Step 3.</p>"]

        const steps = country==="UK"?
                      ["<p>Comprising England, Scotland, Wales, and Northern Ireland, the UK is home to about 67.2 million residents across 374 local authorities.  Similarly to our US example, each UK  local authority is represented by a single circle in the following cartogram.</p>", 
                       
                      "<p>While considered a high income country by the World Bank, some areas of this sovereign state are home to residents that earn higher income than others.  In the following cartogram, each circle's color tone represents the median household income of that local authority.</p>",
                       
                      "<p>Just like with our US example, this same cartogram approach can also be used to investigate the impact of COVID-19 on mobility patterns to the workplace.</p>", 
                       
                      "<p>As with our US example, in considering both household income and mobility patterns to the workplace, insight can be generated into who benefits from WFH in the pandemic era.", 
                       
                      "<p>For example, take Islington.  This local authority is observed to be home to high income residents that were able to reduce their mobility to the workplace during the pandemic.  These residents have benefited from WFH policies.</p>", 
                       
                      "<p>Just like with our US example, a correlation between median household income and residents' travel to the workplace can be observed.  Again, and in in general, the wealthier local authority residents are, the more likely that they are to have benefited from WFH policies and reduced their mobility to the office.</p>", 
                       
                      "<p>Consider the local authority of the City of London.  Here, with a median household income of about £65k, residents are categorized as high income.  Since February 2020, these residents were able to reduce their travel to the place by about 55% relative to the pre-pandemic 2019 baseline.</p>", 
                       
                      "<p>Also consider the local authority of Blaenau Gwent.  Here the median household income is categorized as low at  about £ 25k.  Since February 2020, these residents were not able to reduce their travel to the workplace relative to the pre-pandemic 2019 baseline.</p>", 
                       
                      "<p>While most UK local authorities are distinguished by the same inverse linear relationship between income and mobility patterns observed in our US example, outliers to such norm do exist.<br><br>For example, Glasgow, a local authority in Scotland, defies this notion.  Here, residents can be categorized as low income, as the median household income is about £27k.  Despite this low income status, Glasgow residents were able to reduce their mobility to the workplace by about 43% relative to the pre-pandemic 2019 baseline.   However with about <a href='https://www.heraldscotland.com/news/13063064.students-boost-glasgows-economy-500m-year/#:~:text=Around%2067%2C000%20people%20%E2%80%93more%20than,average%20%C2%A311%2C000%20a%20year' target=__blank>11% of the Glasgow population enrolled in some form of higher-education</a>, the student status of many residents may help explain this phenomenon.  While many students may be considered low income, their affiliated universities have all made efforts to switch to online learning during the pandemic.  As such, many students have continued to learn from home, and they no longer commute to campuses.</p>", 
                       
                      "<p>The local authority of East Hampshire also defies this typical notion of the income and mobility pattern relationship.  Here, residents are categorized as high income with a median household income of about £54k.  Regardless of this high income status, however, these residents only experienced a slight 25% reduction in their mobility to the workplace.  Here, residents have not greatly benefited from WFH policies.</p>", 
                       
                      "<p>Despite outliers like Glasgow and East Hampshire, the majority of local authorities fall into one of the same two extremes as our US example.  UK residents are also either 1) high income and they have benefited greatly from WFH policies, or they are 2) low income and they have not benefited greatly from WFH policies.<br><br>Just like the US, the “Low Income, High Travel” and “High Income, Low Travel” bins encompass the most local authorities when compared to the other income/mobility categories.</p>", 
                       
                      "<p>Diving deeper into our understanding of <i>who</i> benefits from WFH policies in the UK, local authorities were then also categorized by their urban and rural statuses. <br><br>In the UK, “Low Income, High Travel” local authorities (orange) can be found in both urban and rural settings.  However, a majority of the “High Income, Low Travel” local authorities (green) are categorized as urban areas.  But <i>which urban areas in the UK are home to such high income residents who benefit from WFH?</i></p>", 
                       
                      "<p>As it turns out, an overwhelming majority of “High Income, Low Travel” local authorities (green) are located in London's Larger Urban Zone.  <i>What exactly does this mean?</i></p>", 
                       
                      "<p>Relevant to household income and WFH benefits, clear social and spatial segregation exists in the UK.  A strong correlation is present where wealthier residents benefit more than low income residents from WFH policies in response to the COVID-19 pandemic. What's more? These wealthier residents overwhelmingly reside in London's Larger Urban Zone.  Similarly to the US, has the UK forgotten about its rural residents?  Or, has the UK championed its financial center, London, to the dismay of everywhere else in the kingdom?</p>",
                       
                      "<p>The evidence seems stark to us.  But once again, explore the data and decide for yourself.</p>"]:

                      ["<p>With regard to the US, we learned that about 330 million residents dispersed over 3,174 counties call the nation home.  Here, with a cartogram approach, each of these counties are illustrated with a circle.</p>", 

                       "<p>Of course, of these 330 million US residents, some are wealthier than others.  The median household income of each county can be observed in the following figure.  While each county is represented by a circle in the cartogram, the median household income of residents in that county is represented by the circle's color tone.  Counties colored in dark green represent areas where residents are higher income.  Counties colored in light green represent areas where residents are low income.</p>",

                       "<p>This same cartogram approach can also be used to investigate the impact of COVID-19 on US mobility patterns to the workplace.  While employer implemented WFH policies during the COVID-19 pandemic have allowed some US residents to earn a living from home, many people must still commute to the workplace, despite the public health risk, in search of income.  As illustrated below, counties colored in dark red represent areas where residents have benefited from WFH policies (i.e., these residents were able to greatly reduce their mobility to the workplace during the pandemic).  Contrastingly, counties colored in light orange represent areas where residents were unable to benefit from WFH, and they have continued to commute to the workplace.</p>", 

                       "<p>In considering both US household income and US mobility patterns to the workplace, insight can be generated into who benefits from WFH in the pandemic era.  The resulting color scheme of such investigation is twofold.  Each of the cartogram circles' color tone is dependent on both 1) the median household income of the county's residents and 2) the residents' percent change in mobility to workplaces relative to a pre-pandemic 2019 baseline. </p>", 

                       "<p>Consider, for example, New York County (New York).  This county is observed to be home to high income residents with a median household income of about $83k.  In general, these residents were able to reduce their mobility to the workplace during the pandemic by 55% when compared to the 2019 baseline.  As such, these residents have benefited from WFH policies.</p>", 

                       "<p>A striking correlation between median household income and residents' travel to the workplace can be observed.  In general, the wealthier county residents are, the more likely that they are to have benefited from WFH policies and reduced their mobility to the office.</p>", 

                       "<p>Take San Francisco County (California).  Here, with a median household income of about $105k, residents are categorized as high income.  Since February 2020, these residents were able to reduce their travel to the place by about 61% relative to the pre-pandemic 2019 baseline.</p>", 

                       "<p>On the other hand, consider Douglas County (Missouri).  Here the median household income is categorized as low at about $34k.  Since February 2020, these residents were only able to reduce their travel to the workplace by a meager 5% relative to the pre-pandemic 2019 baseline.</p>", 

                       "<p>While most US counties are distinguished by this inverse linear relationship between income and mobility patterns, outliers to such norm do exist.<br><br>For example, the county of Lafayette (Arkansas) defies this notion.  Here, residents can be categorized as low income, as the median household income is about $32k.  Despite this low income status, Lafayette County (Arkansas) residents were able to reduce their mobility to the workplace by a staggering 56% relative to the pre-pandemic 2019 baseline.</p>", 

                       "<p>Nantucket County (Massachusetts) also defies this typical notion of the income and mobility pattern relationship.  Here, residents are categorized as high income with a median household income of about $105k.  Regardless of this high income status, however, these residents only experienced a scant 3% reduction in their mobility to the workplace.  Here, residents have not greatly benefited from WFH policies.</p>",

                       "<p>Despite outliers like Lafayette County (Arkansas) and Nantucket County (Massachusetts), a disproportionate number of US counties falls into one of two extremes.  County residents are either 1) high income and they have benefited greatly from WFH policies, or they are 2) low income and they have not benefited greatly from WFH policies.<br><br>As observed in the following figure, the “Low Income, High Travel” and “High Income, Low Travel” bins encompass the most US counties when compared to the other income/mobility categories.</p>", 

                       "<p>Diving deeper into our understanding of <i>who</i> benefits from WFH policies in the US, counties were then categorized by their urban and rural statuses.<br><br>Shockingly, in the US, a majority of “Low Income, High Travel” counties (orange) are found in rural settings. In addition, an overwhelming number of “High Income, Low Travel” counties (green) are categorized as urban areas.  A clear divide exists.  Residents of urban US counties are more likely to be earning higher median household incomes and benefiting from WFH policies when compared to their rural counterparts.</p>", 

                       "<p>Relevant to household income and WFH benefits, clear social and spatial segregation exists in the US.  A strong correlation is present where wealthier residents benefit more than low income residents from WFH policies in response to the COVID-19 pandemic. What's more? These wealthier residents overwhelmingly reside in urban counties.  <i>Has the US forgotten about its rural residents?</i>  The data most definitely presents this way.</p>",
                       
                       "<p>But don't take our word for it.  Explore the data yourself.</p>"]


        // raise and lower functions
        d3.selection.prototype.moveToFront = function() {
        return this.each(function(){
          this.parentNode.appendChild(this);
            });
        };

      d3.selection.prototype.moveToBack = function() {
        return this.each(function() {
            var firstChild = this.parentNode.firstChild;
            if (firstChild) {
                this.parentNode.insertBefore(this, firstChild);
            }
          });
      };

        // search bar
        let options = country==="UK"?hexesClean.map(d=>{return{value:d.key, text:d.n}}):hexesClean.map(d=>{return{value:d.fullName.replaceAll(",", "").replaceAll(" ", ""), text:d.county+","+d.abbrv}});

        $: selectionUS = null;
        $: selectionUK = null;
        $: valueUK = null;
        $: valueUS = null;
        $: GEOID = null

        // setTimeout(() => {
        //   value = 'de';
        // }, 4000);

        $: if (valueUK!== null) {
          console.log(valueUK)
          d3.selectAll(".laCircleUK").attr("opacity", 0.4).attr("r", radiusUK).attr("stroke", "#fafafa").attr("stroke-width", 0.5).moveToBack()
          // d3.selectAll(".LStextUK").attr("font-size", fontSize).style("font-weight", "500")
          d3.select("#"+valueUK+"Group").raise()//.attr("stroke-width", 1.5).raise()
          d3.select("#"+valueUK).attr("opacity", 1).attr("stroke-width", 2).attr("r", radiusHoverUK).moveToFront()//.attr("stroke-width", 1.5).raise()
          d3.selectAll(".LStextUK").attr("font-size", fontSize).style("font-weight", "500")
          d3.select("#label"+valueUK).attr("opacity", 1).attr("font-size", fontSize*2).style("font-weight", "700").moveToFront()
          
        } else if (valueUK === null) {
          d3.selectAll(".LStextUK").attr("font-size", fontSize).style("font-weight", "500")
          d3.selectAll(".laCircleUK").attr("opacity", 1).attr("stroke", "#fafafa").attr("stroke-width", 0.5).attr("r", radiusUK)
          // d3.select("#"+value).attr("opacity", 1)//.attr("stroke-width", 0.5).lower()
        }

        $: if (valueUS!== null) {
          console.log(valueUS)
          // d3.selectAll(".stateAbbrv").attr('opacity', '0.5')
          d3.select("#"+valueUS+"Group").raise()
          d3.selectAll(".laCircleUS").attr("opacity", 0.4).attr("r", radiusUS).attr("stroke", "#fafafa").attr("stroke-width", 0.5).moveToBack()
          d3.select("#"+valueUS).attr("opacity", 1).attr("stroke-width", 2).attr("r", radiusHoverUS).moveToFront()//.attr("stroke-width", 1.5).raise()
        } else if (valueUS === null) {
          // d3.selectAll(".stateAbbrv").attr('opacity', '1')
          d3.selectAll(".laCircleUS").attr("opacity", 1).attr("stroke", "#fafafa").style("font-weight", "500").attr("stroke-width", 0.5).attr("r", radiusUS)
          // d3.select("#"+value).attr("opacity", 1)//.attr("stroke-width", 0.5).lower()
        }

</script>
<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight /> <!--on:resize='{resize}' -->
<!-- <div id="staticTooltip"></div> -->
<div class="chart">
<!-- <label for="areaSel">{country==="UK"?"Select a local authority":"Select a county"}</label> -->
<!-- {#if country==="UK" && currentStep===14}
<Svelecte {options} 
  inputId="areaSelUK"
  bind:readSelection={selectionUK}
  bind:value={valueUK}
  placeholder={country==="UK"?"Select a local authority":"Select a county"}
></Svelecte>
{:else if country === "US"&& currentStep===13}
<Svelecte {options} 
  inputId="areaSelUS"
  bind:readSelection={selectionUK}
  bind:value={valueUS}
  placeholder={country==="UK"?"Select a local authority":"Select a county"}
></Svelecte>
{/if} -->

<!-- legends and axes -->
<div class="chartElements">
  <div class="legendContainer">
  {#if currentStep == 0}
  <span class="frameTitle"></span>
  <!-- <span class="frameTitle">A map of the U.S.</span> -->
  {:else if currentStep == 1}
    <UnivariateLegend {width} country={country} {hexesClean} colorVar={"income"} numTicks={6}/>
  {:else if currentStep == 2}
    <UnivariateLegend {width} country={country} {hexesClean} colorVar={"mobility"} numTicks={6}/>
  {:else if currentStep == 3}
    <Legend {colors} {country}></Legend>
  {:else if country==="UK" && currentStep===14}
    <div class="searchDropdown">
      <Svelecte {options} 
        inputId="areaSelUK"
        bind:readSelection={selectionUK}
        bind:value={valueUK}
        placeholder={country==="UK"?"Select a local authority":"Select a county"}
      ></Svelecte>
    </div>
    {:else if country === "US"&& currentStep===13}
    <div class="searchDropdown">
      <Svelecte {options} 
        inputId="areaSelUS"
        bind:readSelection={selectionUK}
        bind:value={valueUS}
        placeholder={country==="UK"?"Select a local authority":"Select a county"}
      ></Svelecte>
    </div>
    {/if}
  </div>
  <svg viewBox="0 0 800 600" bind:this={svg}>
  {#if currentStep==5||currentStep==6||currentStep==7||currentStep==8||currentStep==9}
    <!-- y axis -->
    <g class='axis y-axis'>
      {#each yTicksMob as tick}
        <g class='tick tick-{tick}' transform='translate(0, {normScaleYMob(tick)})'>
          <line x1='{margin.left}' x2='{width-margin.right}'/>
          <text x='{margin.left - 8}' y='+4' font-size={innerWidth>650?"12px":innerWidth<550? "18px":innerWidth<500? "22px" :"12px"}>{tick}</text>
        </g>
      {/each}
    </g>
    <g class='axisTitle' text-anchor=start transform='translate(0, {country==="UK"?normScaleYMob(-12):normScaleYMob(1)})'>
      <text x={margin.left} y="+0" font-size={innerWidth>650?"12px":innerWidth<550? "18px":innerWidth<500? "22px" :"12px"}>AVERAGE CHANGE IN PEOPLE GOING TO WORKPLACES SINCE FEB. 2020 (%)</text>
    </g>
    <!-- x axis -->
    <g class='axis x-axis'>
      {#each xTicksIncome as tick}
        <g class='tick' transform='translate({normScaleXInc(tick)},0)'>
          <line y1='{normScaleYMob(0)}' y2='{normScaleYMob(d3.min(yTicksMob))}'/>
          <text y='{height - margin.bottom + 16}' font-size={innerWidth>650?"12px":innerWidth<550? "18px":innerWidth<500? "22px" :"12px"}>{tick}</text>
        </g>
      {/each}
    </g>
    <g class='axisTitle' text-anchor=end transform='translate({country==="UK"?normScaleXInc(65000):normScaleXInc(140000)}, {height - margin.bottom})'>
      <text x={0} y="+35" font-size={innerWidth>650?"12px":innerWidth<550? "18px":innerWidth<500? "22px" :"12px"}>MEDIAN HOUSEHOLD INCOME</text>
    </g>
    {:else if currentStep==10}
    <g class='axis x-axis'>
      {#each categoriesX as tick, i}
        <g class='axisTitle' transform='translate({normScaleCatX(tick)+margin.left/4},{0})' text-anchor=middle>
          {#each categoryLabels[i].split(",") as words, m}
            <text 
            y='{height - margin.bottom + 20}' 
            font-size={innerWidth>650?"10px":innerWidth<550? "10px":innerWidth<500? "12px" :"10px"}
            transform="translate(0, {(m-1)*12})"
            >{words}</text>
          {/each}
        </g>
      {/each}
    </g>
    {:else if currentStep==11}
    <g class='axis x-axis'>
      {#each urbCategoryLabels as tick}
        <g class='axisTitle' transform='translate({normScaleCatXUrb(tick)+margin.left+10},{0})' text-anchor=middle>
          <text y='{height - margin.bottom + 16}' font-size={innerWidth>650?"10px":innerWidth<550? "10px":innerWidth<500? "12px" :"10px"}>{tick}</text>
        </g>
      {/each}
    </g>
    {:else if currentStep==12 && country==="UK"}
    <g class='axis x-axis'>
      {#each leCatX as tick, i}
        <g class='axisTitle' transform='translate({normScaleCatXLe(tick)+margin.left+10},{0})' text-anchor=middle>
          <text y='{height - margin.bottom + 16}' font-size={innerWidth>650?"10px":innerWidth<550? "10px":innerWidth<500? "12px" :"10px"}>{leCatLabels[i]}</text>
        </g>
      {/each}
    </g>
    <!-- {:else if currentStep == 1}
    <UnivariateLegend {width} country={"UK"} {hexesClean} colorVar={"income"}/> -->
    {/if}
  </svg>
  </div>
</div>
<Scroll bind:value={currentStep}>
  {#each steps as text, i}
    <div class="step" class:active={currentStep === i}>
      <div class="step-content">
        {@html text}
      </div>
    </div>
  {/each}
</Scroll>
<!-- <div class="spacer" /> -->
<!-- <div class='spacer'>
  <h1> 
    Thanks!
  </h1>
  <h2>
    <a href='https://twitter.com/CL_Rothschild' target="_blank">Questions and Tips</a>
  </h2>
</div> -->
<!-- <p>Outerwidth: {outerWidth}</p>
<p>Innerwidth: {innerWidth}</p> -->
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
		fill: #445312;
    font-weight: 700;
    text-transform: uppercase
	}

  .axisText {
    background: #445312;
    color: #fafafa;
  }


.legendTitle {
  font-family:'Roboto', sans-serif;
  font-size: 10px;
  font-weight: 200;
  text-transform: None;
  fill: #445312;
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
    font-size: calc(10px + 1.2vw);
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