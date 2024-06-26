<script>
  // @ts-nocheck
  import { writable } from "svelte/store";
  import { onMount } from "svelte";
  import { scale, fade, slide } from "svelte/transition";

  let prompt = writable("");
  let negativePrompt = writable("");
  let aspectRatio = writable("1024x1024");
  let steps = writable("30");
  let selectedStyle = writable("No Style");
  let generatedImages = writable([]);
  let sampler = writable("k_dpmpp_2m");
  let cfgScale = writable("7");
  let seed = writable("");
  let model = writable("AlbedoBase XL (SDXL)");
  let showAdvancedOptions = false;
  let apiKey = "0000000000";
  let isDialogOpen = writable(!apiKey);
  let interval;
  let imageCount = writable(1);
  let kudos = writable(0);
  let models = writable([]);

  async function toggleAdvancedOptions() {
    showAdvancedOptions = !showAdvancedOptions;
    if (typeof window !== "undefined") {
      localStorage.setItem("showAdvancedOptions", showAdvancedOptions);
    }
  }

  async function toggleDialog() {
    isDialogOpen.set(!$isDialogOpen);
  }
  async function saveApiKey() {
    localStorage.setItem("apiKey", apiKey);
  }

  async function abortGeneration() {
    clearInterval(interval);
    isTaskRunning = false;
  }

  async function resetAll() {
    if (
      confirm(
        "This will reset ALL of your selected options.\nAre you sure you want to continue?",
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
      model.set("AlbedoBase XL (SDXL)");
      showAdvancedOptions = false;
      if (typeof window !== "undefined") {
        localStorage.removeItem("showAdvancedOptions");
      }
    }
  }

  onMount(async () => {
    if (typeof window !== "undefined") {
      apiKey = localStorage.getItem("apiKey") || "";
      isDialogOpen.set(!apiKey);
      await fetchModels();
      await fetchKudos();
    }
  });

  async function fetchModels() {
    try {
      const response = await fetch(
        "https://aihorde.net/api/v2/status/models?type=image&model_state=all",
        {
          headers: {
            accept: "application/json",
            "client-agent": "BreadImagine:v.0.2:(discord)bread.trademark",
          },
        },
      );

      const data = await response.text();
      const modelArray = JSON.parse(data);
      models.set(modelArray);
    } catch (error) {
      console.error("Error fetching models:", error);
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
    apiKey = localStorage.getItem("apiKey") || "";
    negativePrompt.set(localStorage.getItem("negativePrompt") || "");
    aspectRatio.set(localStorage.getItem("aspectRatio") || "1024x1024");
    steps.set(localStorage.getItem("steps") || "30");
    selectedStyle.set(localStorage.getItem("selectedStyle") || "No Style");
    sampler.set(localStorage.getItem("sampler") || "k_dpmpp_2m");
    cfgScale.set(localStorage.getItem("cfgScale") || "7");
    model.set(localStorage.getItem("model.name") || "AlbedoBase XL (SDXL)");
    imageCount.set(parseInt(localStorage.getItem("imageCount")) || 1);

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
      localStorage.setItem("model.name", value);
    });
    imageCount.subscribe((value) => {
      localStorage.setItem("imageCount", Number(value));
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
      (style) => style.style === $selectedStyle,
    );
    let positivePrompt = selectedStyleObj.positive.replace("{prompt}", $prompt);
    let negativePrompt = selectedStyleObj.negative;

    let bodyObject = {
      prompt: `${positivePrompt} ### ${negativePrompt}`,
      params: {
        cfg_scale: parseInt($cfgScale),
        sampler_name: $sampler,
        height: parseInt(height),
        width: parseInt(width),
        post_processing: [],
        steps: parseInt($steps),
        tiling: false,
        karras: true,
        hires_fix: false,
        clip_skip: 1,
        n: $imageCount,
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
        "client-agent": "BreadImagine:v0.1:(discord)bread.trademark",
        "content-type": "application/json",
        Apikey: apiKey,
      },
      body: JSON.stringify(bodyObject),
      method: "POST",
    });

    let data = await response.json();
    startCheckStatus(data.id);
    await fetchKudos();
  }

  let status = "";

  let isTaskRunning = false;

  async function startCheckStatus(id) {
    interval = setInterval(async () => {
      await checkStatus(id);
    }, 3000);
  }

  async function checkStatus(id) {
    const url = `https://aihorde.net/api/v2/generate/check/${id}`;
    let statusResponse = await fetch(url, {
      headers: {
        accept: "*/*",
        "accept-language": "en-US,en;q=0.9",
        "client-agent": "BreadImagine:v.0.2:(discord)bread.trademark",
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
          "client-agent": "BreadImagine:v.0.2:(discord)bread.trademark",
          "content-type": "application/json",
        },
      },
    );
    const data = await response.json();
    const images = data.generations.map((gen) => gen.img);
    generatedImages.set(images);
  }
  async function handleKeyDown(event) {
    if (event.ctrlKey && event.key === "Enter") {
      if (!isTaskRunning) {
        generateImage(event);
      }
      event.preventDefault();
    }
  }
  async function fetchKudos() {
    if (!apiKey) return; // Exit if API key is not available

    try {
      const response = await fetch("https://aihorde.net/api/v2/find_user", {
        headers: {
          accept: "application/json",
          apikey: apiKey,
        },
        method: "GET",
      });

      const data = await response.json();
      kudos.set(data.kudos);
    } catch (error) {
      console.error("Error fetching kudos:", error);
    }
  }
</script>

<div
  class="h-screen flex flex-col items-center justify-center text-gray-300 px-4 lg:px-0"
>
  {#if $isDialogOpen}
    <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
    <div
      role="dialog"
      class="fixed inset-0 flex items-center justify-center z-50 bg-black bg-opacity-50 transition-all"
      on:click={toggleDialog}
    >
      <div
        role="dialog"
        class="p-6 bg-slate-700 text-2xl text-slate-200 rounded-lg border-2 border-slate-600 align-middle shadow-2xl shadow-slate-700 drop-shadow-2xl"
        on:click|stopPropagation
      >
        <p>AI Horde API key</p>
        <form method="dialog" on:submit|preventDefault={saveApiKey}>
          <label>
            <input
              type="password"
              required
              bind:value={apiKey}
              class="p-2 rounded border border-slate-600 bg-slate-800 w-full plausible-event-name=prompt my-3 text-lg"
              placeholder="API Key"
            />
          </label>
          <button
            class="text-lg text-slate-200 py-1 px-4 rounded bg-sky-800 hover:bg-sky-900 transition-all w-full"
            on:click={toggleDialog}>Close</button
          >
        </form>
      </div>
    </div>
  {/if}
  <h1
    class="font-mono text-2xl lg:text-4xl font-bold tracking-wider w-full text-center"
  >
    BreadImagine
  </h1>
  <p class="mb-4 hidden md:block">
    Generate images using <span class="font-semibold">Stable Diffusion</span>,
    powered by
    <a
      href="https://github.com/Haidra-Org/AI-Horde"
      class="underline decoration-gray-500">AI Horde</a
    >.
  </p>
  <span
    class="md:fixed text-slate-400 md:top-0 md:right-0 md:p-8 text-xs md:text-s lg:text-sm"
  >
    Kudos: {$kudos}
  </span>
  <div
    class="bg-gray-700 p-4 lg:p-10 rounded-lg mt-4 text-gray-300 w-full lg:w-5/6 mx-auto flex flex-col lg:flex-row h-5/6"
  >
    <!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
    <form
      class="flex flex-col space-y-4 w-full lg:w-1/2 mr-0 lg:mr-4 overflow-y-auto"
      on:keydown={handleKeyDown}
    >
      <button
        type="submit"
        class="group plausible-event-name=generate text-lg font-medium w-full p-2 bg-blue-700 rounded hover:bg-blue-800 transition-all {isTaskRunning
          ? 'button-disabled'
          : ''}"
        disabled={isTaskRunning}
        on:click={generateImage}
        >Generate <span class=" font-light">{$imageCount} images</span><span
          class="hidden md:inline-block text-xs ml-1 font-medium bg-blue-800 p-1 rounded group-hover:bg-blue-900"
          >CTRL+Enter</span
        >
      </button>
      <label>
        <textarea
          rows="2"
          bind:value={$prompt}
          class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=prompt"
          placeholder="Prompt"
        />
      </label>

      <div>
        <label class="w-full">
          Model:
          <select
            bind:value={$model}
            class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=model"
          >
            {#each $models as model}
              <option value={model.name}>{model.name}</option>
            {/each}
          </select>
        </label>
      </div>
      <div class="flex space-x-4">
        <label class="w-full">
          Ratio:
          <select
            bind:value={$aspectRatio}
            class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=aspectRatio"
          >
            <option value="1024x1024">1:1 (Square)</option>
            <option value="1344x768">16:9 (Landscape)</option>
            <option value="768x1344">9:16 (Portrait)</option>
            <option value="1152x896">4:3 (Landscape)</option>
            <option value="896x1152">3:4 (Portrait)</option>
          </select>
        </label>
        <label class="w-full">
          Style:
          <select
            bind:value={$selectedStyle}
            class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=selectedStyle"
          >
            {#each $styles as style (style.style)}
              <option>{style.style}</option>
            {/each}
          </select>
        </label>
      </div>
      <div class="flex">
        <input
          type="range"
          min="1"
          max="10"
          bind:value={$imageCount}
          class="w-full rounded appearance-none bg-gray-800 border-gray-600 border [&::-webkit-slider-runnable-track]:rounded [&::-webkit-slider-thumb]:appearance-none [&::-webkit-slider-thumb]:h-[45px] [&::-webkit-slider-thumb]:w-[50px] [&::-webkit-slider-thumb]:rounded [&::-webkit-slider-thumb]:bg-slate-400"
        />
        <button
          on:click={toggleAdvancedOptions}
          class="p-2 rounded border border-gray-600 bg-gray-800 plausible-event-name=settings hover:bg-gray-900 transition-all"
        >
          <img src="options.svg" alt="Settings" width="48" />
        </button>
        <button
          on:click={toggleDialog}
          class="p-2 rounded border border-gray-600 bg-gray-800 plausible-event-name=settings hover:bg-gray-900 transition-all"
        >
          <img src="settings.svg" alt="Settings" width="48" />
        </button>
        <button
          on:click={resetAll}
          class="p-2 rounded border border-gray-600 bg-gray-800 plausible-event-name=resetAll hover:bg-gray-900 transition-all"
        >
          <img src="trash.svg" alt="Reset all" width="48" />
        </button>
      </div>

      {#if showAdvancedOptions}
        <div transition:slide={{ duration: 200 }}>
          <hr class="border-gray-600 mb-4 border-4 rounded" />
          <div class="flex">
            <!--<label class="flex-grow">
              <textarea
                rows="1"
                bind:value={$negativePrompt}
                class="p-2 rounded border border-gray-600 bg-gray-800 mt-1 h-16 w-full plausible-event-name=negativePrompt"
                placeholder="Negative Prompt (Optional)"
              />
            </label>-->
          </div>
          <div class="flex space-x-4">
            <label class="w-full">
              <select
                bind:value={$steps}
                class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=steps"
              >
                <option value="25">Steps: 25</option>
                <option value="30">Steps: 30</option>
                <option value="35">Steps: 35</option>
                <option value="40">Steps: 40</option>
                <option value="45">Steps: 45</option>
                <option value="50">Steps: 50</option>
              </select>
            </label>
          </div>
          <div class="flex space-x-4">
            <label class="w-full">
              <select
                bind:value={$cfgScale}
                class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=cfgScale mt-1"
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
                class="p-2 rounded border border-gray-600 bg-gray-800 w-full plausible-event-name=sampler mt-1"
              >
                <option value="k_dpmpp_2m">DPM++ 2M</option>
                <option value="k_dpmpp_2s_a">DPM++ 2S A</option>
                <option value="k_dpmpp_sde">DPM++ SDE</option>
                <option value="k_euler">Euler</option>
                <option value="k_euler_a">Euler A</option>
                <option value="DDIM">DDIM</option>
                <option value="dpmsolver">dpmsolver</option>
              </select>
            </label>
          </div>
        </div>
      {/if}
    </form>
    <div
      class="lg:ml-2 mt-4 lg:mt-0 bg-gray-600 w-full lg:w-3/4 xl:w-full h-full rounded-lg flex items-center justify-center text-lg font-semibold text-center overflow-auto"
    >
      {#if isTaskRunning}
        <div>
          <div class="loading-container" transition:scale={{ duration: 200 }}>
            <img src="loading.svg" alt="Loading" width="100px" height="100px" />
          </div>
          <p transition:scale={{ duration: 200 }}>Wait time: {status}</p>
          <button
            transition:scale={{ duration: 200 }}
            type="button"
            class="group plausible-event-name=abort text-lg font-medium w-full p-2 mt-4 bg-cyan-700 rounded hover:bg-cyan-800 transition-all {isTaskRunning
              ? ''
              : 'button-disabled'}"
            disabled={!isTaskRunning}
            on:click={abortGeneration}
          >
            Abort Generation
          </button>
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

<style lang="postcss">
  :global(html) {
    background: theme(colors.gray.900);
  }

  ::-webkit-scrollbar {
    width: 10px;
  }

  ::-webkit-scrollbar-track {
    background: theme(colors.slate.800);
  }

  ::-webkit-scrollbar-thumb {
    background: theme(colors.slate.600);
    border: 1px solid theme(colors.neutral.800);
  }

  ::-webkit-scrollbar-thumb:hover {
    background: theme(colors.slate.500);
  }

  .loading-container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }

  .loading-container img {
    width: 100px;
    height: 100px;
  }

  .image-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
  }
  .generated-image {
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
    flex-grow: 0;
    flex-shrink: 0;
    padding: 5px;
  }
  .button-disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
