<script>
import * as d3 from 'd3';
let arcGenerator = d3.arc().innerRadius(30).outerRadius(50);
let arc = arcGenerator({
	startAngle: 0,
	endAngle: 2 * Math.PI
});
export let data = [];


let sliceGenerator = d3.pie().value(d => d.value);
let arcData = sliceGenerator(data);
let arcs = arcData.map(d => arcGenerator(d));
let colors = d3.scaleOrdinal(d3.schemeTableau10);



</script> 


<div class="container">

<svg viewBox="-50 -50 100 100">
	<path d={arc} fill="red" />
    {#each arcs as arc, index}
  <path d={ arc } fill={ colors(index) } />
{/each}

</svg>

<ul class="legend">
	{#each data as d, index}
		<li style="--color: { colors(index) }">
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
</style>