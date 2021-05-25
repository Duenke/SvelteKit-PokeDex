<script>
    export let sprites
  </script>
  
  {#if sprites.front_default}
    <img alt="sprites.front_default" src={sprites.front_default} />
  {/if}
  {#if sprites.back_default}
    <img alt="sprites.back_default" src={sprites.back_default} />
  {/if}
  
  {#if sprites.front_female}
    <img alt="sprites.front_female" src={sprites.front_female} />
  {/if}
  {#if sprites.back_female}
    <img alt="sprites.back_female" src={sprites.back_female} />
  {/if}
  
  {#if sprites.front_shiny}
    <img alt="sprites.front_shiny" src={sprites.front_shiny} />
  {/if}
  {#if sprites.back_shiny}
    <img alt="sprites.back_shiny" src={sprites.back_shiny} />
  {/if}
  
  {#if sprites.front_shiny_female}
    <img alt="sprites.front_shiny_female" src={sprites.front_shiny_female} />
  {/if}
  {#if sprites.back_shiny_female}
    <img alt="sprites.back_shiny_female" src={sprites.back_shiny_female} />
  {/if}