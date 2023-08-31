<script>
  // @ts-nocheck
  import { writable } from "svelte/store";
  import { Turnstile } from "svelte-turnstile";

  let prompt = writable("");
  let negativePrompt = writable("");
  let aspectRatio = writable("1024x1024");
  let steps = writable("30");
  let selectedStyle = writable("No Style");
  let generatedImages = writable([]);
  let sampler = writable("k_dpmpp_2m");
  let cfgScale = writable("7");
  let seed = writable("");
  let model = writable("SDXL_beta::stability.ai#6901");
  let showAdvancedOptions = false;

  function toggleAdvancedOptions() {
    showAdvancedOptions = !showAdvancedOptions;
    if (typeof window !== "undefined") {
      localStorage.setItem("showAdvancedOptions", showAdvancedOptions);
    }
  }

  function resetAll() {
    if (
      confirm(
        "This will reset ALL of your selected options.\nAre you sure you want to continue?"
      )
    ) {
      prompt.set("");
      negativePrompt.set("");
      aspectRatio.set("1024x1024");
      steps.set("30");
      selectedStyle.set("No Style");
      generatedImages.set([]);
      sampler.set("k_dpmpp_2m");
      cfgScale.set("7");
      seed.set("");
      model.set("SDXL_beta::stability.ai#6901");
      showAdvancedOptions = false;
      if (typeof window !== "undefined") {
        localStorage.removeItem("showAdvancedOptions");
      }
    }
  }

  let styles = writable([
    {
      style: "No Style",
      positive: "{prompt}",
      negative: "",
    },
    {
      style: "Enhance",
      positive:
        "breathtaking {prompt} . award-winning, professional, highly detailed",
      negative: "ugly, deformed, noisy, blurry, distorted, grainy",
    },
    {
      style: "Anime",
      positive:
        "anime artwork {prompt} . anime style, key visual, vibrant, studio anime,  highly detailed",
      negative:
        "photo, deformed, black and white, realism, disfigured, low contrast",
    },
    {
      style: "Photographic",
      positive:
        "cinematic photo {prompt} . 35mm photograph, film, bokeh, professional, 4k, highly detailed",
      negative:
        "drawing, painting, crayon, sketch, graphite, impressionist, noisy, blurry, soft, deformed, ugly",
    },
    {
      style: "Digital Art",
      positive:
        "concept art {prompt} . digital artwork, illustrative, painterly, matte painting, highly detailed",
      negative: "photo, photorealistic, realism, ugly",
    },
    {
      style: "Comic Book",
      positive:
        "comic {prompt} . graphic illustration, comic art, graphic novel art, vibrant, highly detailed",
      negative: "photograph, deformed, glitch, noisy, realistic, stock photo",
    },
    {
      style: "Fantasy Art",
      positive:
        "ethereal fantasy concept art of {prompt} . magnificent, celestial, ethereal, painterly, epic, majestic, magical, fantasy art, cover art, dreamy",
      negative:
        "photographic, realistic, realism, 35mm film, dslr, cropped, frame, text, deformed, glitch, noise, noisy, off-center, deformed, cross-eyed, closed eyes, bad anatomy, ugly, disfigured, sloppy, duplicate, mutated, black and white",
    },
    {
      style: "Analog Film",
      positive:
        "analog film photo {prompt} . faded film, desaturated, 35mm photo, grainy, vignette, vintage, Kodachrome, Lomography, stained, highly detailed, found footage",
      negative:
        "painting, drawing, illustration, glitch, deformed, mutated, cross-eyed, ugly, disfigured",
    },
    {
      style: "Neonpunk",
      positive:
        "neonpunk style {prompt} . cyberpunk, vaporwave, neon, vibes, vibrant, stunningly beautiful, crisp, detailed, sleek, ultramodern, magenta highlights, dark purple shadows, high contrast, cinematic, ultra detailed, intricate, professional",
      negative:
        "painting, drawing, illustration, glitch, deformed, mutated, cross-eyed, ugly, disfigured",
    },
    {
      style: "Isometric",
      positive:
        "isometric style {prompt} . vibrant, beautiful, crisp, detailed, ultra detailed, intricate",
      negative:
        "deformed, mutated, ugly, disfigured, blur, blurry, noise, noisy, realistic, photographic",
    },
    {
      style: "Lowpoly",
      positive:
        "low-poly style {prompt} . low-poly game art, polygon mesh, jagged, blocky, wireframe edges, centered composition",
      negative:
        "noisy, sloppy, messy, grainy, highly detailed, ultra textured, photo",
    },
    {
      style: "Cinematic",
      positive:
        "cinematic film still {prompt} . shallow depth of field, vignette, highly detailed, high budget Hollywood movie, bokeh, cinemascope, moody, epic, gorgeous, film grain, grainy",
      negative:
        "anime, cartoon, graphic, text, painting, crayon, graphite, abstract, glitch, deformed, mutated, ugly, disfigured",
    },
    {
      style: "3D-Model",
      positive:
        "professional 3D model {prompt} . octane render, highly detailed, volumetric, dramatic lighting",
      negative: "ugly, deformed, noisy, low poly, blurry, painting",
    },
    {
      style: "Pixel Art",
      positive:
        "pixel-art {prompt} . low-res, blocky, pixel art style, 8-bit graphics",
      negative:
        "sloppy, messy, blurry, noisy, highly detailed, ultra textured, photo, realistic",
    },
  ]);

  if (typeof window !== "undefined") {
    negativePrompt.set(localStorage.getItem("negativePrompt") || "");
    aspectRatio.set(localStorage.getItem("aspectRatio") || "1024x1024");
    steps.set(localStorage.getItem("steps") || "30");
    selectedStyle.set(localStorage.getItem("selectedStyle") || "No Style");
    sampler.set(localStorage.getItem("sampler") || "k_dpmpp_2m");
    cfgScale.set(localStorage.getItem("cfgScale") || "7");
    model.set(localStorage.getItem("model") || "SDXL_beta::stability.ai#6901");

    negativePrompt.subscribe((value) => {
      localStorage.setItem("negativePrompt", value);
    });
    aspectRatio.subscribe((value) => {
      localStorage.setItem("aspectRatio", value);
    });
    steps.subscribe((value) => {
      localStorage.setItem("steps", value);
    });
    selectedStyle.subscribe((value) => {
      localStorage.setItem("selectedStyle", value);
    });
    sampler.subscribe((value) => {
      localStorage.setItem("sampler", value);
    });
    cfgScale.subscribe((value) => {
      localStorage.setItem("cfgScale", value);
    });
    seed.subscribe((value) => {
      localStorage.setItem("seed", value);
    });
    model.subscribe((value) => {
      localStorage.setItem("model", value);
    });

    const storedOption = localStorage.getItem("showAdvancedOptions");
    showAdvancedOptions = storedOption === "true" ? true : false;
  }

  async function generateImage(e) {
    e.preventDefault();

    isTaskRunning = true;

    let [width, height] = $aspectRatio.split("x");

    let seedValue = $seed || Math.floor(Math.random() * 1000000).toString();

    let selectedStyleObj = $styles.find(
      (style) => style.style === $selectedStyle
    );
    let positivePrompt = selectedStyleObj.positive.replace("{prompt}", $prompt);
    let negativePrompt = selectedStyleObj.negative;

    let bodyObject = {
      prompt: `${positivePrompt} ### ${negativePrompt}`,
      params: {
        cfg_scale: parseInt($cfgScale),
        seed: seedValue,
        sampler_name: $sampler,
        height: parseInt(height),
        width: parseInt(width),
        post_processing: [],
        steps: parseInt($steps),
        tiling: false,
        karras: true,
        hires_fix: $model !== "SDXL_beta::stability.ai#6901",
        clip_skip: 1,
        n: 2,
      },
      nsfw: true,
      censor_nsfw: false,
      trusted_workers: true,
      models: [$model],
      r2: true,
      replacement_filter: true,
      shared: false,
      slow_workers: false,
      dry_run: false,
    };

    let response = await fetch("https://aihorde.net/api/v2/generate/async", {
      headers: {
        accept: "*/*",
        "accept-language": "en-US,en;q=0.9",
        apikey: import.meta.env.VITE_PUBLIC_API_KEY,
        "client-agent": "BreadImagine:v0.1:(discord)bread.trademark",
        "content-type": "application/json",
      },
      body: JSON.stringify(bodyObject),
      method: "POST",
    });

    let data = await response.json();
    startCheckStatus(data.id);
  }

  let loadingIcon = `<img src="loading.svg">`;
  let status = "";

  let interval;
  let isTaskRunning = false;

  function startCheckStatus(id) {
    interval = setInterval(async () => {
      await checkStatus(id);
    }, 1000);
  }

  async function checkStatus(id) {
    const url = `https://aihorde.net/api/v2/generate/check/${id}`;
    let statusResponse = await fetch(url, {
      headers: {
        accept: "*/*",
        "accept-language": "en-US,en;q=0.9",
        "client-agent": "BreadImagine:v.0.1:(discord)bread.trademark",
        "content-type": "application/json",
      },
    });

    let data = await statusResponse.json();
    status = data?.wait_time || "";
    if (data?.done == true || data?.faulted == true) {
      clearInterval(interval);
      isTaskRunning = false;
      await fetchGeneratedImages(id); // fetch images once task is completed
    }
  }

  async function fetchGeneratedImages(id) {
    const response = await fetch(
      `https://aihorde.net/api/v2/generate/status/${id}`,
      {
        headers: {
          accept: "*/*",
          "accept-language": "en-US,en;q=0.9",
          "client-agent": "BreadImagine:v.0.1:(discord)bread.trademark",
          "content-type": "application/json",
        },
      }
    );
    const data = await response.json();
    const images = data.generations.map((gen) => gen.img);
    generatedImages.set(images);
  }
  function handleKeyDown(event) {
    // Checking the Ctrl + Enter combination
    if (event.ctrlKey && event.key === "Enter") {
      if (!isTaskRunning) {
        generateImage(event);
      }
      // prevent the form from submitting and refreshing the page
      event.preventDefault();
    }
  }
</script>

<div
  class="h-screen flex flex-col items-center justify-center tracking-widest text-gray-300 bg-gray-900 px-4 lg:px-0"
>
  <h1
    class="font-mono text-2xl lg:text-4xl font-bold tracking-wider w-full text-center"
  >
    BreadImagine
  </h1>
  <p class="mb-4 hidden md:block">
    Generate images using <span class="font-semibold">SDXL</span> from Stable Horde.
  </p>

  <div
    class="p-4 lg:p-10 rounded-lg bg-gray-700 mt-4 text-gray-300 w-full lg:w-3/4 mx-auto flex flex-col lg:flex-row h-5/6"
  >
    <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
    <form
      class="flex flex-col mt-4 space-y-4 w-full lg:w-1/2 mr-0 lg:mr-4"
      on:keydown={handleKeyDown}
    >
      <label>
        <textarea
          rows="3"
          required
          bind:value={$prompt}
          class="p-2 rounded border border-gray-600 bg-gray-800 mt-0 h-full w-full plausible-event-name=prompt"
          placeholder="Prompt"
          maxlength="999"
        />
      </label>

      <div class="flex">
        <label class="flex-grow">
          <textarea
            rows="1"
            bind:value={$negativePrompt}
            class="p-2 rounded border border-gray-600 bg-gray-800 mt-1 h-16 w-full plausible-event-name=negativePrompt"
            placeholder="Negative Prompt (Optional)"
          />
        </label>
      </div>
      <div class="flex space-x-4">
        <button
          on:click={toggleAdvancedOptions}
          class="p-2 rounded border border-gray-600 bg-gray-800 mt-1 w-full plausible-event-name=toggleAdvancedOptions"
          >{showAdvancedOptions ? "Hide" : "Show"} Advanced Options</button
        >
        <button
          on:click={resetAll}
          class="p-2 rounded border border-gray-600 bg-gray-800 mt-1 plausible-event-name=resetAll"
        >
          <img src="trash.svg" alt="Reset all" class="w-6 h-6" />
        </button>
      </div>

      <div class="flex space-x-4">
        <label class="w-full">
          Ratio:
          <select
            bind:value={$aspectRatio}
            class="p-2 rounded border border-gray-600 bg-gray-800 mt-2 w-full plausible-event-name=aspectRatio"
          >
            <option value="1024x1024">1:1</option>
            <option value="1344x768">16:9</option>
            <option value="768x1344">9:16</option>
            <option value="1152x896">4:3</option>
            <option value="896x1152">3:4</option>
          </select>
        </label>
        <label class="w-full">
          Style:
          <select
            bind:value={$selectedStyle}
            class="p-2 rounded border border-gray-600 bg-gray-800 mt-2 w-full plausible-event-name=selectedStyle"
          >
            {#each $styles as style (style.style)}
              <option>{style.style}</option>
            {/each}
          </select>
        </label>
      </div>

      {#if showAdvancedOptions}
        <hr class="border-gray-600 mb-4 border-4 rounded" />
        <div class="flex space-x-4">
          <label class="w-full">
            <select
              bind:value={$steps}
              class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=steps"
            >
              <option value="30">Steps: 30</option>
              <option value="35">Steps: 35</option>
              <option value="40">Steps: 40</option>
              <option value="45">Steps: 45</option>
              <option value="50">Steps: 50</option>
            </select>
          </label>
          <label class="w-full">
            <select
              bind:value={$model}
              class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=model"
            >
              <option value="SDXL_beta::stability.ai#6901">SDXL</option>
              <option value="ICBINP - I Can't Believe It's Not Photography"
                >ICBINP</option
              >
              <option value="Dreamshaper">Dreamshaper</option>
              <option value="Deliberate">Deliberate</option>
              <option value="Anything Diffusion">Anything Diffusion</option>
            </select>
          </label>
        </div>
        <div class="flex space-x-4">
          <label class="w-full">
            <select
              bind:value={$cfgScale}
              class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=cfgScale"
            >
              <option value="5">CFG: 5</option>
              <option value="6">CFG: 6</option>
              <option value="7">CFG: 7</option>
              <option value="8">CFG: 8</option>
              <option value="9">CFG: 9</option>
            </select>
          </label>
          <label class="w-full">
            <select
              bind:value={$sampler}
              class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=sampler"
            >
              <option value="k_dpmpp_2m">DPM++ 2M</option>
              <option value="k_dpmpp_sde">DPM++ SDE</option>
              <option value="k_euler">Euler</option>
              <option value="k_euler_a">Euler A</option>
            </select>
          </label>
        </div>
        <Turnstile siteKey="0x4AAAAAAAJArzFOsmSXpPOB" theme="dark" appearance="interaction-only"/>
      {/if}
      <button
        type="submit"
        class="group plausible-event-name=generate text-lg font-medium w-full p-2 mt-4 bg-blue-700 rounded hover:bg-blue-800 {isTaskRunning
          ? 'button-disabled'
          : ''}"
        disabled={isTaskRunning}
        on:click={generateImage}
        >Generate <span
          class="hidden md:inline-block text-xs ml-1 font-medium bg-blue-800 p-1 rounded group-hover:bg-blue-900"
          >CTRL+Enter</span
        >
      </button>
    </form>

    <div
      class="lg:ml-2 mt-4 lg:mt-0 bg-gray-600 w-full lg:w-3/5 h-64 lg:h-full rounded-lg flex items-center justify-center text-lg font-semibold text-center"
    >
      {#if isTaskRunning}
        <div>
          <div class="loading-container">
            <img src="loading.svg" alt="Loading" width="100px" height="100px" />
          </div>
          <p>Waiting time: {status}</p>
        </div>
      {:else if $generatedImages.length > 0}
        <div class="image-container">
          {#each $generatedImages as img}
            <!-- svelte-ignore a11y-img-redundant-alt -->
            <img src={img} alt="Generated Image" class="generated-image" />
          {/each}
        </div>
      {:else}
        Image you generate will appear here
      {/if}
    </div>
  </div>
</div>

<style>
  ::-webkit-scrollbar {
    width: 10px;
  }

  /* Track */
  ::-webkit-scrollbar-track {
    background: #f1f1f1;
  }

  /* Handle */
  ::-webkit-scrollbar-thumb {
    background: #888;
  }

  /* Handle on hover */
  ::-webkit-scrollbar-thumb:hover {
    background: #555;
  }

  .loading-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }

  .loading-container img {
    width: 100px; /* adjust as needed */
    height: 100px; /* adjust as needed */
  }

  .image-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    overflow: auto; /* This allows the div to maintain its size and provides scrollbar when images overflow. */
  }
  .generated-image {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
    flex-grow: 0; /* Prevent images from stretching */
    flex-shrink: 0; /* Prevent images from shrinking */
    padding: 5px; /* Add spacing between images */
  }
  .button-disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
