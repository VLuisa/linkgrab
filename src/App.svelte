<script>
  import "@picocss/pico";
  import Mark from "mark.js";

  let linksArray = $state([]);
  let fullText = $state();
  let filteredLines = $state();
  let fullLines = $state();
  let textContext = $state(null);
  let markInstance = $state(null);

  let reg = /https?:\/\/[^\s.]+(?:\.[^\s.]+)+(?!\.\.\.)/gim;

  $effect(() => {
    if (textContext && !markInstance) {
      markInstance = new Mark(textContext);
      markInstance.markRegExp(reg);
    }
  });

  const handleFile = (event) => {
    let file = event.target.files[0];

    let reader = new FileReader();
    reader.readAsText(file);

    reader.onload = () => {
      console.log(reader.result);
      getLinksFromText(reader.result);
    };

    reader.onerror = () => {
      console.log(reader.error);
      alert(
        "There was an error reading your file. Make sure you're uploading the .txt file from a Zoom call."
      );
    };
  };

  const getLinksFromText = (text) => {
    fullText = text;
    fullLines = text.split("\n");
    filteredLines = text
      .split("\n")
      .filter((line) => !line.includes("Reacted to"))
      .filter((line) => !line.includes("Removed a"));
    let filteredText = filteredLines.join("\n");
    linksArray = filteredText.match(reg);
  };
</script>

<main>
  <hgroup>
    <h1>LinkGrab</h1>
    <p>Easily grab all the shared links from a zoom call chat .txt file</p>
  </hgroup>
  <input
    id="uploadInput"
    type="file"
    onchange={(event) => handleFile(event)}
    accept=".txt"
    aria-busy="true"
  />
  {#if linksArray.length > 0}
    <div class="grid">
      <div class="left-column">
        <h3>Zoom chat</h3>
        <article class="uploaded-text" bind:this={textContext}>
          {#each fullLines as line}
            <p>{line}</p>
          {/each}
        </article>
      </div>
      <div class="right-column">
        <div class="header">
          <h3>Links in chat</h3>
          <button class="secondary outline">Copy links as list</button>
        </div>

        <article class="links">
          {#each linksArray as link}
            <p>- {link}</p>
          {/each}
        </article>
      </div>
    </div>
  {/if}
</main>

<style>
  main {
    padding: 2rem;
  }
  .links {
    height: fit-content;
    margin-top: 1rem;
  }
  .uploaded-text {
    font-family: monospace;
    background-color: var(--pico-background-color);
  }
</style>
