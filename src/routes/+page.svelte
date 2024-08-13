<script lang="ts">
  import { page } from "$app/stores";

  type ParamSet = {
    name: string;
    inner: string[];
  };
  type ParamJSON = {
    sets: ParamSet[];
  };

  let param = $page.url.searchParams.get("e");
  let displayError = false;

  let setText = "<SET>";
  let setElement: HTMLHeadingElement;
  let innerSetText = "<INNER>";
  let innerElement: HTMLHeadingElement;

  let innerShown = false;

  let setButtonDisabled = false;

  let sets: ParamSet[] = [];

  function doStuff() {
    if (param == null || param === "" || param === undefined) {
      displayError = true;
      return;
    }
    // param should be a base64 encoded json string
    // @ts-ignore - we return if it's invalid so womp womp
    let json: ParamJSON = undefined;
    try {
      let decoded = atob(param);
      json = JSON.parse(decoded);
      //   json = {
      //     sets: [
      //       {
      //         name: "test_set1",
      //         inner: ["test1", "test2", "test3", "test4"],
      //       },
      //       {
      //         name: "test_set2",
      //         inner: ["test5", "test6", "test7", "test8"],
      //       },
      //       {
      //         name: "test_set3",
      //         inner: ["test9", "test10", "test11", "test12"],
      //       },
      //       {
      //         name: "test_set4",
      //         inner: ["test13", "test14", "test15", "test16"],
      //       },
      //     ],
      //   };
      if (!json.sets) {
        console.error("No sets found in json");
      } else if (!Array.isArray(json.sets)) {
        console.error("Sets is not an array");
        // @ts-ignore - TS doesn't like this, but it's fine
      } else if (json.sets.length === 0) {
        throw new Error("No sets found in json");
      }
      console.log(json.sets);
    } catch (e) {
      console.error(e);
      displayError = true;
      return;
    }
    for (let set of json.sets) {
      if (!set.name || !set.inner) {
        console.error("Invalid set found in json");
        displayError = true;
        return;
      }
      if (!Array.isArray(set.inner)) {
        console.error("Inner is not an array");
        displayError = true;
        return;
      }
      console.log(set);
    }
    sets = json.sets;
  }

  function randomer() {
    setText = sets[Math.floor(Math.random() * sets.length)].name;
  }

  function pickSet() {
    console.log("Picking set");
    let i = 0;
    innerShown = false;
    setButtonDisabled = true;
    innerSetText = "<INNER>";
    let interval = setInterval(() => {
      setElement.classList.add("dull");
      randomer();
      i++;
      if (i > 20) {
        if (setElement.classList.contains("dull")) {
          setElement.classList.remove("dull");
        }
        innerShown = true;
        setButtonDisabled = false;
        clearInterval(interval);
      }
    }, 100);
  }

  function innerRandomer() {
    innerSetText =
      getSetFromName(setText).inner[
        Math.floor(Math.random() * getSetFromName(setText).inner.length)
      ];
  }

  function pickInner() {
    console.log("Picking inner");
    let i = 0;
    innerElement.classList.add("dull");
    setButtonDisabled = true;
    let interval = setInterval(() => {
      innerElement.classList.add("dull");
      innerRandomer();
      i++;
      if (i > 50) {
        if (innerElement.classList.contains("dull")) {
          innerElement.classList.remove("dull");
        }
        setButtonDisabled = false;
        clearInterval(interval);
      }
    }, 100);
  }

  function getSetFromName(name: string): ParamSet {
    for (let set of sets) {
      if (set.name === name) {
        return set;
      }
    }
    throw new Error("Set not found");
  }

  doStuff();
</script>

{#if displayError}
  <h1>404 - Page not found</h1>
  <p>Did someone send you a broken link? Please let them know.</p>
{:else}
  <h1>Sets</h1>
  <h2 contenteditable="false" bind:innerText={setText} bind:this={setElement}>
    &lt;SET&gt;
  </h2>
  <button disabled={setButtonDisabled} onclick={pickSet}>Pick Set</button>

  {#if innerShown}
    <h2>Inner</h2>
    <ul>
      {#each getSetFromName(setText).inner as item}
        <li>{item}</li>
      {/each}
    </ul>

    <h2
      contenteditable="false"
      bind:innerText={innerSetText}
      bind:this={innerElement}
    >
      &lt;INNER&gt;
    </h2>

    <button disabled={setButtonDisabled} onclick={pickInner}>Pick Inner</button>
  {/if}
  <br />
  <br />
  <details>
    <summary>Debug</summary>
    {#each sets as set}
      <h2>{set.name}</h2>
      <ul>
        {#each set.inner as item}
          <li>{item}</li>
        {/each}
      </ul>
    {/each}
  </details>
{/if}

<style lang="scss">
  @use "$lib/mocha.scss" as *;
  :global(body, html) {
    background-color: $ctp-base;
    color: $ctp-text;
    font-family: "Roboto", sans-serif;
  }
  :global(button) {
    background-color: $ctp-surface0;
    border-color: $ctp-surface1;
    color: $ctp-text;
    border-radius: 10px;
    padding: 5px 10px;
  }
  :global(.dull) {
    font-style: italic;
    opacity: 0.75;
  }
  :global(h2:not(.dull)) {
    font-weight: bold;
  }
</style>
