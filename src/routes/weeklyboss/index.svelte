<script>
  import { characters } from '../../data/characters';
  import Button from '../../components/Button.svelte';
  import Icon from '../../components/Icon.svelte';
  import { mdiMinus, mdiPlus } from '@mdi/js';
  import { weeklyboss } from '../../data/weeklyboss';
  import { onMount } from 'svelte';
  import debounce from 'lodash.debounce';
  import { getAccountPrefix } from '../../stores/account';
  import { readSave, updateSave, fromRemote } from '../../stores/saveManager';

  const talents = {
    NormalAttack: 0,
    ElementalSkill: 1,
    ElementalBurst: 2,
  };

  const counterType = {
    Unlock: 0,
    Wanted: 1,
    Max: 2,
  };

  let itemsCounter = [];
  let talentsCounter = [];
  let bossList = [];
  let charactersList = [];

  onMount(async () => {
    for (const [id, boss] of Object.entries(weeklyboss)) {
      for (const [id, item] of Object.entries(boss.unique_drop)) {
        itemsCounter[item.id] = [0,0,0];
      }
    }
    bossList = Object.entries(weeklyboss);
    charactersList = Object.entries(characters);
    for (const [id, character] of charactersList) {
      talentsCounter[character.id] = [[1,1,1],[1,1,1]];
      itemsCounter[character.material.boss.id][counterType.Max]+=18;
    }
    readLocalData();
  });

  function talentsLvlPush(character,talent,counter = counterType.Unlock) {
    let talentLvl = talentsCounter[character][counter][talent];
    if(talentLvl>9)
        return;
    
    talentLvl++;

    if(counter == counterType.Unlock
    && talentLvl > talentsCounter[character][counterType.Wanted][talent])
      talentsCounter[character][counterType.Wanted][talent] = talentLvl;

    talentsCounter[character][counter][talent] = talentLvl;
    bossweeklyCalculator();
    saveData();
  }

  
  function talentsLvlPull(character,talent,counter = counterType.Unlock) {
    let talentLvl = talentsCounter[character][counter][talent];
    if(talentLvl<=1)
        return;
    
    talentLvl--;

    if(counter == counterType.Wanted 
    && talentLvl < talentsCounter[character][counterType.Unlock][talent])
        return;

    talentsCounter[character][counter][talent] = talentLvl;
    bossweeklyCalculator();
    saveData();
  }

  function bossweeklyCalculator() {

    let itemsCounterTmp = [];

    for (const [id, boss] of Object.entries(weeklyboss)) {
      for (const [id, item] of Object.entries(boss.unique_drop)) {
        itemsCounterTmp[item.id] = [0,0];
      }
    }

    for (const [id, character] of charactersList) {
      for (const [id, talent] of Object.entries(talents)) {
        let nb_unlock = lvlTalent2nbItem(talentsCounter[character.id][counterType.Unlock][talent]);
        itemsCounterTmp[character.material.boss.id][counterType.Unlock]+=nb_unlock;
        let nb_wanted = lvlTalent2nbItem(talentsCounter[character.id][counterType.Wanted][talent]) - nb_unlock;
        itemsCounterTmp[character.material.boss.id][counterType.Wanted]+=nb_wanted;
      }
    }

    for (const [id, itemCounter] of Object.entries(itemsCounterTmp)) {
      itemsCounter[id][counterType.Unlock] = itemsCounterTmp[id][counterType.Unlock];
      itemsCounter[id][counterType.Wanted] = itemsCounterTmp[id][counterType.Wanted];
    }
  }

  function lvlTalent2nbItem(lvl) {
    switch (lvl) {
      case 7:
        return 1;
      case 8:
        return 2;
      case 9:
        return 4;
      case 10:
        return 6;
    }
    return 0;
  }

  const saveData = debounce(async () => {
    const wbdata = {};
    for (const [characterid, character] of charactersList) {
      wbdata[characterid] = [[],[]];
      for (const [talentid, talent] of Object.entries(talents)) {
        wbdata[characterid][counterType.Unlock][talent] = talentsCounter[characterid][counterType.Unlock][talent];
        wbdata[characterid][counterType.Wanted][talent] = talentsCounter[characterid][counterType.Wanted][talent];
      }
    }
    const prefix = getAccountPrefix();
    await updateSave(`${prefix}weeklyboss`, wbdata);
  }, 2000);
  
  async function readLocalData() {
    const prefix = getAccountPrefix();
    const wbdata = await readSave(`${prefix}weeklyboss`);
    for (const [characterid, character] of charactersList) {
      for (const [talentid, talent] of Object.entries(talents)) {
        talentsCounter[characterid][counterType.Unlock][talent] = wbdata[characterid][counterType.Unlock][talent];
        talentsCounter[characterid][counterType.Wanted][talent] = wbdata[characterid][counterType.Wanted][talent];
      }
    }
    bossweeklyCalculator();
  }

  $: if ($fromRemote) {
    console.log('update from google drive');
    readLocalData();
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
  {#each bossList as [id, boss] (id)}
    <div class="bg-item ml-2 px-2 mt-2 py-2 rounded-lg">
      
      <div class="font-semibold mx-auto text-xl">
        {boss.name}
      </div>

      <table class="table-auto">
        <thead>
          <tr>
            <th class="w-64">Item name</th>
            <th class="w-24">Icon</th>
            <th>Unlock</th>
            <th>diff</th>
            <th>wanted</th>
          </tr>
        </thead>
        <tbody>
          {#each Object.entries(boss.unique_drop) as [id, item] (id)}
          <tr>
            <td>{item.name}</td>
            <td><img class="block h-12 rounded-full mx-auto" src={`/images/items/${item.id}.png`} alt="{item.name}"></td>
            <td>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">
                {itemsCounter[item.id][counterType.Unlock]} / {itemsCounter[item.id][counterType.Max]}</p>
            </td>
            <td>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">
                {(itemsCounter[item.id][counterType.Max]-itemsCounter[item.id][counterType.Unlock])}</p>
            </td>
            <td>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">{itemsCounter[item.id][counterType.Wanted]}</p>
            </td>
          </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {/each}
  </div>

  <h2 class="font-display px-4 md:px-8 font-black text-4xl text-white">Talents</h2>

  <div class="flex flex-wrap text-white text-center">
    {#each charactersList as [id, character] (id)}
    <div class="flex flex-wrap bg-item ml-2 px-2 mt-2 py-2 rounded-lg">
      
      <div class="w-32 font-semibold">
        {character.name}
        <img class="block h-24 rounded-full mx-auto" src={`/images/characters/${character.id}.png`} alt="{character.name}">
        <img class="block h-24 rounded-lg mx-auto" src={`/images/items/${character.material.boss.id}.png`} alt="{character.name}">
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
          {#each Object.entries(talents) as [talentid, talent] (talentid)}
          <tr>
            <td>{talentid}</td>
            <td><div class="flex justify-center">
              <Button on:click={() => talentsLvlPull(character.id,talent)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">
                {talentsCounter[character.id][counterType.Unlock][talent]}</p>
              <Button on:click={() => talentsLvlPush(character.id,talent)}><Icon path={mdiPlus} /></Button>
            </div></td>
            <td><div class="flex justify-center">
              <Button on:click={() => talentsLvlPull(character.id,talent,counterType.Wanted)}><Icon path={mdiMinus} /></Button>
              <p class="mx-2 bg-background rounded-xl text-bold text-white py-4 w-16 text-center">
                {talentsCounter[character.id][counterType.Wanted][talent]}</p>
              <Button on:click={() => talentsLvlPush(character.id,talent,counterType.Wanted)}><Icon path={mdiPlus} /></Button>
            </div></td>
          </tr>
          {/each}
        </tbody>
      </table>
    </div>
  {/each}
  </div>
</div>

