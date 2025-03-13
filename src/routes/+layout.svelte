

<script>
$:localStorage.colorScheme = colorScheme;
let localStorage = globalThis.localStorage ?? {};
let root = globalThis?.document?.documentElement;
$: root?.style.setProperty("color-scheme", colorScheme);
let colorScheme = localStorage.colorScheme ?? "light dark";



import { page } from "$app/stores";

let pages = [
  { url: "./", title: "Home" },
  { url: "./projects", title: "Projects" },
  { url: "./contact", title: "Contact" },
  { url: "./resume", title: "Resume" },
  { url: "https://github.com/MarialeP24", title: "Github" }
  ];

  
</script>


<nav>
    {#each pages as p}
    <a
    href={p.url}
    class:current={"." + $page.route.id === p.url}
    target={p.url.startsWith("http") ? "_blank" : null}
  >
    {p.title}
  </a>
  
    {/each}
  </nav>

  <!-- <p>Current color scheme: {colorScheme}</p> -->
  
  <label class="color-scheme">
    Theme:
    <select bind:value={ colorScheme }>

        <option value="light dark">Auto</option>
        <option value="dark">Dark</option>
        <option value="light">Light</option>
    </select>
</label>

  <slot />

  <style> 
  
    nav ul,
   nav li{
    display: contents;
   }

   nav{
    --border-color: oklch(50% 10% 200 / 40%);
    display: flex;
    border-bottom: 1px;
    border-bottom-style: solid;
    border-bottom-color:var(--border-color);
    margin-bottom: 2em;
    
   }

   nav a {
   flex:1;
   text-align: center;
   text-decoration: none;
   color: inherit;
   text-align: center;
   padding: 0.5em;
   margin-bottom: 0.5em
   }

   nav a.current{ 
    border-bottom: 0.4em;
    border-bottom-style: solid;
    border-bottom-color:var(--border-color);
    margin-bottom: 0.1em

   }

   nav a:hover{
    border-bottom: 0.4em;border-bottom-style: solid;border-bottom-color:var(--color-accent);
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
    margin-bottom: 0.1em
   }

   .color-scheme {
        position: absolute;
        top: 1rem;
        right: 1rem;
        font-size: 80%;
        display: inline-flex;
        gap: 4px;
      }



  </style>

  