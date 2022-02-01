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

    // view reactivity
    $: view = "horizontal"  
    $: outerWidth = 0
    $: innerWidth = 0
    $: outerHeight = 0
    $: innerHeight = 0

    // export let vizTheme;
    export let colors;
    export let hex_la;
    export let ukUpd_tot;
    export let ukUpd_time;
    export let ukUrbRural;
    export let uSuPd_tot;
    export let hex_us;
    export let country;

    // console.log(hex_us)

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

    $: radius = country==="UK"?width/80:width/220//9.8
    $: radiusHover = radius*2
    $: boundWidth = 4
    $: fontSize = 6.5
    $: yVar = "mobilityWork"
    // const xVar = "mobilityRes"
    $: xVar = "income"
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
    let dataBivar = country==="UK"?Object.assign(new Map(ukUpd_tot.map(d=>[d.area_code, [d["workplaces_percent_change_from_baseline"], d["Total annual income (£)"]]])), {title: ["Travel to Work", "Med. Income"]}):
                                   Object.assign(new Map(uSuPd_tot.map(d=>[d.area_code, [d["workplaces_percent_change_from_baseline"], d["Total annual income (£)"]]])), {title: ["Travel to Work", "Med. Income"]})
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

    // Render the hexes
    let hexes = country==="UK"?
    renderHexJSON(hex_la, width, height).map(d=> ({ ...d, 
          income: 
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0] === undefined)||
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0]["Total annual income (£)"] === null)||
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0]["Total annual income (£)"] === undefined)?null:
                ukUpd_tot.filter(c=>c.area_code===d.key)[0]["Total annual income (£)"],
          mobilityWork: 
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0] === undefined)||
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0]["workplaces_percent_change_from_baseline"] === null)||
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0]["workplaces_percent_change_from_baseline"] === undefined)?null:
                ukUpd_tot.filter(c=>c.area_code===d.key)[0]["workplaces_percent_change_from_baseline"],
                                                  
          mobilityRes: 
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0] === undefined)||
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0]["residential_percent_change_from_baseline"] === null)||
                (ukUpd_tot.filter(c=>c.area_code===d.key)[0]["residential_percent_change_from_baseline"] === undefined)?null:
                ukUpd_tot.filter(c=>c.area_code===d.key)[0]["residential_percent_change_from_baseline"],
          
          category: ukUpd_tot.filter(c=>c.area_code===d.key)[0] === undefined ? "#ccc": 
          ukUpd_tot.filter(c=>c.area_code===d.key)[0]["Total annual income (£)"] === null? "#ccc":
          ukUpd_tot.filter(c=>c.area_code===d.key)[0]["Total annual income (£)"] === undefined ? "#ccc":        
          colorBivar([ukUpd_tot.filter(c=>c.area_code===d.key)[0]["workplaces_percent_change_from_baseline"],ukUpd_tot.filter(c=>c.area_code===d.key)[0]["Total annual income (£)"]]),

          urbCategory: ukUrbRural.filter(c=>c.LAD11CD===d.key)[0] === undefined ? null: 
          ukUrbRural.filter(c=>c.LAD11CD===d.key)[0]["RUC11"] === null? null:
          ukUrbRural.filter(c=>c.LAD11CD===d.key)[0]["RUC11"] === undefined ? null:        
          ukUrbRural.filter(c=>c.LAD11CD===d.key)[0]["RUC11"],
                                                                            
        })):
        hex_us.map(d=> ({ ...d, 
          income: 
                uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]===undefined?null:
                uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["Total annual income (£)"],
                
          mobilityWork: 
                uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]===undefined?null:
                uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["workplaces_percent_change_from_baseline"],
                                                  
          mobilityRes: 
                uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]===undefined?null:
                uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["residential_percent_change_from_baseline"],
          
          category: uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]===undefined? "#ccc": 
          uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["Total annual income (£)"] === null? "#ccc":
          uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["Total annual income (£)"] === undefined ? "#ccc":        
          colorBivar([uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["workplaces_percent_change_from_baseline"],uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["Total annual income (£)"]]),

          urbCategory: 
                  uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]===undefined?null:
                  uSuPd_tot.filter(c=>c.fullName === d.fullName)[0]["urbCategory"]

                                                                            
        }))
        
        // console.log("urb",ukUrbRural)
        console.log("hex us", hexes)

        // variables for dot clusters bars
        let clusterData = d3.groups(hexes, v=>v.category).map(d=> { return {category: d[0], data: d[1].map((c, i) =>({ ...c, row: i}))}}).filter(d=>d.category!=="#ccc")
        console.log("cluster", clusterData)

        //   variables for dot clusters bars urban rural
        let ruralData = country==="UK"?
        d3.groups(
            ukUrbRural
            .map(d=>({...d, category: hexes.filter(c=>c.key===d.LAD11CD)[0]!==undefined?
                                    hexes.filter(c=>c.key===d.LAD11CD)[0].category:null
                                            }))
            .filter(d=>d.category!=="#ccc")
            .filter(d=>hexes.map(d=>d.key).filter(onlyUnique).includes(d.LAD11CD)), v=>v.RUC11)
            .map(d=> { 
                return {urbCategory: d[0], data: d[1].sort(function(a, b) {
                return sortOrder.indexOf(a.category) - sortOrder.indexOf(b.category);
                })//.sort((a,b)=> d3.descending(a.category,b.category))
                .map((c, i) =>({ ...c, row: i}))
                }
            }):
            d3.groups(
              hexes,
            //   .map(d=>({...d, category: hexes.filter(c=>c.key===d.LAD11CD)[0]!==undefined?
            //                                       hexes.filter(c=>c.key===d.LAD11CD)[0].category:null
            //                                     }))
                // .filter(d=>hexes.map(d=>d.fullName).filter(onlyUnique).includes(d.fullName)), 
                v=>v.urbCategory
                )
                .filter(d=>d[0]!==null)
                .map(d=> { 
                  return {urbCategory: d[0], data: d[1]
                    .filter(d=>d.urbCategory!==null&&colors.includes(d.category)&&d.mobilityWork!==null&&d.income!==null).sort(function(a, b) {
                    return sortOrder.indexOf(a.category) - sortOrder.indexOf(b.category);
                  })//.sort((a,b)=> d3.descending(a.category,b.category))
                    .map((c, i) =>({ ...c, row: i}))
                    }
                })

        console.log("urb rural", ruralData)
        let urbCategoriesX = ruralData.sort((a,b)=>b.data.length-a.data.length).map(d=>d.urbCategory)   
        let urbCategoryLabels = ["Urban", "Rural"]

        var featureCollection = country==="US"?
        { type:"FeatureCollection", features: hexes.map(function(d) {
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
                        .domain(d3.extent(hexes, d => d.income))
                        .range([margin.left, width - margin.right])
      
        $: normScaleYMob = d3.scaleLinear()
                        .domain(d3.extent(hexes, d => d.mobilityWork))
                        .range([height - margin.bottom, margin.top])

        // let normScaleXhex = d3.scaleLinear()
        //                 .domain(d3.extent(hexes, d => d.x))
        //                 .range(margin.top, figWidth - margin.bottom)
      
        // let normScaleYhex = d3.scaleLinear()
        //                 .domain(d3.extent(hexes, d => d.y))
        //                 .range([margin.top, figWidth - margin.bottom])

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


        let hexesClean = country==="UK"?
                              hexes.map(d=>({...d, mobilityWork: d.mobilityWork,
                                             income: d.income,
                                             x: d.x,
                                             y: d.y,
                                             category: d.category,
                                             urbCategory: d.urbCategory,
                                             urbRow: d.urbCategory!==null && 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0]!==undefined? 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0].row:null,
                                             catRow: d.category!=="#ccc"? clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.key===d.key)[0].row:null,
                                             

                                            }))
                                            .map(d=>({...d, 

                                              paddingCat: {x:getPadding("category", d.catRow, d.urbRow, d.urbCategory).x, y:getPadding("category", d.catRow, d.urbRow, d.urbCategory).y},
                                              paddingUrb: {x:getPadding("urbCategory", d.catRow, d.urbRow, d.urbCategory).x, y:getPadding("urbCategory", d.catRow, d.urbRow, d.urbCategory).y}
                                              
                                            })):
                              hexes.map(d=>({...d, mobilityWork: d.mobilityWork,
                                             income: d.income,
                                             x: d.x,
                                             y: d.y,
                                             category: d.category,
                                             urbCategory: d.urbCategory,
                                             urbRow: d.urbCategory!==null && 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.fullName===d.fullName)[0]!==undefined?
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.fullName===d.fullName)[0].row:null,
                                             catRow: d.category!=="#ccc"?clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.fullName===d.fullName)[0].row:null,
                                          
                                            }))
                                            .map(d=>({...d, 

                                              paddingCat: {x:getPadding("category", d.catRow, d.urbRow, d.urbCategory).x, y:getPadding("category", d.catRow, d.urbRow, d.urbCategory).y},
                                              paddingUrb: {x:getPadding("urbCategory", d.catRow, d.urbRow, d.urbCategory).x, y:getPadding("urbCategory", d.catRow, d.urbRow, d.urbCategory).y}
                                              
                                            }))

        console.log(hexesClean)                                    

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
                .attr("r", radius)//innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius)
                .attr("stroke", "#fffae7")
                .attr("stroke-width", "0.5")
                .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
                .attr("class", "laCircle")
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
                .attr('class', 'LStextUK')
                .attr('font-size', fontSize)
                .attr('fill', 'rgb(255,255,255)')
                .attr("z-index", 10)
                .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .text(d=>country==="UK"?d.n.slice(0,3):"")
                .on("mouseover", (event, d)=>console.log(innerWidth))
                .attr("cursor", "pointer")

          // annot = true

                // resize()
        })

        $: console.log("normalized", selectedView.value)

        $: 
        
        // if (hexmap && circles && annot && selectedView.value==="chart") { 

        if (hexmap && circles && annot && currentStep===1) { 

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
          .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
          // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
          //                          country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
          .attr("transform", country==="UK"&&currentStep===0?`translate(${margin.left*2},0)`:
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
          .attr("transform",currentStep===0?`translate(${-margin.left},0)`:`translate(0,0)`)

          // } else if (hexmap && circles && annot && selectedView.value==="map") {

          } else if (hexmap && circles && annot && currentStep===0) {

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
            .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
            // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
            //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
            .attr("transform", country==="UK"&&currentStep===0?`translate(${margin.left*2},0)`:
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
            .attr("transform",currentStep===0?`translate(${margin.left*2},0)`:`translate(0,0)`)

          // } else if (hexmap && circles && annot && selectedView.value==="bars") {
          } else if (hexmap && circles && annot && currentStep===2) {

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
            .attr("cx", d => normScaleCatX(d.category)+d.paddingCat.x)
            .attr("cy", d=> normScaleCatRow(d.catRow+d.paddingCat.y))
            .attr("opacity", d=>d.income!==null?1:0)
            .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
            // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
            //                        country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
            .attr("transform", country==="UK"&&currentStep===0?`translate(${margin.left*2},0)`:
                                   country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)
            
            annot.filter(d=>d.category!=="#ccc")
            .transition()
            .delay((d, i) => {
              return i * Math.random() * 1.5;
              })
            .duration(800)
            .ease(d3.easeLinear)
            .attr("x", d => normScaleCatX(d.category)+d.paddingCat.x)
            .attr("y", d=> normScaleCatRow(d.catRow+d.paddingCat.y))
            .attr("opacity", d=>d.income!==null?1:0)
            // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
            .attr("transform",currentStep===0?`translate(${-margin.left},0)`:`translate(0,0)`)


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

          } else if (hexmap && circles && annot && currentStep===3) {

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
            .attr("cx", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrb.x)
            .attr("cy", d=> normScaleUrbRow(d.urbRow+d.paddingUrb.y))
            .attr("opacity", d=>d.income!==null?1:0)
            .attr("fill", d=>[categoriesX[1], categoriesX[0]].includes(d.category)?colorBivar([d.mobilityWork, d.income]):"#ccc")
            // .attr("transform", country==="UK"&&selectedView.value==="map"?`translate(${margin.left*2},0)`:
            //                         country==="US"&&selectedView.value==="map"?`translate(${margin.left},0)`:`translate(0,0)`)
            .attr("transform", country==="UK"&&currentStep===0?`translate(${margin.left*2},0)`:
                                   country==="US"&&currentStep===0?`translate(${margin.left},0)`:`translate(0,0)`)

            annot.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
            .transition()
            .delay((d, i) => {
              return i * Math.random() * 1.5;
              })
            .duration(800)
            .attr("x", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrb.x)
            .attr("y", d=> normScaleUrbRow(d.urbRow+d.paddingUrb.y))
            .attr("opacity", d=>d.income!==null?1:0)
            // .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
            .attr("transform",currentStep===0?`translate(${-margin.left},0)`:`translate(0,0)`)

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

        } 


        function getPadding(x, catRow, urbRow, urbCat) {


            let padding={x:0, y:0}

            if (country==="UK") {

              if (x === "category") {

                // console.log("showing", x, "row", catRow)

                let rowCheck = d3.range(1,3,1).map(d=>{ return {
                  key:[`num${d+1}`],
                  val:d3.range(d, 500, 3)
                }})
                .reduce((map, obj) => (map[obj.key] = obj.val, map), {})
                
                // console.log("rowcheck", rowCheck)

                rowCheck["num3"].includes(catRow)?padding={x:40, y:0}:
                rowCheck["num2"].includes(catRow)?padding={x:20, y:1}:
                padding={x:0, y:2}

              } else if (x === "urbCategory") {

                // console.log("showing", x, "urbRow", urbRow)

                let rowCheck = d3.range(1,10,1).map(d=>{ return {
                  key:[`num${d+1}`],
                  val:d3.range(d, 1000, 10)
                }})
                .reduce((map, obj) => (map[obj.key] = obj.val, map), {})

                // console.log("rowcheck", rowCheck)

                  rowCheck["num10"].includes(urbRow)?padding={x:180, y:0}:
                  rowCheck["num9"].includes(urbRow)?padding={x:160, y:1}:
                  rowCheck["num8"].includes(urbRow)?padding={x:140, y:2}:
                  rowCheck["num7"].includes(urbRow)?padding={x:120, y:3}:
                  rowCheck["num6"].includes(urbRow)?padding={x:100, y:4}:
                  rowCheck["num5"].includes(urbRow)?padding={x:80, y:5}:
                  rowCheck["num4"].includes(urbRow)?padding={x:60, y:6}:
                  rowCheck["num3"].includes(urbRow)?padding={x:40, y:7}:
                  rowCheck["num2"].includes(urbRow)?padding={x:20, y:8}:
                  padding={x:0, y:9}
                
                }

              } else if (country==="US") {
                // padding={x:0, y:9}

                 if (x === "category") {

                // console.log("showing", x, "row", catRow)

                let rowCheck = d3.range(1,8,1).map(d=>{ return {
                  key:[`num${d+1}`],
                  val:d3.range(d, 1000, 8)
                }})
                .reduce((map, obj) => (map[obj.key] = obj.val, map), {})
                
                // console.log("rowcheck", rowCheck)

                rowCheck["num8"].includes(catRow)?padding={x:56, y:0}:
                rowCheck["num7"].includes(catRow)?padding={x:48, y:1}:
                rowCheck["num6"].includes(catRow)?padding={x:40, y:2}:
                rowCheck["num5"].includes(catRow)?padding={x:32, y:3}:
                rowCheck["num4"].includes(catRow)?padding={x:24, y:4}:
                rowCheck["num3"].includes(catRow)?padding={x:16, y:5}:
                rowCheck["num2"].includes(catRow)?padding={x:8, y:6}:
                padding={x:0, y:7}

              } else if (x === "urbCategory") {

                // console.log("showing", x, "urbRow", urbRow)

                let rowCheck = d3.range(1,26,1).map(d=>{ return {
                  key:[`num${d+1}`],
                  val:d3.range(d, 3000, 26)
                }})
                .reduce((map, obj) => (map[obj.key] = obj.val, map), {})

                // console.log("rowcheck", rowCheck)

                  rowCheck["num26"].includes(urbRow)?padding={x:200, y:0}:
                  rowCheck["num25"].includes(urbRow)?padding={x:192, y:1}:
                  rowCheck["num24"].includes(urbRow)?padding={x:184, y:2}:
                  rowCheck["num23"].includes(urbRow)?padding={x:176, y:3}:
                  rowCheck["num22"].includes(urbRow)?padding={x:168, y:4}:
                  rowCheck["num21"].includes(urbRow)?padding={x:160, y:5}:
                  rowCheck["num20"].includes(urbRow)?padding={x:152, y:6}:
                  rowCheck["num19"].includes(urbRow)?padding={x:144, y:7}:
                  rowCheck["num18"].includes(urbRow)?padding={x:136, y:8}:
                  rowCheck["num17"].includes(urbRow)?padding={x:128, y:9}:
                  rowCheck["num16"].includes(urbRow)?padding={x:120, y:10}:
                  rowCheck["num15"].includes(urbRow)?padding={x:112, y:11}:
                  rowCheck["num14"].includes(urbRow)?padding={x:104, y:12}:
                  rowCheck["num13"].includes(urbRow)?padding={x:96, y:13}:
                  rowCheck["num12"].includes(urbRow)?padding={x:88, y:14}:
                  rowCheck["num11"].includes(urbRow)?padding={x:80, y:15}:
                  rowCheck["num10"].includes(urbRow)?padding={x:72, y:16}:
                  rowCheck["num9"].includes(urbRow)?padding={x:64, y:17}:
                  rowCheck["num8"].includes(urbRow)?padding={x:56, y:18}:
                  rowCheck["num7"].includes(urbRow)?padding={x:48, y:19}:
                  rowCheck["num6"].includes(urbRow)?padding={x:40, y:20}:
                  rowCheck["num5"].includes(urbRow)?padding={x:32, y:21}:
                  rowCheck["num4"].includes(urbRow)?padding={x:24, y:22}:
                  rowCheck["num3"].includes(urbRow)?padding={x:16, y:23}:
                  rowCheck["num2"].includes(urbRow)?padding={x:8, y:24}:
                  padding={x:0, y:25}
                
              }

            }
            
            // else {

            //   padding = {x:0, y:0}
            // }

            // console.log("padding is", padding)

            return padding
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
          annot.lower()
          // d3.select(".tick").attr("font-size", "8")
          // d3.select(".axisTitle").attr("font-size", "8")
        } else if (annot && innerWidth>550) {
          annot.raise()
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
          d3.selectAll(".chart").style("top", "10%")
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
        const steps = country==="UK"?
                      ["<p>UK Step 0!</p>", 
                       "<p>UK Step 1?</p>",
                       "<p>UK Step 2.</p>", 
                       "<p>UK Step 3.</p>"]:
                       ["<p>US Step 0!</p>", 
                       "<p>US Step 1?</p>",
                       "<p>US Step 2.</p>", 
                       "<p>US Step 3.</p>"]

</script>
<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight /> <!--on:resize='{resize}' -->
<!-- <div id="staticTooltip"></div> -->
<div class="chart">
<svg viewBox="0 0 800 600" bind:this={svg}>
{#if selectedView.value==="chart"||currentStep==1}
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
  {:else if selectedView.value === "bars"||currentStep==2}
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
  {:else if selectedView.value === "barsUrban"||currentStep==3}
  <g class='axis x-axis'>
    {#each urbCategoryLabels as tick}
      <g class='axisTitle' transform='translate({normScaleCatXUrb(tick)+margin.left+10},{0})' text-anchor=middle>
        <text y='{height - margin.bottom + 16}' font-size={innerWidth>650?"10px":innerWidth<550? "10px":innerWidth<500? "12px" :"10px"}>{tick}</text>
      </g>
    {/each}
  </g>
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

.spacer {
    height: 40vh;
  }

.chart {
    /* background: whitesmoke; */
    /* width: 400px;
    height: 400px; */
    box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2);
    position: sticky;
    top: 5%;
    margin: auto;
    /* height: 100vh */
    /* bottom: 50%; */
  }

  /* Scrollytelling CSS */
  .step {
    height: 100vh;
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