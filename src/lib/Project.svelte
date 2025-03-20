<script>
  export let data = {};
  export let hLevel = 2;
</script>

<article>
  <svelte:element this={"h" + hLevel}>{data.title}</svelte:element>
  
  <img src={data.image} alt={data.title} />
 
    <div>
        <p>{data.description} </p>
        <br>
        <p> c. {data.year}</p>
    </div>

</article>

<style>
  article {
    display: grid;
    grid-template-rows: subgrid; /* Note: subgrid is experimental and may not work in all browsers */
    grid-row: span 3;
    font-variant-numeric: oldstyle-nums
  }
  
  article h2 {
    margin: 0;
  }
  
  img {
    max-width: 100%;
  }
</style>
