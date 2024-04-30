<script>
    import { geoAlbersUsa, geoPath, scaleSequential, interpolateReds} from 'd3';
    import geojson from './tax-zones-with-percent.json'
    
    let width, height;

    // Define your color scale
    const colorScale = scaleSequential(interpolateReds)
    .domain([0, 4]);

    $: projection = geoAlbersUsa().fitSize([width, height], geojson);
    $: pathGenerator = geoPath(projection);

    

    let features = [];

    $: if (geojson) features = geojson.features.map(feature => {
        return {
            ...feature,
            id: feature.properties.id,
            path: pathGenerator(feature),
            color: colorScale(feature.properties.Percent) 
        };
    });

    let tooltip = { visible: false, x: 0, y: 0, text: '' };

    let hoveredZoneId = null;
    let selectedZoneId = null;
    let svgElement;
    
    function handleMouseEnter(feature, event) {
        const svgPoint = svgElement.createSVGPoint();
        svgPoint.x = event.clientX;
        svgPoint.y = event.clientY;
        const cursorPoint = svgPoint.matrixTransform(svgElement.getScreenCTM().inverse());

        // Adjusting tooltip offsets here
        const offsetX = 5;  // Horizontal offset from the cursor
        const offsetY = 40; // Vertical offset from the cursor, negative to go above the cursor

        tooltip = {
            visible: true,
            x: cursorPoint.x + offsetX,
            y: cursorPoint.y + offsetY,
            text: `${feature.properties.zone}: ${(feature.properties.Percent).toFixed(2)}%`
        };
    }

    function handleMouseLeave() {
        tooltip.visible = false;
    }

    function handleClick(id) {
        console.log('Feature selected:', id);
        selectedZoneId = id;
    }

    // Legend settings
    const legendWidth = 300;
    const legendHeight = 30;
    const numberOfSegments = 10;
    const legendColors = Array.from({length: numberOfSegments}, (v, i) => colorScale(i / numberOfSegments * 4));

</script>

<main
    bind:clientWidth={width}
    bind:clientHeight={height}
    >
    <div id="title" style="padding-top: 10px; padding-left: 20px;">
        <h1>Distribution of Uber/Lyft rides from LaGuardia</h1>
        <h2>Hover over a zone to see the percentage of rides that end there.</h2>
        <p>Data provided by <a href="https://data.cityofnewyork.us/Transportation/2022-High-Volume-FHV-Trip-Records/g6pj-fsah/about_data">Taxi and Limousine Commission (TLC)</a>.</p>
        <p>Data based on destination zone listed for all trips originating from LaGuardia Airport in 2022.</p>
    </div>
        <svg
            bind:this={svgElement}
            width={width}
            height={height}
            >
            {#each features as feature}
                <!-- svelte-ignore a11y-no-static-element-interactions -->
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <path
                    d={feature.path}
                    fill={feature.color}
                    class:active={hoveredZoneId === feature.id}
                    class:selected={selectedZoneId === feature.id}
                    on:mouseenter={(event) => handleMouseEnter(feature, event)}
                    on:mouseleave={handleMouseLeave}
                    on:click={() => handleClick(feature.id)}
            />
            {/each}
        </svg>
        <svg width={width} height={50} style="position: absolute; top: 120px; left: 40px;">
            {#each legendColors as color, index}
                <rect x={index * (legendWidth / numberOfSegments)} y={0} width={legendWidth / numberOfSegments} height={legendHeight} fill={color} />
                <text x={(index * (legendWidth / numberOfSegments)) + (legendWidth / numberOfSegments / 2)} y={legendHeight + 15} style="fill: black; font-size: 12px; text-anchor: middle;">
                    {((index / numberOfSegments) * 4).toFixed(1)}
                </text>
            {/each}
        </svg>
        {#if tooltip.visible}
            <div style="position: absolute; left: {tooltip.x}px; top: {tooltip.y}px; padding: 4px; background-color: white; border: 1px solid black;">
                {tooltip.text}
            </div>
        {/if}
</main>

<style>
    main {
        width: 100vw;
        height: 100vh;
        overflow: hidden;
        position: relative; /* Ensure tooltip positions correctly */
    }

    path {
        stroke: black;
        stroke-width: 1px;
        opacity: 1;
        transition: stroke-width 0.4s ease-out;

    }

    path:hover {
        stroke-width: 3px;
        stroke: black;

    }

    
</style>