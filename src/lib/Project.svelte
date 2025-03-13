<script>
  export let data = {};
  export let hLevel = 2;
</script>

<article>
  <svelte:element this={"h" + hLevel}>{data.title}</svelte:element>
  <p>{data.year}</p>
  <img src={data.image} alt={data.title} />
  <p>{data.description}</p>
</article>

<style>
  article {
    display: grid;
    grid-template-rows: subgrid; /* Note: subgrid is experimental and may not work in all browsers */
    grid-row: span 3;
  }
  
  article h2 {
    margin: 0;
  }
  
  img {
    max-width: 100%;
  }
</style>
