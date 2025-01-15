<script>
  import "@picocss/pico";
  import Mark from "mark.js";
  import { onMount } from "svelte";

  let linksArray = $state([]);
  let fullText = $state();

  let reg = /https?:\/\/[^\s.]+(?:\.[^\s.]+)+(?!\.\.\.)/gim;

  let markInstance;
  const initializeMark = () => {
    setTimeout(() => {
      const context = document.querySelector(".uploaded-text");
      if (context) {
        markInstance = new Mark(context);
      }
    }, 0);
  };

  const handleFile = (event) => {
    let file = event.target.files[0];
    console.log(event.target.files);

    let reader = new FileReader();
    reader.readAsText(file);

    reader.onload = () => {
      console.log(reader.result);
      getLinksFromText(reader.result);
    };

    reader.onerror = () => {
      console.log(reader.error);
    };
  };

  const getLinksFromText = (text) => {
    console.log(text.match(reg));
    fullText = text;
    const filteredLines = text
      .split("\n")
      .filter((line) => !line.includes("Reacted to"))
      .filter((line) => !line.includes("Removed a"))
      .join("\n");
    linksArray = filteredLines.match(reg);

    initializeMark();
    setTimeout(() => {
      if (markInstance) {
        markInstance.unmark();
        markInstance.markRegExp(reg);
      }
    }, 100);
  };
</script>

<main>
  <h1>LinkGrab</h1>
  <p>Easily grab all the shared links from a zoom call chat.</p>
  <input
    id="uploadInput"
    type="file"
    onchange={(event) => handleFile(event)}
    accept=".txt"
  />
  {#if linksArray.length > 0}
    <div class="result">
      <article class="uploaded-text">
        <pre class="context">{fullText}</pre>
      </article>
      <article class="links">
        {#each linksArray as link}
          <p>- {link}</p>
        {/each}
      </article>
    </div>
  {/if}
</main>

<style>
  main {
    padding: 2rem;
  }
  .result {
    display: flex;
    flex-flow: row;
    gap: 1rem;
  }
  .uploaded-text {
    flex-grow: 1;
    word-wrap: pre-wrap;
    line-height: 200%;
  }
  .links {
    flex-grow: 2;
  }
  pre {
    white-space: pre-wrap;
  }
</style>
