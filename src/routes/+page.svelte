<script>
    let profileData = fetch("https://api.github.com/users/MarialeP24");
    import projects from "$lib/projects.json";
    import Project from "$lib/Project.svelte";
</script>

<h1> Alejandra</h1>

<svelte:head>
  <title>About me</title>
</svelte:head>
   
<p>Hello! My name is Maria Alejandra , I have a background in architecture and design, I have spent over six years bridging public policy and spatial design, focusing on community-driven and impactful projects. My work at the Ministry of Education in Peru involved leading nationwide school infrastructure initiatives, emphasizing innovative solutions for rural areas. Later, at El Equipo Mazzanti, I contributed to research-driven projects that reimagine public spaces. Now, as a Master in Design Studies candidate at Harvard GSD, I explore the intersections of policy, public space, and climate-responsive urban design, aiming to shape inclusive and resilient environments through participatory and data-driven strategies. </p>

<img src = "Images /Mota.jpeg" alt = "Merry Xmas from Mota">

{#await fetch("https://api.github.com/users/MarialeP24")}
  <p>Loading...</p>
{:then response}
  {#await response.json()}
    <p>Decoding...</p>
  {:then data}
    <section>
      <h2>My GitHub Stats</h2>
      <dl>
        <dt>Followers:</dt>
        <dd>{data.followers}</dd>
        <dt>Following:</dt>
        <dd>{data.following}</dd>
        <dt>Public Repositories:</dt>
        <dd>{data.public_repos}</dd>
      </dl>
    </section>
  {:catch error}
    <p class="error">Something went wrong: {error.message}</p>
  {/await}
{:catch error}
  <p class="error">Something went wrong: {error.message}</p>
{/await}



<h2>
    Latest Projects
</h2>

<div class="projects">
    {#each projects.slice(0,3) as p}
        <Project data={p} hLevel="3"/>
    {/each}
</div>

<style>
    dl {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 1rem;
      align-items: start;
    }
    
    /* Place each <dt> on the first row and its corresponding <dd> on the second */
    dt {
      grid-row: 1;
      font-weight: bold;
    }
    dd {
      grid-row: 2;
      margin: 0;
    }
  
    /* Additional personal styling */
    h1 {
      font-size: 3rem;
      margin-bottom: 1rem;
    }
    p {
      line-height: 1.6;
      margin-bottom: 1rem;
    }
    img {
      max-width: 100%;
      height: auto;
      border-radius: 8px;
      margin: 1rem 0;
    }
    .projects {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(15em, 1fr));
      gap: 1em;
      margin-top: 2rem;
    }
  </style>