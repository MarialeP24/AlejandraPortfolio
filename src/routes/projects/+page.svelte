<script>
 import * as d3 from 'd3';
  import Pie from '$lib/Pie.svelte';
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";

  let rolledData = d3.rollups(projects, v => v.length, d => d.year);
  let pieData = rolledData.map(([year, count]) => {
    return { value: count, label: year };
  });
  let query = "";
$: filteredProjects = projects.filter(project => {
    if (query) {
      return project.title.includes(query);
    }
    return true;
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
