<script>
    import * as d3 from "d3";
    // import {fade, blur, fly, slide, scale, draw} from "svelte/transition";
    import { tweened } from "svelte/motion";
    import * as easings from 'svelte/easing';
    import {renderHexJSON} from "d3-hexjson"
	  import {annotationLabel, annotation} from "d3-svg-annotation"
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
    
    let svg, hexmap, circles, annot;
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
                          .fitSize([width-margin.right-margin.left, height-margin.top-margin.bottom], featureCollection):null;
                
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
                        .range([margin.left, width - margin.right])

        $: normScaleLeRow = d3.scaleLinear()
                        .domain(d3.extent(hexesClean, d => d.leRow))
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatXLe = d3.scaleBand()
                        .domain(leCatX)
                        .range([margin.left, width - margin.right])

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

        console.log("%percent green in commuter belt", pctLe)

        onMount(() => {

          // console.log("innerwidth", innerWidth)
         
          hexmap = d3.select(svg)
              .selectAll("g")
              .data(hexesClean)
              .join("g")
              // .attr("transform", `translate(${margin.left},0)`)

          circles = hexmap
                .append("circle")
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("r", country==="UK"?radiusUK:radiusUS)//innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius)
                .attr("stroke", "#fffae7")
                .attr("stroke-width", "0.5")
                // .attr("fill", d => d.category)
                .attr("fill", "#ccc")
                .attr("class", "laCircle"+country)
                .attr("id", d=> country==="UK"?d.key:d.fullName.replaceAll(",", "").replaceAll(" ", ""))
                .attr("cursor", "pointer")
                // .style("z-index", 100)
                .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                                   country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
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
                .attr("opacity", 1)
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
                .attr("opacity", 1)
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
                .attr("opacity", 1)
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

              } else if (country==="UK" && hexmap && circles && annot && currentStep===7) { 

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

              } else if (country==="UK" && hexmap && circles && annot && currentStep===8) { 

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
                .attr("opacity", d=>d.leCat!=="Not London"?1:0.5)
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
                .attr("opacity", 1)
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
                .attr("opacity", 1)
                // .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .attr("transform",[0,1,2,3,4,13,14].includes(currentStep)?`translate(${margin.left*2},0)`:`translate(0,0)`)

                firstStep=false
                comingFromMap = true
                // valueUK = null

          }


        

        function annotateBars(x, y, category, row, svg, text, offset, yOffset, xOffset) {
            // console.log(y)
            const type = annotationLabel

            const annotations = [{
            note: {
                label: text,
                bgPadding: 10,
                // title: text
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
                x: d => x(d.category)+offset,
                y: d => y(d.row)
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
            d3.selectAll(".annotation-note-label").attr("fill", "#445312")
            d3.selectAll(".annotation-connector path").attr("stroke", "#445312").attr("fill", "#445312")
        }

        $: if (country==="US" && hexmap && circles && annot && currentStep===0 && firstStep===false){

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
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

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

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", d => d.category!=="#ccc"?incomeColor(d[xVar]):"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

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
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", d => d.category!=="#ccc"?mobilityColor(d[yVar]):"#ccc")
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

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
                .delay((d, i) => {
                  return i * Math.random() * 1.5;
                  })
                .duration(800)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity", 1)
                .attr("fill", d => d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

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
                valueUK = null
                valueUS = "SpokaneCountyWashington"

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
                .attr("transform", [0,1,2,3,4,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)


                // } else if (hexmap && circles && annot && selectedView.value==="bars") {
              } else if (country==="US" && hexmap && circles && annot && currentStep===5) { 

              // d3.select(".chart").style("position", "sticky")
              d3.selectAll(".annotation-group").remove()

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

                  // highlight islington
                  valueUK = null
                  valueUS = "SanFranciscoCountyCalifornia"

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

              } else if (country==="US" && hexmap && circles && annot && currentStep===7) { 

                  // highlight islington
                  valueUK = null
                  valueUS = "DouglasCountyMissouri"

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

              } else if (country==="US" && hexmap && circles && annot && currentStep===8) { 

                  // highlight islington
                  valueUK = null
                  valueUS = "LafayetteCountyArkansas"

                  circles.filter(d=>d.category==="#ccc")
                .transition()
                .duration(0)
                .ease(d3.easeLinear)
                // .attr("cx", 200)
                // .attr("cy", 1000)
                .attr("opacity", 0)

               } else if (country==="US" && hexmap && circles && annot && currentStep===9) { 

                  // highlight islington
                  valueUK = null
                  valueUS = "NantucketCountyMassachusetts"

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

            } else if (country==="US" && hexmap && circles && annot && currentStep===12) {

                // d3.select(".chart").style("position", "sticky")
                d3.selectAll(".annotation-group").remove()

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
                .attr("transform", [0,1,2,3,12].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

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

                circles
                .transition()
                .delay((d, i) => {
                  return i * Math.random();
                  })
                .duration(700)
                .attr("cx", d=>country==="UK"?d.x:projection([d.x, d.y])[0])
                .attr("cy", d=>country==="UK"?d.y:projection([d.x, d.y])[1])
                .attr("opacity",1)
                .attr("fill", d=>d.category)
                // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
                //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
                .attr("transform", [0,1,2,3,12,13].includes(currentStep)?`translate(${margin.left},0)`:`translate(0,0)`)

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

        $: if (innerWidth<850) {
          d3.selectAll(".chart").style("top", "15%")
        } else if (innerWidth<820) {
          d3.selectAll(".chart").style("top", "30%")
        } else if (innerWidth<750) {
          d3.selectAll(".chart").style("top", "40%")
        } else if (innerWidth<700) {
          d3.selectAll(".chart").style("top", "60%")
        } else if (innerWidth<650) {
          d3.selectAll(".chart").style("top", "100%")
        } else if (innerWidth<550) {
          d3.selectAll(".chart").style("top", "200%")
        } else {
          d3.selectAll(".chart").style("top", "2%")
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
                      ["<p>The United Kingdom, comprising England, Scotland, Wales, and Northern Ireland, is home to about 67.2 million residents and 374 (get actual number) local authorities.  With a cartogram approach below, each such local authority is represented by a circle. </p>", 
                       
                      "<p>While considered a high income country by the World Bank, some areas of this sovereign state are home to residents that earn higher income than others. Here, each circle's color tone represents the median household income of that local authority.  As such, local authorities colored in dark green represent areas where residents are higher income.  Local authorities colored in light green represent areas where residents are lower income.</p>",
                       
                      "<p>This same cartogram approach can also be used to investigate the impact of COVID-19 on mobility patterns to the workplace.  While employer implemented WFH policies during the COVID-19 pandemic have allowed some people to earn a living from home, many employees must still commute to the workplace, despite the public health risk, in search of income.  As illustrated below, local authorities colored in light orange represent areas where residents have benefited from WFH policies (i.e., these residents were able to greatly reduce their mobility to the workplace during the pandemic).  Contrastingly, local authorities colored in dark red represent areas where residents were unable to benefit from WFH, and they have continued to commute to the workplace.</p>", 
                       
                      "<p>In considering both household income and mobility patterns to the workplace, insight can be generated into who benefits from WFH in the pandemic era.  The resulting color scheme of such investigation is twofold.  Each of the cartogram circles' color tone is dependent on both 1) the median household income of the local authorities's residents and 2) the residents' percent change in mobility to workplaces relative to a pre-pandemic 2019 baseline.<br><br>As such, local authorities colored in green represent areas where residents are higher income and benefited from WFH policies (i.e., these residents were able to greatly reduce their mobility to the workplace). Contrastingly, local authorities colored in orange represent areas where residents are lower income and did not benefit from WFH policies.  Local authorities colored in dark-green represent areas where residents are higher income and continued to travel to the workplace during the COVID-19 pandemic.  Regardless of the wealthy status of these areas, residents did not benefit from WFH practices.  Finally, local authorities colored in light-gray represent areas where residents are lower income and benefited from WFH policies.  Regardless of the poorer status of these areas, residents in these national spatial units were still able to benefit from WFH practices.</p>", 
                       
                      "<p>For example, take Islington.  This local authority is observed to be home to high income residents that were able to reduce their mobility to the workplace during the pandemic.  These residents have benefited from WFH policies.</p>", 
                       
                      "<p>A striking correlation between median household income and residents' travel to the workplace can be observed.  In general, the wealthier local authority residents are, the more likely that they are to have benefited from WFH policies and reduced their mobility to the office.</p>", 
                       
                      "<p>Consider the local authority of the City of London.  Here, with a median household income of about £65k, residents are categorized as high income.  Since February 2020, these residents were able to reduce their travel to the place by about 55% relative to the pre-pandemic 2019 baseline.</p>", 
                       
                      "<p>Also consider the local authority of Blaenau Gwent.  Here the median household income is categorized as low at  about £ 25k.  Since February 2020, these residents were not able to reduce their travel to the workplace relative to the pre-pandemic 2019 baseline.</p>", 
                       
                      "<p>While most UK local authorities are distinguished by this inverse linear relationship between income and mobility patterns, outliers to such norm do exist.<br><br>For example, Glasgow, a local authority in Scotland, defies this notion.  Here, residents can be categorized as lower income, as the median household income is about £ 27k.  Despite this lower income status, Glasgow residents were able to reduce their mobility to the workplace by about 43% relative to the pre-pandemic 2019 baseline.  With about 11% of the Glasgow population enrolled in some form of higher-education, the student status of many residents may help explain this phenomenon.  While many students may be considered lower income, their affiliated universities have all made efforts to switch to online learning during the pandemic.  As such, many students have continued to learn from home, and they no longer commute to campuses.</p>", 
                       
                      "<p>The local authority of East Hampshire also defies this typical notion of the income and mobility pattern relationship.  Here, residents are categorized as high income with a median household income of about £ 54k.  Regardless of this high income status, however, these residents only experienced a slight 25% reduction in their mobility to the workplace.  Here, residents have not greatly benefited from WFH policies.</p>", 
                       
                      "<p>Despite outliers like Glasgow and East Hampshire, the majority of local authorities fall into one of two extremes.  Residents are either 1) high income and they have benefited greatly from WFH policies, or they are 2) lower income and they have not benefited greatly from WFH policies. As observed in the figure below, the “Low Income High Travel” and “High Income Low Travel” bins encompass the most UK local authorities when compared to the other income/mobility categories.</p>", 
                       
                      "<p>Diving deeper into our understanding of who benefits from WFH policies in the UK, local authorities were then categorized by their urban and rural statuses. <br><br>“Low income, High travel” local authorities (orange) can be found in both urban and rural settings.  Similarly, a majority of the “high income and lower travel” local authorities (green) are categorized as urban areas.  But which urban areas in the UK are home to such high income residents who benefit from WFH?</p>", 
                       
                      "<p>As it turns out, an overwhelming majority of “high income and lower travel” local authorities (green) are located in London's Larger Urban Zone.  What exactly does this mean?</p>", 
                       
                      "<p>Relevant to household income and WFH benefits, clear social and spatial segregation exists in the UK.  A strong correlation is present where wealthier residents benefit more than lower income residents from WFH policies in response to the COVID-19 pandemic. What's more? These wealthier residents overwhelmingly reside in London's Larger Urban Zone.  Has the UK forgotten about its rural residents?  Is there an urban vs rural divide?  Or, has the UK simply decided to focus on London specifically, while forgetting about everybody else?</p>",
                       
                      "<p>Here, feel free to interact with the map below to explore these patterns, or use the search bar above to search for your own local authority Otherwise, keep scrolling to find out how WFH patterns have played out in another large large economy: the United States of America.</p>"]:

                      ["<p>US Naked</p>", 
                       "<p>US Income</p>",
                       "<p>US Mobility</p>", 
                       "<p>US Bivariate</p>", 
                       "<p>US Highlight orange and green example</p>", 
                       "<p>US scatter</p>", 
                       "<p>US highlight example</p>", 
                       "<p>US highlight other ex</p>", 
                       "<p>US highlight other ex</p>", 
                       "<p>US highlight other ex</p>",
                       "<p>US bars</p>", 
                       "<p>US urban vs rural</p>", 
                       "<p>US urban + green areas on map</p>",
                       "<p>US Explore the map</p>"]


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

        // setTimeout(() => {
        //   value = 'de';
        // }, 4000);

        $: if (valueUK!== null) {
          console.log(valueUK)
          d3.selectAll(".laCircleUK").attr("opacity", 0.4).attr("r", radiusUK).attr("stroke", "#fffae7").attr("stroke-width", 0.5).moveToBack()
          // d3.selectAll(".LStextUK").attr("font-size", fontSize).style("font-weight", "500")
          d3.select("#"+valueUK).attr("opacity", 1).attr("stroke-width", 2).attr("r", radiusHoverUK).moveToFront()//.attr("stroke-width", 1.5).raise()
          d3.selectAll(".LStextUK").attr("font-size", fontSize).style("font-weight", "500")
          d3.select("#label"+valueUK).attr("opacity", 1).attr("font-size", fontSize*2).style("font-weight", "700").moveToFront()
        } else if (valueUK === null) {
          d3.selectAll(".LStextUK").attr("font-size", fontSize).style("font-weight", "500")
          d3.selectAll(".laCircleUK").attr("opacity", 1).attr("stroke", "#fffae7").attr("stroke-width", 0.5).attr("r", radiusUK)
          // d3.select("#"+value).attr("opacity", 1)//.attr("stroke-width", 0.5).lower()
        }

        $: if (valueUS!== null) {
          console.log(valueUS)
          d3.selectAll(".laCircleUS").attr("opacity", 0.4).attr("r", radiusUS).attr("stroke", "#fffae7").attr("stroke-width", 0.5).moveToBack()
          d3.select("#"+valueUS).attr("opacity", 1).attr("stroke", "#445312").attr("stroke-width", 3).attr("r", radiusHoverUS).moveToFront()//.attr("stroke-width", 1.5).raise()
        } else if (valueUS === null) {
          d3.selectAll(".laCircleUS").attr("opacity", 1).attr("stroke", "#fffae7").style("font-weight", "500").attr("stroke-width", 0.5).attr("r", radiusUS)
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
<div class="legendContainer">
{#if currentStep == 1}
  <UnivariateLegend {width} country={country} {hexesClean} colorVar={"income"} numTicks={6}/>
{:else if currentStep == 2}
  <UnivariateLegend {width} country={country} {hexesClean} colorVar={"mobility"} numTicks={6}/>
{:else if currentStep == 3}
  <Legend {colors}></Legend>
{:else if country==="UK" && currentStep===14}
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
    font-family: 'Lato', sans-serif;
    src: url('https://fonts.googleapis.com/css2?family=Lato&display=swap');
}

@font-face {
    font-family:'Texturina', serif;
    src: url('https://fonts.googleapis.com/css2?family=Texturina:ital,wght@1,300&display=swap');
    font-weight: normal;
    font-style: normal;
}

.legendContainer {
    width:200px;
    height:45px;
    margin:auto;
}

.spacer {
    height: 40vh;
  }

.sv-control {
  width:20vw;
}

.chart {
    /* background: whitesmoke; */
    /* width: 400px;
    height: 400px; */
    /* box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2); */
    position: sticky;
    top: 2%;
    margin: auto;
    /* height: 100vh */
    /* bottom: 50%; */
  }

  /* Scrollytelling CSS */
  .step {
    height: 150vh;
    display: flex;
    place-items: center;
    justify-content: center;
  }

  .step-content {
    background: #f8f6ec;
    color: #ccc;
    border-radius: 5px;
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2);
    z-index: 10;
  }

  .step.active .step-content {
    background: #fffae7;
    color: black;
  }

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
  background-color: #fffae7;
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
    color: #fffae7;
}

/* .vizElement {
  max-width:400px;
} */

body, main {
    background-color: #fffae7;
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

  .axisText {
    background: #445312;
    color: #fffae7;
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