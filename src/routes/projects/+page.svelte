<script>
 import * as d3 from 'd3';
  import Pie from '$lib/Pie.svelte';
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";

 // Make sure the variable definition is *outside* the block
let pieData;

$: {
// Initialize to an empty object every time this runs
    pieData = {};
    
// Calculate rolledData and pieData based on filteredProjects here
    let rolledData = d3.rollups(filteredProjects, v => v.length, d => d.year);

// We don't need 'let' anymore since we already defined pieData
    pieData = rolledData.map(([year, count]) => {
        return { value: count, label: year };
    });
}

  let query = "";
  $: filteredProjects = projects.filter(project => {
	let values = Object.values(project).join("\n").toLowerCase();
	return values.includes(query.toLowerCase());
});


</script>


<svelte:head>
  <title>My projects</title>
</svelte:head>

<h1>PROJECTS { projects.length }</h1>


<Pie data={pieData} />
<input type="search" bind:value={query}
       aria-label="Search projects" placeholder="🔍 Search projects…" />
    


<div class="projects">
    {#each filteredProjects as p}
    <Project data={p} />
    {/each}
</div>

<style>
    input[type="search"] {
    width: 100%;
    box-sizing: border-box; /* Ensures padding is included in the width */
    padding: 0.3em;
    margin: 0.5em 0;
  }
</style>
