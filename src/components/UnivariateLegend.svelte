<script>
    import {range, extent, min, scaleQuantize, schemeGreens, schemeOranges} from "d3";
    export let country;
    export let hexesClean;
    export let numTicks;
    // export let margin;
    export let colorVar;

    let width = 240
    let margin = ({ right: 20, left: 20, top:8})

    let legendWidth = width - margin.right - margin.left
    let legendSvgHeight = 45
    let extentIncome = country==="UK"?extent(hexesClean, d => d.income):[min(hexesClean, d => d.income), 80000]
    let extentMob = country==="UK"?extent(hexesClean, d => d.mobilityWork):[-40, -5]
     // color scales
    let legendColor = colorVar==="income"?
                scaleQuantize()
                        .domain(extentIncome)
                        .range(schemeGreens[9]).nice()
    :
                scaleQuantize()
                        .domain(extentMob)
                        .range(schemeOranges[9]).nice()
                        

    let legendHeight = 15
    let legendTickWidth = legendWidth/numTicks
    let interval = colorVar==="income"?(extentIncome[1] - extentIncome[0])/numTicks:(extentMob[1] - extentMob[0])/numTicks

    // need to keep the legend static if we dont show numbers anyways
    let legendTicks = colorVar==="income"? range(extentIncome[0],extentIncome[1],interval): range(extentMob[0],extentMob[1],interval)//.reverse()

    console.log("legend data", legendTicks)
</script>

<!-- <div class="legendContainer"> -->
    <svg width={width} height={legendSvgHeight} display=block margin=auto > 
    <!-- transform={`translate(${margin.left}, 0)`} -->
    <g transform={`translate(${margin.left}, ${margin.top})`}>
    {#each legendTicks as data, i}
        <rect
        transform={`translate(0,5)`}
        width={legendTickWidth}
        height={legendHeight}
        fill={legendColor(data)}
        x={i*legendTickWidth}
        y=0
        >
        </rect>
        <!-- stroke="#4d004b"
        stroke-width=1.8px -->
        <line
        transform={`translate(0,0)`}
        stroke="#fafafa"
        stroke-width=1
        x1={i*legendTickWidth}
        y1=0
        x2={i*legendTickWidth}
        y2={legendHeight*1.3}
        >
        </line>
    <!-- {#each legendTicks as data, i}
        <text
        transform={data!== 2 * Math.floor(maxStories/2)?`translate(20,0)`:`translate(15,0)`}
        class="legendTicks"
        id="legendLabel"
        color="black"
        x={i*legendTickWidth}
        y={legendHeight*2}
        text-anchor={data!== 2 * Math.floor(maxStories/2)?"middle":"start"}
        >
        {data!== 2 * Math.floor(maxStories/2)?data:data + " Stories"}
        </text>
    {/each} -->
        <text
        transform={`translate(10,0)`}
        class="legendTicks"
        id="legendLabel"
        x={i*legendTickWidth}
        y={legendHeight*2}
        text-anchor=middle
        >
        {colorVar==="income"?(data/1000).toFixed()+"k":i===0?"More Travel":i===numTicks-1?"Less Travel":""}
        </text>

        <!-- {data===min?"middle":data===2 * Math.floor(max/2)?"start":""} -->
        <!-- {data===min?"Less Stories":data===2 * Math.floor(max/2)?"More Stories":""} -->

        <!-- data.toFixed()+"%" -->
    {/each}
    </g>
    <g transform={`translate(${width/2}, 0)`}>
        <text
        class="legendTicks"
        x={0}
        y={margin.top*0.8}
        text-anchor=middle
        >
        {colorVar==="income"?"Household income": "Travel to workplaces"}
        </text>
    </g>
    </svg>
<!-- </div> -->

<style>

.legendTicks {
        font-family: 'Source', sans-serif;
        font-size: 8px;
        font-weight: 700;
        fill: #445312;
		text-transform: uppercase;
    }

</style>