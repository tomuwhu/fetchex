<script>
  import { onMount } from "svelte";
  var data = []
  const h = new Set()
  var ht = []
  onMount(async () => {
    data = (await fetch(
      "https://tomuwhu.github.io/orarend_generator_svelte/orarend.json"
    ).then((v) => v.json())).orarend
    data.forEach(v => {
      h.add(v.osztaly)
    })
    ht = Array.from(h)
  })
</script>

<main>
  Cica
  {#each ht as v}
    Cica
    {v}
  {/each}
</main>

<style>
</style>
