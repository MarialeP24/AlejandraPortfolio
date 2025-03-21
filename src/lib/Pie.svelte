<script>
import * as d3 from 'd3';
let arcGenerator = d3.arc().innerRadius(30).outerRadius(50);
let arc = arcGenerator({
	startAngle: 0,
	endAngle: 2 * Math.PI
});
export let data = [];
export let selectedIndex = -1;



let colors = d3.scaleOrdinal(d3.schemeTableau10);
let sliceGenerator = d3.pie().value(d => d.value);

// Define arcData and arcs outside the reactive block
let arcData;
let arcs;

    $: {
		// Reactively calculate arcData and arcs the same way we did before with sliceGenerator and arcGenerator
		arcData = sliceGenerator(data);
		arcs = arcData.map(d => arcGenerator(d));
    }



</script> 




<div class="container">

<svg viewBox="-50 -50 100 100">
	<path d={arc} fill="red" />
    {#each arcs as arc, index}
    <path 
      d={arc} 
      fill={colors(index)}
      class:selected={selectedIndex === index}
      on:click={e => selectedIndex = selectedIndex === index ? -1 : index} />
  {/each}

</svg>

<ul class="legend">
    {#each data as d, index}
      <li 
        style="--color: {colors(index)}"
        class:selected={selectedIndex === index}
        on:click={() => selectedIndex = selectedIndex === index ? -1 : index}>
        <span class="swatch"></span>
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>

.container {
    display: flex;
    align-items: center; /* vertically centers both elements */
    gap: 2em; /* space between the pie chart and legend */
  }

svg {
	max-width: 20em;
	margin-block: 2em;

	/* Do not clip shapes outside the viewBox */
	overflow: visible;
}

 .legend {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(8em, 1fr));
    gap: 1em;
    border: 1px solid black;
    padding: 1em;
    margin: 1em 0;
    list-style: none;
    flex: 1; /* allows legend to occupy remaining space */
  }

  .legend li {
    display: flex;
    align-items: center; /* Vertical centering */
    gap: 0.5em; /* Smaller gap between swatch and text */
  }

  .swatch {
    display: inline-block;
    width: 1em;
    height: 1em;
    background-color: var(--color);
    border-radius: 50%; /* makes the swatch a circle */
  }

  svg:has(path:hover) path:not(:hover) {
	opacity: 10%;
}
path {
	transition: 300ms;
    cursor: pointer;
}

path:hover {
	opacity: 100% !important;
}

/* When a path is selected, make all non-selected paths 50% opacity */
svg:has(.selected) path:not(.selected) {
   opacity: 10%;
}

.selected {
	--color: oklch(60% 45% 0) !important;
	
	&:is(path) {
		fill: var(--color) !important;
	}
	
	&:is(li) {
		color: var(--color);
	}
}

ul:has(.selected) li:not(.selected) {
	color: gray;
}


</style>