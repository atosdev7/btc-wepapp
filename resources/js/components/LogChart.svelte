<script>
    import * as d3 from 'd3';

    // Receive plot data as prop.
    export let data;

    // The chart dimensions and margins as optional props.
    export let width = 928;
    export let height = 400;
    export let marginTop = 20;
    export let marginRight = 30;
    export let marginBottom = 30;
    export let marginLeft = 40;

    // Create the x (horizontal position) scale.
    const xScale = d3.scaleUtc(
        d3.extent(data, (d) => new Date(d.time)),
        [marginLeft, width - marginRight],
    );

    // Create the y (vertical position) scale.
    let yEnd = d3.max(data, (d) => d.current_temp);
    
    const yScale = d3.scaleLinear([0, (yEnd? yEnd : 60) + 20], [height - marginBottom, marginTop]);

    // Create the line generator.
    const lineTemp = d3
        .line()
        .x((d) => xScale(new Date(d.time)))
        .y((d) => yScale(d.current_temp));
    // Create the line generator.
    const lineSol = d3
        .line()
        .x((d) => xScale(new Date(d.time)))
        .y((d) => yScale(d.solenoid * 10));        
</script>

<svg {width} {height} viewBox="0 0 {width} {height}" style:max-width="100%" style:height="auto">
    <!-- X-Axis -->
    <g transform="translate(0,{height - marginBottom})">
        <line stroke="currentColor" x1={marginLeft - 6} x2={width} />

        {#each xScale.ticks(24) as tick}
            <!-- X-Axis Ticks -->
            <line stroke="currentColor" x1={xScale(tick)} x2={xScale(tick)} y1={0} y2={6} />

            <!-- X-Axis Tick Labels -->
            <text fill="currentColor" text-anchor="middle" x={xScale(tick)} y={22}>
                {tick.getHours()}:{tick.getMinutes()}
            </text>
        {/each}
    </g>

    <!-- Y-Axis and Grid Lines -->
    <g transform="translate({marginLeft},0)">
        {#each yScale.ticks() as tick}
            {#if tick !== 0}
                <!-- 
            Grid Lines. 
            Note: First line is skipped since the x-axis is already present at 0. 
          -->
                <line stroke="currentColor" stroke-opacity="0.1" x1={0} x2={width - marginLeft} y1={yScale(tick)} y2={yScale(tick)} />

                <!-- 
            Y-Axis Ticks. 
            Note: First tick is skipped since the x-axis already acts as a tick. 
          -->
                <line stroke="currentColor" x1={0} x2={-6} y1={yScale(tick)} y2={yScale(tick)} />
            {/if}

            <!-- Y-Axis Tick Labels -->
            <text fill="currentColor" text-anchor="end" dominant-baseline="middle" x={-9} y={yScale(tick)}>
                {tick}
            </text>
        {/each}

        <!-- Y-Axis Label -->
        <text fill="currentColor" text-anchor="start" x={-marginLeft} y={15}> Temp (°F) </text>
    </g>

    <path fill="none" stroke="green" stroke-width="2" d={lineTemp(data)} />
    <path fill="none" stroke="red" stroke-width="2" d={lineSol(data)} />
</svg>
