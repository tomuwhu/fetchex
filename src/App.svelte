<script>
  import { onMount } from "svelte";
  var [data, tanarok, targyak, ht, orl, orak] = [[], [], [], [], [], []];
  var [tanarset, targyset, orarendset] = [{}, {}, {}];
  var selectedoszt;
  var [mino, maxo] = [20, 0];
  const napok = "H K Sz Cs P".split(" ");
  const napnevek = "Hétfő Kedd Szerda Csütörtök Péntek".split(" ");
  const h = new Set();
  onMount(async () => {
    data = (
      await fetch(
        "https://tomuwhu.github.io/orarend_generator_svelte/orarend.json",
      ).then((v) => v.json())
    ).orarend;
    data.forEach((v) => {
      h.add(v.osztaly);
    });
    ht = Array.from(h)
      .filter((v) => !v.includes(","))
      .sort();
    tanarok = (
      await fetch(
        "https://tomuwhu.github.io/orarend_generator_svelte/tanarok.json",
      ).then((v) => v.json())
    ).tanarok;
    tanarok.forEach(
      (v) =>
        (tanarset[v.ok] = v.tit?.trim()
          ? `${v.tit} ${v.familyname} ${v.firstname}`
          : `${v.familyname} ${v.firstname}`),
    );
    targyak = (
      await fetch(
        "https://tomuwhu.github.io/orarend_generator_svelte/targyak.json",
      ).then((v) => v.json())
    ).targyak;
    targyak.forEach((v) => {
      let [c, ...kl] = v.ttrov.toLowerCase();
      if (!";.".includes(c)) kl = c + kl.join("");
      else kl = kl.join("");
      targyset[kl] = v.ttnev;
    });
  });
  const terem = (ora, nap) =>
    orl.filter((v) => v.ora === ora).filter((v) => v.nap === nap)[0]?.terem ||
    "";
  const tanar = (ora, nap) => {
    let tk =
      orl.filter((v) => v.ora === ora).filter((v) => v.nap === nap)[0]?.tanar ||
      "-";
    if (tk === "-") return "";
    let tanarnev = tanarset[tk];
    if (tanarnev?.length) return tanarnev;
    return tk;
  };
  const targy = (ora, nap) => {
    let tk =
      orl.filter((v) => v.ora === ora).filter((v) => v.nap === nap)[0]?.targy ||
      "-";
    if (tk === "-") return "";
    let targynev =
      targyset[
        tk.replaceAll("é", "e").replaceAll("á", "a").replaceAll("ö", "o")
      ];
    if (targynev?.length) return targynev;
    return `<i>${tk}</i>`;
  };
</script>

<main>
  <select
    bind:value={selectedoszt}
    on:change={() => {
      [mino, maxo] = [20, 0];
      orak = [];
      orl = data.filter(
        (v) => v.osztaly.includes(selectedoszt) && v.het === "A",
      );
      orl.forEach((v) => {
        if (mino > v.ora) mino = v.ora;
        if (maxo < v.ora) maxo = v.ora;
      });
      for (let i = mino; i <= maxo; orak.push(i++)) {}
      orak;
    }}
  >
    {#each ht as v}
      <option value={v}>
        {v}
      </option>
    {/each}
  </select>
  {#if orak.length}
    <table>
      <tr>
        <th>#</th>
        {#each napnevek as napnév}
          <th>{napnév}</th>
        {/each}
      </tr>
      {#each orak as ora}
        <tr>
          <th>{ora}</th>
          {#each napok as nap}
            <td>
              <div class="targy">
                {@html targy(ora, nap)}
              </div>
              <div class="tanar">
                {tanar(ora, nap)}
              </div>
              <div class="terem">
                {terem(ora, nap)}
              </div>
            </td>
          {/each}
        </tr>
      {/each}
    </table>
  {/if}
</main>

<style>
  table {
    border-spacing: 10px;
  }
  td {
    background-color: rgb(187, 235, 219);
    color: black;
    border: solid 2px green;
    width: 120px;
    height: 60px;
    box-shadow: 1px 1px 3px black;
  }
  div.targy {
    font-size: 10px;
  }
  div.tanar {
    font-size: 9px;
    color: white;
    background-color: rgb(14, 108, 59);
  }
  div.terem {
    font-size: 12px;
  }
  :global(i) {
    color: red;
  }
</style>
