<script>
    import * as d3 from "d3";
    // import {fade, blur, fly, slide, scale, draw} from "svelte/transition";
    import { tweened } from "svelte/motion";
    import * as easings from 'svelte/easing';
    import {renderHexJSON} from "d3-hexjson"
	  import {annotationLabel} from "d3-svg-annotation"
    import {onlyUnique} from "../utils.js"
    import { onMount } from "svelte";

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

    // console.log(hex_us)

    let countryOptions = [
		{ id: 1, value: `UK`, text: `United Kingdom`},
		{ id: 2, value: `US`, text: `United States`}
	];

    let viewOptions = [
		{ id: 1, value: `map`, text: `Map`},
		{ id: 2, value: `chart`, text: `Scatter-plot`},
    { id: 3, value: `bars`, text: `Bars`},
    { id: 4, value: `barsUrban`, text: `Bars Urban`}
	];

	  let selectedCountry = countryOptions[0];
    $: selectedView = viewOptions[0];
    
    let svg, hexmap, circles, annot;
    let scale = 1
    let width = 800
    let height = 600
    // responsive margins
    let margin = ({ top: 0.03*height, right: 0.04*width, bottom: 0.1*height, left: 0.1*width})

    $: radius = width/80//9.8
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
    let categoriesX = [colors[6], colors[2], colors[4], colors[7], colors[5], colors[3], colors[1], colors[0], colors[8]];
    let categoryLabels = ["Low Income, High Travel", "High Income, Low Travel", "Mid. Income, Mid. Travel", "Mid. Income, High Travel", 
                            "High Income, Mid. Travel", "Low Income, Mid. Travel", "Mid. Income, Low Travel", "Low Income, Low Travel", "High Income, High Travel"];

    let sortOrder = [categoriesX[1], categoriesX[0], categoriesX[4], categoriesX[2], categoriesX[7], categoriesX[5], categoriesX[6], categoriesX[3], categoriesX[8]] 
    // bivar settings
    let dataBivar = Object.assign(new Map(ukUpd_tot.map(d=>[d.area_code, [d["workplaces_percent_change_from_baseline"], d["Total annual income (£)"]]])), {title: ["Travel to Work", "Med. Income"]})
    let n = Math.floor(Math.sqrt(colors.length))
    let yBivar = d3.scaleQuantile(Array.from(dataBivar.values(), d => d[1]), d3.range(n))
    let xBivar = d3.scaleQuantile(Array.from(dataBivar.values(), d => d[0]), d3.range(n))
    let labels = ["low", "medium", "high"]
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
    let hexes = renderHexJSON(hex_la, width, height).map(d=> ({ ...d, 
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
                                                                            
        }))
        
        // console.log("urb",ukUrbRural)
        // console.log("hex", hexes)

        // variables for dot clusters bars
        let clusterData = d3.groups(hexes, v=>v.category).map(d=> { return {category: d[0], data: d[1].map((c, i) =>({ ...c, row: i}))}}).filter(d=>d.category!=="#ccc")
        // console.log("cluster", clusterData)

        //   variables for dot clusters bars urban rural
        let ruralData = d3.groups(
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
            })

        // console.log("urb rural", ruralData)
        let urbCategoriesX = ruralData.sort((a,b)=>b.data.length-a.data.length).map(d=>d.urbCategory)                               
       
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
                        .domain([0, 85])
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatX = d3.scaleBand()
                        .domain(categoriesX)
                        .range([margin.left, width - margin.right])


        $: normScaleUrbRow = d3.scaleLinear()
                        .domain([0, 285])
                        .range([height - margin.bottom, margin.top])

        $: normScaleCatXUrb = d3.scaleBand()
                        .domain(urbCategoriesX)
                        .range([margin.left, width - margin.right])



        // let hexesNorm = hexes.map(d=>({...d, mobilityWorkNorm: normScaleYMob(d.mobilityWork),
        //                                      incomeNorm: normScaleXInc(d.income),
        //                                      xNorm: normScaleXhex(d.x),
        //                                      yNorm: normScaleYhex(d.y),
        //                                      rowNorm: d.category!=="#ccc"? normScaleCatRow(clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.key===d.key)[0].row):1720,
        //                                      categoryNorm: normScaleCatX(d.category),
        //                                      urbRowNorm: d.urbCategory!==null && 
        //                                                  ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
        //                                                  ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0]!==undefined? 
        //                                                  normScaleUrbRow(ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0].row):1500,
        //                                     // urbRowNorm: ruralData.filter(c=>c.urbCategory===d.urbCategory)[0],//.data.filter(e=>e.key===d.key)[0]),
        //                                      urbCategoryNorm: normScaleCatXUrb(d.urbCategory),
        //                                      urbRow: d.urbCategory!==null && 
        //                                                  ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
        //                                                  ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0]!==undefined? 
        //                                                  ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0].row:1500,
        //                                      catRow: d.category!=="#ccc"? clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.key===d.key)[0].row:1720,

        //                                     }))

        let hexesClean = hexes.map(d=>({...d, mobilityWork: d.mobilityWork,
                                             income: d.income,
                                             x: d.x,
                                             y: d.y,
                                             category: d.category,
                                             urbCategory: d.urbCategory,
                                             urbRow: d.urbCategory!==null && 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0]!==undefined? 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0].row:1500,
                                             catRow: d.category!=="#ccc"? clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.key===d.key)[0].row:1720,
                                             

                                            }))
                                            .map(d=>({...d, 

                                              paddingCat: {x:getPadding("category", d.catRow, d.urbRow, d.urbCategory).x, y:getPadding("category", d.catRow, d.urbRow, d.urbCategory).y},
                                              paddingUrb: {x:getPadding("urbCategory", d.catRow, d.urbRow, d.urbCategory).x, y:getPadding("urbCategory", d.catRow, d.urbRow, d.urbCategory).y}
                                              
                                            }))

        onMount(() => {

          console.log("innerwidth", innerWidth)
         
          hexmap = d3.select(svg)
              .selectAll("g")
              .data(hexesClean)
              .join("g")
              // .attr("transform", `translate(${margin.left},0)`)

          circles = hexmap
                .append("circle")
                .attr("cx", d=>d.x)
                .attr("cy", d=>d.y)
                .attr("r", radius)//innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius)
                .attr("stroke", "#fffae7")
                .attr("stroke-width", "0.5")
                .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
                .attr("class", "laCircle")
                .attr("cursor", "pointer")
                .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .on("mouseover", (event,d)=>console.log([normScaleXInc(d[xVar]), normScaleYMob(d[yVar])], event.clientX))
                // .on("mouseover", (event, d)=>console.log(normScaleYhex(+d.y)))


          annot = hexmap
                .append("text")
                .attr("x", d=>d.x)
                .attr("y", d=>d.y)
                .attr("text-anchor", "middle")
                .attr('class', 'LStextUK')
                .attr('font-size', fontSize)
                .attr('fill', 'rgb(255,255,255)')
                .attr("z-index", 10)
                .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)
                .text(d=>d.n.slice(0,3))
                .on("mouseover", (event, d)=>console.log(innerWidth))
                .attr("cursor", "pointer")

                resize()
        })

        $: console.log("normalized", selectedView.value)

        $: if (hexmap && circles && annot && selectedView.value==="chart") {

        // svg.selectAll(".AxisLAN").remove()
        // d3.selectAll(".annotation-group").remove()

          circles.filter(d=>d.category==="#ccc")
          .transition()
          .duration(750)
          .ease(d3.easeLinear)
          .attr("cx", 200)
          .attr("cy", 1000)
          // .attr("opacity", 0)

          annot.filter(d=>d.category==="#ccc")
          .transition()
          .duration(750)
          .ease(d3.easeLinear)
          .attr("x", 200)
          .attr("y", 1000)
          // .attr("opacity", 0)
          
          circles.filter(d=>d.category!=="#ccc")
          .transition()
          .duration(750)
          .ease(d3.easeLinear)
          .attr("cx", d => normScaleXInc(d[xVar]))
          .attr("cy", d=> normScaleYMob(d[yVar]))
          .attr("opacity", d=>d.income!==null?1:0)
          .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
          .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
      
          annot.filter(d=>d.category!=="#ccc")
          .transition()
          .duration(750)
          .ease(d3.easeLinear)
          .attr("x", d => normScaleXInc(d[xVar]))
          .attr("y", d=> normScaleYMob(d[yVar]))
          .attr("opacity", d=>d.income!==null?1:0)
          .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)

          } else if (hexmap && circles && annot && selectedView.value==="map") {

            circles
            .transition()
            .duration(750)
            .ease(d3.easeLinear)
            .attr("cx", d=>d.x)
            .attr("cy", d=>d.y)
            .attr("opacity", 1)
            .attr("fill", d => colorBivar([d.mobilityWork, d.income]))
            .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)

        
            annot
            .transition()
            .duration(750)
            .ease(d3.easeLinear)
            .attr("x", d=>d.x)
            .attr("y", d=>d.y)
            .attr("opacity", 1)
            .attr("transform",selectedView.value==="map"?`translate(${margin.left*2},0)`:`translate(0,0)`)

          } else if (hexmap && circles && annot && selectedView.value==="bars") {

            circles.filter(d=>d.category==="#ccc")
            .transition()
            .duration(750)
            .ease(d3.easeLinear)
            .attr("cx", 200)
            .attr("cy", 1000)
            // .attr("opacity", 0)

            annot.filter(d=>d.category==="#ccc")
            .transition()
            .duration(750)
            .ease(d3.easeLinear)
            .attr("x", 200)
            .attr("y", 1000)
            // .attr("opacity", 0)
            
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
            .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
        
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
            .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)

          } else if (hexmap && circles && annot && selectedView.value==="barsUrban") {

          circles.filter(d=>d.urbCategory===null||d.category==="#ccc")
          .transition()
          .duration(750)
          .ease(d3.easeLinear)
          .attr("cx", 200)
          .attr("cy", 1000)
          .attr("opacity", 0)
    
          annot.filter(d=>d.urbCategory===null||d.category==="#ccc")
          .transition()
          .duration(750)
          .ease(d3.easeLinear)
          .attr("x", 200)
          .attr("y", 1000)
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
          .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)
      
          annot.filter(d=>d.urbCategory!==null&&d.category!=="#ccc")
          .transition()
          .delay((d, i) => {
            return i * Math.random() * 1.5;
            })
          .duration(800)
          .attr("x", d => normScaleCatXUrb(d.urbCategory)+d.paddingUrb.x)
          .attr("y", d=> normScaleUrbRow(d.urbRow+d.paddingUrb.y))
          .attr("opacity", d=>d.income!==null?1:0)
          .attr("transform",selectedView.value==="map"?`translate(${-margin.left},0)`:`translate(0,0)`)

        } 


        function getPadding(x, catRow, urbRow, urbCat) {


            let padding={x:0, y:0}
            if (x === "category") {

              // console.log("showing", x, "row", catRow)

              let rowCheck = d3.range(1,3,1).map(d=>{ return {
                key:[`num${d+1}`],
                val:d3.range(d, 500, 3)
              }})
              .reduce((map, obj) => (map[obj.key] = obj.val, map), {})
              
              console.log("rowcheck", rowCheck)

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

              console.log("rowcheck", rowCheck)

              if (urbCat==="Urban") {

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
                
              } else if (urbCat==="Rural") {

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

            } 
            // else {

            //   padding = {x:0, y:0}
            // }

            console.log("padding is", padding)

            return padding
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
        } else if (annot && innerWidth>550) {
          annot.raise()
          // annot.attr('font-size', fontSize*2)
        }

</script>
<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight /> <!--on:resize='{resize}' -->
<!-- dropdowns -->
    <!-- <select id="chartView" bind:value={selectedView} on:change="{(e)=>updateScales(selectedView.value)}"> -->
  <!-- <select id="chartView" bind:value={selectedView} on:change="{(e)=>updateScales(selectedView.value)}"> -->
  <select id="chartView" bind:value={selectedView}>
		{#each viewOptions as option}
			<option value={option}>
				{option.text}
			</option>
		{/each}
	</select>	
    <!-- <p>selected {selectedCountry ? selectedCountry.value : '[waiting...]'}</p>
    <p>selected {selectedView ? selectedView.value : '[waiting...]'}</p> -->
<!-- </div> -->
<br>
<div id="staticTooltip"></div>
<div display=flex justify-content=center>
  <svg width={100} height={100} display=block margin=auto transform="translate({innerWidth/2-50},{0})">
    <g class="legend" transform="translate({15},{15})">  <!-- transform="translate({innerWidth/2},{margin.top/2})"-->
        <marker id="arrow" markerHeight=10 markerWidth=10 refX=3 refY=3 orient=auto>
            <path d="M0,0L6,3L0,6Z" />
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
                    {dataBivar.title[0]}{labels[j] && ` (${labels[j]})`}
                    {dataBivar.title[1]}{labels[i] && ` (${labels[i]})`}
                </title>
            </circle>
        {/each}
        <line marker-end="url(#arrow)" x1=0 x2={n * k} y1={n * k} y2={n * k} stroke=black stroke-width=1.5 />
        <line marker-end="url(#arrow)" y2=0 y1={n * k} stroke=black stroke-width=1.5 />
        <text font-weight="bold" dy="0.71em" transform="rotate(90) translate({n / 2 * k},6)" text-anchor="middle">{dataBivar.title[0]}</text>
        <text font-weight="bold" dy="0.71em" transform="translate({n / 2 * k},{n * k + 6})" text-anchor="middle">{dataBivar.title[1]}</text>
    </g>
  </svg>
</div>
<svg viewBox="0 0 800 600" bind:this={svg}></svg>
<!-- <div id="chart" style="text-align:center" class="svg-container" bind:clientWidth={figWidth}> bind:clientWidth={figWidth} -->
    <!-- <svg width={figWidth} height={figWidth} transform="translate({0},{-margin.bottom/2})"> -->
        <!-- <g class="vizElement">
                {#each hexesClean as d, i}
                    <g>
                        <circle
                        class="laCircle"
                        cursor="pointer"
                        cx={normScaleXhex(d.x)}
                        cy={normScaleYhex(d.y)}
                        r={innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius}
                        stroke="#fffae7"
                        stroke-width=0.5
                        fill={colorBivar([d.mobilityWork, d.income])}
                        category={d.category}
                        ></circle>
                    </g>
                    <g>
                        <text
                        class="LStextUK"
                        cursor="pointer"
                        font-size={innerWidth>600?fontSize:fontSize*0.8}
                        text-anchor="middle"
                        x={normScaleXhex(d.x)}
                        y={normScaleYhex(d.y)}
                        fill="rgb(255,255,255)"
                        stroke-width=0.5
                        >{innerWidth>500?d.n.slice(0,3):""}
                        </text>
                    </g>
                {/each} -->
                <!-- {#each $tweenedData as d}
                    <g>
                        <circle
                        class="laCircle"
                        cursor="pointer"
                        cx={xScale(d.xVar)+d.padding.x}
                        cy={yScale(d.yVar+d.padding.y)}
                        r={innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.88*radius}
                        stroke="#fffae7"
                        stroke-width=0.5
                        fill={colorBivar([d.mobilityWork, d.income])}
                        on:mouseover={(event)=> {
                            console.log(d)
                        }}
                        
                        ></circle>

                    </g>
                    <g>
                        <text
                        class="LStextUK"
                        cursor="pointer"
                        font-size={innerWidth>600?fontSize:fontSize*0.8}
                        text-anchor="middle"
                        x={xScale(d.xVar)+d.padding.x}
                        y={yScale(d.yVar+d.padding.y)}
                        fill="rgb(255,255,255)"
                        stroke-width=0.5
                        >{innerWidth>550?d.n.slice(0,3):""}
                        </text>
                    </g>
                {/each} -->

                                    <!-- in:fly="{{x:scaleX(xAccessor(d)), y: scaleY(yAccessor(d)), opacity:1, duration: 500, delay: Math.random()*i}}"
                        out:fly="{{x:200, y:200, opacity:0, duration: 500, delay: Math.random()*i}}"
                        in:transition={{ duration: 1000 }} -->

            <!-- {:else if selectedView.value==="chart"}
                {#each hexes as d, i}
                <g>
                    <circle
                    class="laCircle"
                    cursor="pointer"
                    cx={xScaleInc(+d[xVar])} 
                    cy={yScaleMob(+d[yVar])}
                    r={innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius}
                    stroke="#fffae7"
                    stroke-width=0.5
                    fill={colorBivar([d.mobilityWork, d.income])}
                    ></circle>
                </g> -->
                <!-- <g>
                    <text
                    class="LStextUK"
                    cursor="pointer"
                    font-size={innerWidth>600?fontSize:fontSize*0.8}
                    text-anchor="middle"
                    transition:fly="{{ x: xScaleInc(d[xVar]), y: yScaleMob(d[yVar]), duration: 2000 }}"
                    fill="rgb(255,255,255)"
                    stroke-width=0.5
                    >{innerWidth>500?d.n.slice(0,3):""}
                    </text>
                </g> -->
                <!-- {/each} -->
            <!-- {/if} -->
        <!-- </g>-->
<!-- </div> -->
<p>Outerwidth: {outerWidth}</p>
<p>Innerwidth: {innerWidth}</p>
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

/* .vizElement {
  max-width:400px;
} */

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