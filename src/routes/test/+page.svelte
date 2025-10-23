<script>
  import { onMount } from "svelte";

  const months = [
    'january', 'february', 'march', 'april', 'may', 'june',
    'july', 'august', 'september', 'october', 'november', 'december',
  ];

  const currentYear = new Date().getFullYear();

  let selectedMonth = $state(0);
  let selectedDay = $state(0);
  let selectedYear = $state(0);

  let rolling = $state(false);

  let spinnerMonth = $state(null);
  let spinnerDay = $state(null);
  let spinnerYear = $state(null);

  // Create tripled arrays for infinite scroll effect
  let monthsTripled = $derived([...months, ...months, ...months]);
  let daysTripled = $derived([...Array(31), ...Array(31), ...Array(31)]);
  
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
  
  let yearsTripled = $derived([...years, ...years, ...years]);

  // Track actual display indices (in tripled array)
  let displayMonth = $state(0);
  let displayDay = $state(0);
  let displayYear = $state(0);

  onMount(() => {
    // Start in middle set
    displayMonth = months.length + selectedMonth;
    displayDay = 31 + selectedDay;
    displayYear = years.length + selectedYear;
  });

  $effect(() => {
    if (spinnerMonth && spinnerMonth.children[displayMonth]) {
      spinnerMonth.children[displayMonth].scrollIntoView({ block: 'center', behavior: 'smooth' });
    }
  });

  $effect(() => {
    if (spinnerDay && spinnerDay.children[displayDay]) {
      spinnerDay.children[displayDay].scrollIntoView({ block: 'center', behavior: 'smooth' });
    }
  });

  $effect(() => {
    if (spinnerYear && spinnerYear.children[displayYear]) {
      spinnerYear.children[displayYear].scrollIntoView({ block: 'center', behavior: 'smooth' });
    }
  });

  const sleep = (delay) => new Promise((resolve) => setTimeout(resolve, delay ?? 200));
  
  async function handleRoll() {
    rolling = true;

    const targetMonth = Math.floor(Math.random() * months.length);
    const targetDay = Math.floor(Math.random() * 31);
    const targetYear = Math.floor(Math.random() * years.length);

    const delayStart = 100;
    const rollsMonth = 32;
    const rollsDay = 64;
    const rollsYear = 96;

    await Promise.all([
      // Month roller
      (async () => {
        for (let i = 0; i < rollsMonth; i++) {
          displayMonth++;
          selectedMonth = displayMonth % months.length;

          if (rollsMonth - i < 5) {
            await sleep(delayStart * 4);
          } else if (rollsMonth - i < 10) {
            await sleep(delayStart * 2);
          } else {
            await sleep(delayStart);
          }
        }
        
        // Reset to middle set for next roll
        displayMonth = months.length + targetMonth;
        selectedMonth = targetMonth;
      })(),
      
      // Day roller
      (async () => {
        for (let i = 0; i < rollsDay; i++) {
          displayDay++;
          selectedDay = displayDay % 31;

          if (rollsDay - i < 5) {
            await sleep(delayStart * 4);
          } else if (rollsDay - i < 10) {
            await sleep(delayStart * 2);
          } else {
            await sleep(delayStart);
          }
        }
        
        displayDay = 31 + targetDay;
        selectedDay = targetDay;
      })(),
      
      // Year roller
      (async () => {
        for (let i = 0; i < rollsYear; i++) {
          displayYear++;
          selectedYear = displayYear % years.length;

          if (rollsYear - i < 5) {
            await sleep(delayStart * 4);
          } else if (rollsYear - i < 10) {
            await sleep(delayStart * 2);
          } else {
            await sleep(delayStart);
          }
        }
        
        displayYear = years.length + targetYear;
        selectedYear = targetYear;
      })()
    ]);
    
    rolling = false;
  }
</script>

<div class="bg-surface-800 text-surface-300 w-max p-2 rounded-xl space-y-2 font-medium">
  <h3 class="h3 text-primary-500 font-extrabold text-center">Pick a date!</h3>

  <div class="w-max grid grid-cols-[repeat(4,_auto)] items-center gap-2 overflow-hidden">

    <ul class="flex flex-col gap-2 capitalize w-full h-8 border border-surface-400 rounded-md p-2 overflow-y-auto pr-2 pointer-events-none" bind:this={spinnerMonth}>
    {#each monthsTripled as month, i}
      <li
        class="whitespace-nowrap transition-all {selectedMonth === (i % months.length) && displayMonth === i ? 'text-surface-50' : ''}"
      ><span>{((i % months.length) + 1).toString().padStart(2, '0')}</span> - {month}</li>
    {/each}
    </ul>

    <ul class="flex flex-col gap-2 w-full h-8 border border-surface-400 rounded-md p-2 overflow-y-auto pr-2 pointer-events-none" bind:this={spinnerDay}>
    {#each daysTripled as _, i}
      <li
        class="whitespace-nowrap transition-all {selectedDay === (i % 31) && displayDay === i ? 'text-surface-50' : ''}"
      >{((i % 31) + 1).toString().padStart(2, '0')}</li>
    {/each}
    </ul>

    <ul class="flex flex-col gap-2 w-full h-8 border border-surface-400 rounded-md p-2 overflow-y-auto pr-2 pointer-events-none" bind:this={spinnerYear}>
    {#each yearsTripled as year, i}
      <li
        class="whitespace-nowrap transition-all {selectedYear === (i % years.length) && displayYear === i ? 'text-surface-50' : ''}"
      >{year}</li>
    {/each}
    </ul>

    <button
      class="btn preset-filled-success-500"
      onclick={()=>handleRoll()}
      disabled={rolling}
    >Roll</button>

  </div>
</div>