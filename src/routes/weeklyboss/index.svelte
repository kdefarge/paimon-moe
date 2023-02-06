<script>
  import { characters } from '../../data/characters';
  import Button from '../../components/Button.svelte';
  import Icon from '../../components/Icon.svelte';
  import { mdiMinus, mdiPlus } from '@mdi/js';
  $: chars = Object.entries(characters);
  
  let randomEvent = 0;
  
  function changeRandomEvent(count) {
    randomEvent += count;
    if (randomEvent < 0) randomEvent = 0;
    if (randomEvent > 10) randomEvent = 10;
    changed = true;
  }

</script>

<svelte:head>
  <title>Weekly Boss - Paimon.moe</title>
  <meta name="description" content="Genshin Impact (soon)" />
  <meta property="og:description" content="Genshin Impact (soon)" />
</svelte:head>

<div class="lg:ml-64 pt-20 lg:pt-8">

  <h1 class="font-display px-4 md:px-8 font-black text-5xl text-white">Weekly Boss</h1>

  <p class="text-gray-400 px-4 md:px-8 font-medium pb-4" style="margin-top: -1rem;">
    descriptions soon
  </p>

  <div class="flex flex-wrap text-white text-center">
  {#each chars as [id, char] (id)}
    <div class="flex flex-wrap bg-item ml-2 px-2 mt-2 py-2 rounded-lg">
      
      <div class="w-32 font-semibold">
        {char.name}
        <img class="block h-24 rounded-full mx-auto" src={`/images/characters/${id}.png`} alt="{char.name}">
        <img class="block h-24 rounded-lg mx-auto" src={`/images/items/${char.material.boss.id}.png`} alt="{char.name}">
      </div>

      <table class="table-auto">
        <thead>
          <tr>
            <th>Talents</th>
            <th>unlock</th>
            <th>wanted</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>Normal Attack</td>
            <td><div class="flex justify-center">
              <Button on:click={() => changeRandomEvent(-1)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{randomEvent}</p>
              <Button on:click={() => changeRandomEvent(1)}><Icon path={mdiPlus} /></Button>
            </div></td>
            <td><div class="flex justify-center">
              <Button on:click={() => changeRandomEvent(-1)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{randomEvent}</p>
              <Button on:click={() => changeRandomEvent(1)}><Icon path={mdiPlus} /></Button>
            </div></td>
          </tr>
          <tr>
            <td>Elemental Skill</td>
            <td><div class="flex justify-center">
              <Button on:click={() => changeRandomEvent(-1)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{randomEvent}</p>
              <Button on:click={() => changeRandomEvent(1)}><Icon path={mdiPlus} /></Button>
            </div></td>
            <td><div class="flex justify-center">
              <Button on:click={() => changeRandomEvent(-1)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{randomEvent}</p>
              <Button on:click={() => changeRandomEvent(1)}><Icon path={mdiPlus} /></Button>
            </div></td>
          </tr>
          <tr>
            <td>Elemental Burst</td>
            <td><div class="flex justify-center">
              <Button on:click={() => changeRandomEvent(-1)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{randomEvent}</p>
              <Button on:click={() => changeRandomEvent(1)}><Icon path={mdiPlus} /></Button>
            </div></td>
            <td><div class="flex justify-center">
              <Button on:click={() => changeRandomEvent(-1)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{randomEvent}</p>
              <Button on:click={() => changeRandomEvent(1)}><Icon path={mdiPlus} /></Button>
            </div></td>
          </tr>
        </tbody>
      </table>
    </div>
  {/each}
  </div>
</div>

