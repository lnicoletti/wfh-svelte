<script>
    import * as d3 from "d3";
    // import {fade, blur, fly, slide, scale, draw} from "svelte/transition";
    import { tweened } from "svelte/motion";
    import * as easings from 'svelte/easing';
    import {renderHexJSON} from "d3-hexjson"
	import {annotationLabel} from "d3-svg-annotation"
    import {onlyUnique} from "../utils.js"

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
    // let selectedView;
    
    $: marginTop = 5
    $: marginBottom = 40
    $: marginLeft = 100
    // const figScale = 2.1
    // const figWidth = width
    // const figHeight = height*figScale
    let svg;
    let width = 1850
    let height = width/2.9
    // responsive margins
    let margin = ({ top: 0.03*height, right: 0.04*width, bottom: 0.03*height, left: 0.04*width})
    let figWidth = width - margin.left - margin.right
    let figHeight = height - margin.top - margin.bottom

    $: scatterWidth = figWidth-100
    $: radius = figWidth/62//9.8
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
    let hexes = renderHexJSON(hex_la, figWidth, figHeight).map(d=> ({ ...d, 
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
        
        console.log("urb",ukUrbRural)
        console.log("hex", hexes)

        // variables for dot clusters bars
        let clusterData = d3.groups(hexes, v=>v.category).map(d=> { return {category: d[0], data: d[1].map((c, i) =>({ ...c, row: i}))}}).filter(d=>d.category!=="#ccc")
        console.log("cluster", clusterData)
        let scaleXCategory = d3.scaleBand().domain(categoriesX).range([margin.left, scatterWidth - margin.right]).paddingInner([0.4]);  
        let scaleXCategoryLabels = d3.scaleBand().domain(categoryLabels).range([margin.left, scatterWidth - margin.right]).paddingInner([0.4]);     
        let scaleY = d3.scaleLinear().domain([0, 85]).range([figHeight- margin.bottom, 0]); 

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

        console.log("urb rural", ruralData)
        let urbCategoriesX = ruralData.sort((a,b)=>b.data.length-a.data.length).map(d=>d.urbCategory)                               
        let scaleXurbCategory = d3.scaleBand().domain(urbCategoriesX).range([margin.left, scatterWidth - margin.right]).paddingInner([0.4]);  
        let scaleYurb = d3.scaleLinear().domain([0, 285]).range([figHeight-margin.bottom, 0]); 


        // normalize each variable to prevent svelte transition bug when scales change
        let normScaleXInc = d3.scaleLinear()
                        .domain(d3.extent(hexes, d => d.income))
                        .range([margin.left, figWidth - margin.right])
      
        let normScaleYMob = d3.scaleLinear()
                        .domain(d3.extent(hexes, d => d.mobilityWork))
                        .range([margin.top, figWidth - margin.bottom])

        let normScaleXhex = d3.scaleLinear()
                        .domain(d3.extent(hexes, d => d.x))
                        .range([margin.left, figWidth - margin.right])
      
        let normScaleYhex = d3.scaleLinear()
                        .domain(d3.extent(hexes, d => d.y))
                        .range([figWidth - margin.bottom, margin.top])

        let normScaleCatRow = d3.scaleLinear()
                        .domain([85, 0])
                        .range([figWidth - margin.bottom, margin.top])

        let normScaleCatX = d3.scaleBand()
                        .domain(categoriesX)
                        .range([margin.left, figWidth - margin.right])


        let normScaleUrbRow = d3.scaleLinear()
                        .domain([285, 0])
                        .range([figWidth - margin.bottom, margin.top])

        let normScaleCatXUrb = d3.scaleBand()
                        .domain(urbCategoriesX)
                        .range([margin.left, figWidth - margin.right])



        let hexesNorm = hexes.map(d=>({...d, mobilityWorkNorm: normScaleYMob(d.mobilityWork),
                                             incomeNorm: normScaleXInc(d.income),
                                             xNorm: normScaleXhex(d.x),
                                             yNorm: normScaleYhex(d.y),
                                             rowNorm: d.category!=="#ccc"? normScaleCatRow(clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.key===d.key)[0].row):1720,
                                             categoryNorm: normScaleCatX(d.category),
                                             urbRowNorm: d.urbCategory!==null && 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0]!==undefined? 
                                                         normScaleUrbRow(ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0].row):1500,
                                            // urbRowNorm: ruralData.filter(c=>c.urbCategory===d.urbCategory)[0],//.data.filter(e=>e.key===d.key)[0]),
                                             urbCategoryNorm: normScaleCatXUrb(d.urbCategory),
                                             urbRow: d.urbCategory!==null && 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)!==undefined &&
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0]!==undefined? 
                                                         ruralData.filter(c=>c.urbCategory===d.urbCategory)[0].data.filter(e=>e.LAD11CD===d.key)[0].row:1500,
                                             catRow: d.category!=="#ccc"? clusterData.filter(c=>c.category===d.category)[0].data.filter(e=>e.key===d.key)[0].row:1720,

                                            }))

        console.log("normalized", hexesNorm)

        // ////////////////////

        let xShow = "xNorm";
        let yShow = "yNorm";
        let padding={x:0, y:0}

        let dataUpdated = hexesNorm.map((d, index) => ({
            xVar: +d[xShow],
            yVar: +d[yShow],
            income:d.income,
            mobilityWork:d.mobilityWork,
            n:d.n,
            category: d.category,
            urbCategory: d.urbCategory,
            catRow: d.catRow,
            urbRow: d.urbRow,
            padding: padding
            // padding: getPadding(xShow, d.row)
            // paddingmobilityWorkNorm: 0,
            // paddingxNorm: 0,
            // paddingcategoryNormNorm: 100,
            // paddingUrbcategoryNormNorm: -100,

        }));


        function getPadding(x, catRow, urbRow) {


            if (x === "categoryNorm") {

              console.log("showing", x, "row", catRow)

              let rowCheck = d3.range(1,3,1).map(d=>{ return {
                key:[`num${d+1}`],
                val:d3.range(d, 500, 3)
              }})
              .reduce((map, obj) => (map[obj.key] = obj.val, map), {})
              
              console.log("rowcheck", rowCheck)

              rowCheck["num3"].includes(catRow)?padding={x:40, y:normScaleCatRow(0)}:
              rowCheck["num2"].includes(catRow)?padding={x:20, y:normScaleCatRow(1)}:
              padding={x:0, y:normScaleCatRow(2)}

            } else if (x === "urbCategoryNorm") {

              console.log("showing", x, "urbRow", urbRow)

              let rowCheck = d3.range(1,10,1).map(d=>{ return {
                key:[`num${d+1}`],
                val:d3.range(d, 1000, 10)
              }})
              .reduce((map, obj) => (map[obj.key] = obj.val, map), {})

              console.log("rowcheck", rowCheck)

              rowCheck["num10"].includes(urbRow)?padding={x:180, y:normScaleUrbRow(0)}:
              rowCheck["num9"].includes(urbRow)?padding={x:160, y:normScaleUrbRow(1)}:
              rowCheck["num8"].includes(urbRow)?padding={x:140, y:normScaleUrbRow(2)}:
              rowCheck["num7"].includes(urbRow)?padding={x:120, y:normScaleUrbRow(3)}:
              rowCheck["num6"].includes(urbRow)?padding={x:100, y:normScaleUrbRow(4)}:
              rowCheck["num5"].includes(urbRow)?padding={x:80, y:normScaleUrbRow(5)}:
              rowCheck["num4"].includes(urbRow)?padding={x:60, y:normScaleUrbRow(6)}:
              rowCheck["num3"].includes(urbRow)?padding={x:40, y:normScaleUrbRow(7)}:
              rowCheck["num2"].includes(urbRow)?padding={x:20, y:normScaleUrbRow(8)}:
              padding={x:0, y:normScaleUrbRow(9)}

            } else {

              padding = {x:0, y:normScaleCatRow(0)}
            }

            console.log("padding is", padding)

            return padding
        }

        console.log("dataUpdated", dataUpdated)//.map(d=>d.category))

        $: tweenedData = tweened(dataUpdated, {
            delay: 0,
            duration: 800,
            easing: easings.cubicOut
          });


        function setTween(x, y) {
          tweenedData.set(hexesNorm.map((d, index) => ({
              xVar: +d[x],
              yVar: +d[y],
              income:d.income,
              mobilityWork:d.mobilityWork,
              n:d.n,
              category: d.category,
              urbCategory: d.urbCategory,
              padding: getPadding(x, d.catRow, d.urbRow)
              // padding: +d.padding
            }))
          );
        }

       function updateScales(view) {
            if (view==="chart") {
                // console.log("new view", selectedView.value)
                xShow = xVar + "Norm";
                yShow = yVar + "Norm";
                setTween(xShow, yShow, view)
                console.log($tweenedData)

            } else if (view==="map") {
                console.log("new view:", selectedView.value)
                // console.log("before",$tweenedData)
                xShow = "xNorm";
                yShow = "yNorm";
                setTween(xShow, yShow)
                console.log("after",$tweenedData)

            } else if (view==="bars") {
                console.log("new view:", selectedView.value)
                // console.log("before",$tweenedData)
                xShow = "categoryNorm";
                yShow = "rowNorm";
                setTween(xShow, yShow)
                console.log("after",$tweenedData)

            } else if (view==="barsUrban") {
                console.log("new view:", selectedView.value)
                // console.log("before",$tweenedData)
                xShow = "urbCategoryNorm";
                yShow = "urbRowNorm";
                setTween(xShow, yShow)
                console.log("after",$tweenedData)
            }
        }

        $: extentX = d3.extent($tweenedData, (d) => d.xVar)
        $: extentY = d3.extent($tweenedData, (d) => d.yVar)

        $: console.log(extentY)

      //   let rangeX;
      //   // function getRangeX(view) {

      //     if (view === "map") {
      //       rangeX = innerWidth>520? [margin.left, figWidth - margin.right]: innerWidth<300? [margin.left/1.7, figWidth - margin.right/1.7]:[margin.left/1.2, figWidth - margin.right/1.2]
          
      //     } else if (view === "chart"||view === "bars"||view === "barsUrban") {
      //       rangeX = innerWidth>520? [margin.left/3, figWidth - margin.right]: innerWidth<300? [margin.left/1.7/3, figWidth - margin.right/1.7]:[margin.left/1.2/3, figWidth - margin.right/1.2]

      //   }
      // //   return rangeX
      // // }

      // $: console.log("VIEW", selectedView.value)

      $: rangeX = selectedView.value === "map"? innerWidth>520? [margin.left, figWidth - margin.right]: innerWidth<300? [margin.left/1.7, figWidth - margin.right/1.7]:[margin.left/1.2, figWidth - margin.right/1.2]:
                                               innerWidth>520? [margin.left/3, figWidth - margin.right]: innerWidth<300? [margin.left/1.7/3, figWidth - margin.right/1.7]:[margin.left/1.2/3, figWidth - margin.right/1.2]


        $: xScale = //selectedView.value!=="bars"?
              d3.scaleLinear()
                        .domain(extentX)
                        // .domain(selectedView.value==="map"? [20.17923270954032, 474.21196867419746]:[0, 66825]).nice()
                        // .range([0, figWidth - margin.right-margin.left])
                        .range(rangeX)
                        // .range(innerWidth>520? [margin.left, figWidth - margin.right]: innerWidth<300? [margin.left/1.7, figWidth - margin.right/1.7]:[margin.left/1.2, figWidth - margin.right/1.2])
              //           :

              // d3.scaleBand()
              //           .domain(extentX)
              //           // .domain(selectedView.value==="map"? [20.17923270954032, 474.21196867419746]:[0, 66825]).nice()
              //           // .range([0, figWidth - margin.right-margin.left])
              //           .range(innerWidth>520? [margin.left, figWidth - margin.right]: innerWidth<300? [margin.left/1.7, figWidth - margin.right/1.7]:[margin.left/1.2, figWidth - margin.right/1.2])
      

        $: yScale = d3.scaleLinear()
                        .domain(extentY)
                        // .domain(selectedView.value==="map"? [11.650485436893204, 588.3495145631067]:[0, -60]).nice()
                        // .range([0, figWidth - margin.right-margin.left].reverse())
                        .range([figWidth - margin.bottom, margin.top])


        $: console.log("tweened", tweenedData)



</script>
<svelte:window bind:innerWidth bind:outerWidth bind:innerHeight bind:outerHeight/>
<!-- dropdowns -->
<div style="text-align:center" class="custom-select">
    <span class="mapCredit">SELECT COUNTRY AND VIEW</span><br>
    <select id="chartCountry" bind:value={selectedCountry}>
		{#each countryOptions as option}
			<option value={option}>
				{option.text}
			</option>
		{/each}
	</select>
    <!-- <select id="chartView" bind:value={selectedView} on:change="{(e)=>updateScales(selectedView.value)}"> -->
  <select id="chartView" bind:value={selectedView} on:change="{(e)=>updateScales(selectedView.value)}">
		{#each viewOptions as option}
			<option value={option}>
				{option.text}
			</option>
		{/each}
	</select>	
    <!-- <p>selected {selectedCountry ? selectedCountry.value : '[waiting...]'}</p>
    <p>selected {selectedView ? selectedView.value : '[waiting...]'}</p> -->
</div>
<br>
<div id="staticTooltip"></div>
<div id="chart" style="text-align:center" class="svg-container" bind:clientWidth={figWidth}> <!-- bind:clientWidth={figWidth} -->
    <svg width={figWidth} height={figWidth} transform="translate({0},{-margin.bottom/2})">
        <g class="vizElement">
            <!-- {#if selectedView.value==="map"} -->
                <!-- {#each hexes as d, i}
                    <g>
                        <circle
                        class="laCircle"
                        cursor="pointer"
                        cx={scaleX(xAccessor(d))}
                        cy={scaleY(yAccessor(d))}
                        r={innerWidth>600?radius:innerWidth>500?0.95*radius:innerWidth>450?0.9*radius:0.85*radius}
                        stroke="#fffae7"
                        stroke-width=0.5
                        fill={colorBivar([d.mobilityWork, d.income])}
                        
                        ></circle>
                    </g>
                    <g>
                        <text
                        class="LStextUK"
                        cursor="pointer"
                        font-size={innerWidth>600?fontSize:fontSize*0.8}
                        text-anchor="middle"
                        x={scaleX(xAccessor(d))}
                        y={scaleY(yAccessor(d))}
                        fill="rgb(255,255,255)"
                        stroke-width=0.5
                        >{innerWidth>500?d.n.slice(0,3):""}
                        </text>
                    </g>
                {/each} -->
                {#each $tweenedData as d}
                    <g>
                        <!-- svelte-ignore a11y-mouse-events-have-key-events -->
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
                        <!-- opacity={d.xVar===NaN||d.yVar===NaN?0:1}-->
                        <!-- cx={d.xVar===NaN||d.yVar===NaN?Math.random()*-200:xScale(d.xVar)}
                        cy={d.xVar===NaN||d.yVar===NaN?Math.random()*-200:yScale(d.yVar)} -->
                        <!--  fill={colorBivar([d.data.mobilityWork, d.data.income])} -->
                        <!-- cx={xScale(d.xVar)}
                        cy={yScale(d.yVar)} -->
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
                {/each}

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
        </g>
        <g class="legend" transform="translate({innerWidth>500? figWidth*0.85: figWidth*0.72},{margin.top})">
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