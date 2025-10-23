<script>
  
  import { onMount } from "svelte";

  import { animate } from 'animejs';
  import { marked } from "marked";

  import { Slider } from "bits-ui";
  import { legalDoc } from "$lib/legalDoc";

  const months = [
    'january',
    'february',
    'march',
    'april',
    'may',
    'june',
    'july',
    'august',
    'september',
    'october',
    'november',
    'december',
  ]

  const currentYear = new Date().getFullYear();
  
  let selectedMonth = $state(-1);
  let selectedDay = $state(-1);
  let selectedYear = $state(-1);

  let expRes = $state({month: selectedMonth, day: selectedDay, year: selectedYear});

  let rolling = $state(false);
  let pullAmount = $state(100);
  let proxyAmount = {value: pullAmount};
  let pullHandle = $state(null);
  let pullThreshold = 5;
  let flightCost = $state('');
  let baseCost = $state('');

  let spinnerMonth = $state(null);
  let spinnerDay = $state(null);
  let spinnerYear = $state(null);

  let rollingPromise;

  let years = $derived.by(() => {
    let arr = [];

    for (let i = 50; i > 0; i--) {
      arr.push(currentYear - (i + 1));
    }

    arr.push(currentYear);

    for (let i = 0; i < 50; i++) {
      arr.push(currentYear + (i + 1));
    }

    return arr;
  });

  $effect(()=>{
    if (spinnerMonth) {
      const currentItem = spinnerMonth.children[selectedMonth];
      if (currentItem) {
        currentItem.scrollIntoView({ block: 'center', behavior: 'smooth' });
      }
    }
    if (spinnerDay) {
      const currentItem = spinnerDay.children[selectedDay];
      if (currentItem) {
        currentItem.scrollIntoView({ block: 'center', behavior: 'smooth' });
      }
    }
    if (spinnerYear) {
      const currentItem = spinnerYear.children[selectedYear];
      if (currentItem) {
        currentItem.scrollIntoView({ block: 'center', behavior: 'smooth' });
      }
    }
  });

  function animatePull() {
    animate(proxyAmount, {
      value: 100,
      duration: 6000,
      onUpdate: () => {
        pullAmount = proxyAmount.value;
      }
    });
  }

  function resetPull(e) {
    pullAmount = 100;
  }

  const sleep = (delay) => {return new Promise((resolve) => setTimeout(resolve, delay ?? 200));};
  
  function handlePull(e) {
    if (pullAmount < pullThreshold && !rolling) {
      resetPull(e);
      handleRoll(e);
    }
  };

  async function handleRoll() {
    rolling = true;
    flightCost = '';

    const targetMonth = Math.floor(Math.random() * months.length);
    const targetDay = Math.floor(Math.random() * 31);
    const targetYear = Math.floor(Math.random() * years.length);

    expRes = {month: targetMonth, day: targetDay, year: targetYear};

    const delayStart = 100; // 20ms between changes
    const rollsMonth = 20;
    const rollsDay = 40;
    const rollsYear = 60;

    selectedMonth = ((targetMonth - rollsMonth) % months.length + months.length) % months.length;
    selectedDay = ((targetDay - rollsDay) % 31 + 31) % 31;
    selectedYear = ((targetYear - rollsYear) % years.length + years.length) % years.length;

    rollingPromise = await Promise.all([
      // Month roller
      (async () => {
        for (let i = 0; i < rollsMonth; i++) {
          selectedMonth = (selectedMonth + 1) % months.length;

          if (rollsMonth - i < 5) {
            await sleep(delayStart * 4);
          } else if (rollsMonth - i < 10) {
            await sleep(delayStart * 2);
          } else {
            await sleep(delayStart);
          }

          console.log(pullAmount)
        }
      })(),
      
      // Day roller
      (async () => {
        for (let i = 0; i < rollsDay; i++) {
          selectedDay = (selectedDay + 1) % 31;

          if (rollsDay - i < 5) {
            await sleep(delayStart * 4);
          } else if (rollsDay - i < 10) {
            await sleep(delayStart * 2);
          } else {
            await sleep(delayStart);
          }
        }
      })(),
      
      // Year roller
      (async () => {
        for (let i = 0; i < rollsYear; i++) {
          selectedYear = (selectedYear + 1) % years.length;

          if (rollsYear - i < 5) {
            await sleep(delayStart * 4);
          } else if (rollsYear - i < 10) {
            await sleep(delayStart * 2);
          } else {
            await sleep(delayStart);
          }
        }
      })()
    ]);

    baseCost = Math.ceil(Math.random() * 1000);

    if (years[selectedYear] < currentYear) {
      flightCost = Math.ceil(Math.random() * (baseCost * ((currentYear - years[selectedYear]) * 4)));
    } else {
      flightCost = Math.ceil(Math.random() * Math.ceil(baseCost / (years[selectedYear] - currentYear)));
    }

    if (flightCost < baseCost) {
      flightCost = baseCost;
    }
    
    rolling = false;
  }

  async function rollNumber() {
    
  };

</script>

<!-- svelte-ignore a11y_click_events_have_key_events -->
<!-- svelte-ignore a11y_no_noninteractive_element_interactions -->
<div class="mb-2 bg-surface-800 p-2 rounded-xl">
  <h2 class="h2 text-center text-primary-500">Plan your next flight!</h2>
  <p class="w-full text-center ">No round-trips!</p>
</div>
<div class="mb-2 bg-surface-950 p-2 rounded-xl">
  <p class="text-sm text-center text-surface-300">Destination:</p>
  <p class="w-full text-center text-error-500 font-bold">Pre-determined on your behalf.</p>
</div>
<div class="grid grid-cols-[auto_max-content] gap-2">
  <div class="bg-surface-800 text-surface-300 w-full p-2 rounded-xl space-y-2 font-medium">
    <div class="w-fit flex items-center gap-2 overflow-hidden mx-auto">

      <hr class="hr w-4 border-primary-500">

      <ul class="flex flex-col gap-2 capitalize w-fit h-24 border border-surface-400 rounded-md p-2 overflow-y-auto py-12 pr-2 pointer-events-none" bind:this={spinnerMonth}>
      {#each months as month, i}
        <li
          class="whitespace-nowrap {selectedMonth === i ? 'text-primary-500 font-bold' : ''}"
          onclick={()=>{selectedMonth = i}}
        ><span class="">{(i + 1).toString().padStart(2, '0')}</span> - {month}</li>
      {/each}
      </ul>
      
      <hr class="hr w-4 border-primary-500">

      <ul class="flex flex-col gap-2 w-fit h-24 border border-surface-400 rounded-md p-2 overflow-y-auto py-12 pr-2 pointer-events-none" bind:this={spinnerDay}>
      {#each {length: 31} as _, i}
        <li
          class="whitespace-nowrap {selectedDay === i ? 'text-primary-500 font-bold' : ''}"
          onclick={()=>{selectedDay = i}}
        >{(i + 1).toString().padStart(2, '0')}</li>
      {/each}
      </ul>

      <hr class="hr w-4 border-primary-500">

      <ul class="flex flex-col gap-2 w-fit h-24 border border-surface-400 rounded-md p-2 overflow-y-auto py-12 pr-2 pointer-events-none" bind:this={spinnerYear}>
      {#each years as year, i}
        <li
          class="whitespace-nowrap {selectedYear === i ? 'text-primary-500 font-bold' : ''}"
          onclick={()=>{selectedYear = i}}
        >{year}</li>
      {/each}
      </ul>

      <hr class="hr w-4 border-primary-500">
    </div>
  </div>

  <div class="w-min bg-surface-800 text-surface-300 p-4 rounded-xl space-y-2 font-medium" tabindex="-1">
    <Slider.Root
      type="single"
      bind:value={pullAmount}
      class="relative flex h-full touch-none select-none flex-col items-center {pullAmount < pullThreshold || rolling === true ? 'pointer-events-none' : ''}"
      trackPadding={3}
      orientation="vertical"
      onValueChange={(e)=>{handlePull(e)}}
      onValueCommit={(e)=>{e.target?.blur(); resetPull(e)}}
    >
      <span
        class="bg-transparent relative h-full w-2 cursor-pointer overflow-hidden rounded-full"
      >
        <Slider.Range class="bg-surface-500 absolute w-full rounded-full" />
      </span>
      <Slider.Thumb
        index={0}
        class="border-2 {rolling === true ? 'bg-error-500 border-error-800' : 'border-success-800 bg-success-500 hover:border-success-950'} block size-6 cursor-pointer rounded-full shadow-sm transition-colors focus-visible:ring-2 focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50"
      />
    </Slider.Root>
  </div>
</div>
<div class="bg-surface-800 p-2 rounded-xl mt-2">
  <p class="text-sm max-w-96 text-surface-300 w-full text-center text-pretty leading-tight">Click the handle on the right side and drag down to set your arrival date!</p>
</div>
<div class="mb-2 bg-surface-800 p-2 rounded-xl mt-2">
  <h4 class="h4 text-center text-primary-500">Flight cost estimate:</h4>
  <!-- <div class="flex flex-col px-4">
    <div class="w-full grid grid-cols-[max-content_auto_max-content]">
      <p class="">Base Ticket Price</p>
      <hr class="w-full hr">
      <p class="">${baseCost?.toString()?.padStart(2, '-')}</p>
    </div>
  </div> -->
  <h1 class="h1 w-full text-center tracking-widest">${flightCost?.toString()?.padStart(2, '-')}</h1>
</div>
<div class="w-full bg-surface-950 text-surface-400 p-2 rounded-xl text-sm text-center">
  <p class="text-sm mb-2">Disclaimer</p>
  <div class="max-h-48 overflow-y-auto pr-2">
    <p class="max-w-96 text-xs space-y-4 mx-auto **:border-surface-700">{@html marked.parse(legalDoc)}</p>
  </div>
</div>