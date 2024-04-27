<script>
    import { geoMercator, geoAlbersUsa, geoPath} from 'd3';
    import geojson from './new_taxi_data.json'
    
    let width, height;

    $: projection = geoAlbersUsa().fitSize([width, height], geojson);

    $: pathGenerator = geoPath(projection);

    let features = [];

    $: if (geojson) features = geojson.features.map(feature => {
        return {
            ...feature,
            id: feature.properties.id,
            destination_id: feature.properties.destination_id,
            path: pathGenerator(feature)
        };
    });

    let hoveredZoneId = null;
    let selectedZoneId = null;
    let selectedDestinationId = null;
    
    function handleMouseEnter(id) {
        console.log('Mouse enter:', id);
        hoveredZoneId = id;
    }

    function handleClick(id, destination_id) {
        console.log('Feature selected:', id);
        selectedZoneId = id;
        selectedDestinationId = destination_id;
    }

</script>

<main
    bind:clientWidth={width}
    bind:clientHeight={height}
    >
    <div>
        {#if selectedZoneId !== null}
            {#each features as feature}
                {#if feature.id === selectedZoneId}
                    <h1>You selected: {feature.properties.zone}</h1>
                    <h3>The most popular destination is: {feature.properties.destination_zone}</h3>
                    <br>
                    <p>Based on For-Hire Vehicle Trip Records data from August 2022.</p>
                    <p>If the most popular destination was the originating zone, the next most popular one was chosen.</p>
                {/if}
            {/each}
        {/if}
    </div>
        <svg
            width={width}
            height={height}
            >
            {#each features as { destination_id, id, path }}
                <!-- svelte-ignore a11y-no-static-element-interactions -->
                <!-- svelte-ignore a11y-click-events-have-key-events -->
                <path
                    d={path}
                    class:active={hoveredZoneId === id}
                    class:selected={selectedZoneId === id}
                    class:related={selectedDestinationId === id}
                    on:mouseenter={() => handleMouseEnter(id)}
                    on:click={() => handleClick(id, destination_id)}
                />
            {/each}
        </svg>
</main>

<style>
    main {
        width: 100vw;
        height: 100vh;
        overflow: hidden;
    }

    path {
        fill: darkgray;
        stroke: white;
        opacity: 0.5;
        transition: opacity 0.4s ease-out;

    }

    path.active {
        fill: darkorange;
        opacity: 0.8;
    }

    path.selected {
        opacity: 1.0;
        fill: darkorange;
        stroke: black;
        stroke-width: 2px;
    }

    path.related {
        opacity: 1;
        fill: darkgreen;
        stroke: black;
        stroke-width: 2px;
    }
    
</style>
