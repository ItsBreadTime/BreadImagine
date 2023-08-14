<script>
  // @ts-nocheck
  import { writable } from "svelte/store";

  let prompt = writable("");
  let negativePrompt = writable("");
  let aspectRatio = writable("1024x1024");
  let steps = writable("30");
  let selectedStyle = writable("No Style");
  let generatedImages = writable([]);

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
  }

  async function generateImage(e) {
    e.preventDefault();

    let [width, height] = $aspectRatio.split("x");

    let seed = Math.floor(Math.random() * 1000000).toString();

    let selectedStyleObj = $styles.find(
      (style) => style.style === $selectedStyle
    );
    let positivePrompt = selectedStyleObj.positive.replace("{prompt}", $prompt);
    let negativePrompt = selectedStyleObj.negative;

    let bodyObject = {
      prompt: `${positivePrompt} ### ${negativePrompt}`,
      params: {
        cfg_scale: 7,
        seed: seed, // Use the random seed value
        sampler_name: "k_euler",
        height: parseInt(height),
        width: parseInt(width),
        post_processing: [],
        steps: parseInt($steps),
        tiling: false,
        karras: true,
        hires_fix: false,
        clip_skip: 1,
        n: 2,
      },
      nsfw: true,
      censor_nsfw: false,
      trusted_workers: true,
      models: ["SDXL_beta::stability.ai#6901"],
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
        apikey: "f326232c-23a7-4cee-930a-fe9063a142db",
        "client-agent": "BreadImagine:v0.1:(discord)bread.trademark",
        "content-type": "application/json",
      },
      body: JSON.stringify(bodyObject),
      method: "POST",
    });

    let data = await response.json();
    startCheckStatus(data.id);
    isTaskRunning = true;
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
    Generate image using <span class="font-semibold">SDXL</span> from Stable Horde.
  </p>

  <div
    class="p-4 lg:p-10 rounded-lg bg-gray-700 mt-4 text-gray-300 w-full lg:w-3/4 mx-auto flex flex-col lg:flex-row h-5/6"
  >
    <form class="flex flex-col mt-4 space-y-4 w-full lg:w-1/2 mr-0 lg:mr-4">
      <label>
        <textarea
          rows="3"
          required
          bind:value={$prompt}
          class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-0 h-full w-full"
          placeholder="Prompt (Required)"
        />
      </label>

      <label>
        <textarea
          rows="3"
          bind:value={$negativePrompt}
          class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 max-h-16 w-full"
          placeholder="Negative Prompt (Optional - Will get removed if style is selected)"
        />
      </label>

      <div class="flex space-x-4">
        <label class="w-full">
          Aspect Ratio:
          <select
            bind:value={$aspectRatio}
            class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full"
          >
            <option value="1024x1024">1:1 - 1024x1024</option>
            <option value="1344x768">16:9 - 1344x768</option>
            <option value="768x1344">9:16 - 768x1344</option>
            <option value="1152x896">4:3 - 1152x896</option>
            <option value="896x1152">3:4 - 896x1152</option>
          </select>
        </label>

        <label class="w-full">
          Steps:
          <input
            bind:value={$steps}
            type="number"
            min="20"
            max="50"
            class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full"
          />
        </label>

        <label class="w-full">
          Style:
          <select
            bind:value={$selectedStyle}
            class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full"
          >
            {#each $styles as style (style.style)}
              <option>{style.style}</option>
            {/each}
          </select>
        </label>
      </div>

      <button
        type="submit"
        class="w-full p-2 mt-4 bg-blue-500 text-white rounded hover:bg-blue-700"
        on:click={generateImage}>Generate</button
      >
    </form>

    <div
      class="lg:ml-3 mt-4 lg:mt-0 bg-gray-600 w-full lg:w-1/2 h-64 lg:h-full rounded-lg flex items-center justify-center text-lg font-semibold text-center"
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
</style>
