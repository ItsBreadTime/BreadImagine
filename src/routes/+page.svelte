<script>
    import postcss from 'postcss';
  import { writable } from 'svelte/store';
  
  let prompt = writable('');  
  let negativePrompt = writable(''); 
  let aspectRatio = writable('1024x1024');
  let steps = writable('30');
  let selectedStyle = writable('No Style');
  
  let styles = writable([
  {
    style: "No Style",
    positive: "",
    negative: ""
  },
  {
    style: "Enhance",
    positive: "breathtaking {prompt} . award-winning, professional, highly detailed",
    negative: "ugly, deformed, noisy, blurry, distorted, grainy"
  },
  {
    style: "Anime",
    positive: "anime artwork {prompt} . anime style, key visual, vibrant, studio anime,  highly detailed",
    negative: "photo, deformed, black and white, realism, disfigured, low contrast"
  },
  {
    style: "Photographic",
    positive: "cinematic photo {prompt} . 35mm photograph, film, bokeh, professional, 4k, highly detailed",
    negative: "drawing, painting, crayon, sketch, graphite, impressionist, noisy, blurry, soft, deformed, ugly"
  },
  {
    style: "Digital Art",
    positive: "concept art {prompt} . digital artwork, illustrative, painterly, matte painting, highly detailed",
    negative: "photo, photorealistic, realism, ugly"
  },
  {
    style: "Comic Book",
    positive: "comic {prompt} . graphic illustration, comic art, graphic novel art, vibrant, highly detailed",
    negative: "photograph, deformed, glitch, noisy, realistic, stock photo"
  },
  {
    style: "Fantasy Art",
    positive: "ethereal fantasy concept art of {prompt} . magnificent, celestial, ethereal, painterly, epic, majestic, magical, fantasy art, cover art, dreamy",
    negative: "photographic, realistic, realism, 35mm film, dslr, cropped, frame, text, deformed, glitch, noise, noisy, off-center, deformed, cross-eyed, closed eyes, bad anatomy, ugly, disfigured, sloppy, duplicate, mutated, black and white"
  },
  {
    style: "Analog Film",
    positive: "analog film photo {prompt} . faded film, desaturated, 35mm photo, grainy, vignette, vintage, Kodachrome, Lomography, stained, highly detailed, found footage",
    negative: "painting, drawing, illustration, glitch, deformed, mutated, cross-eyed, ugly, disfigured"
  },
  {
    style: "Neonpunk",
    positive: "neonpunk style {prompt} . cyberpunk, vaporwave, neon, vibes, vibrant, stunningly beautiful, crisp, detailed, sleek, ultramodern, magenta highlights, dark purple shadows, high contrast, cinematic, ultra detailed, intricate, professional",
    negative: "painting, drawing, illustration, glitch, deformed, mutated, cross-eyed, ugly, disfigured"
  },
  {
    style: "Isometric",
    positive: "isometric style {prompt} . vibrant, beautiful, crisp, detailed, ultra detailed, intricate",
    negative: "deformed, mutated, ugly, disfigured, blur, blurry, noise, noisy, realistic, photographic"
  },
  {
    style: "Lowpoly",
    positive: "low-poly style {prompt} . low-poly game art, polygon mesh, jagged, blocky, wireframe edges, centered composition",
    negative: "noisy, sloppy, messy, grainy, highly detailed, ultra textured, photo"
  },
  {
    style: "Cinematic",
    positive: "cinematic film still {prompt} . shallow depth of field, vignette, highly detailed, high budget Hollywood movie, bokeh, cinemascope, moody, epic, gorgeous, film grain, grainy",
    negative: "anime, cartoon, graphic, text, painting, crayon, graphite, abstract, glitch, deformed, mutated, ugly, disfigured"
  },
  {
    style: "3D-Model",
    positive: "professional 3D model {prompt} . octane render, highly detailed, volumetric, dramatic lighting",
    negative: "ugly, deformed, noisy, low poly, blurry, painting"
  },
  {
    style: "Pixel Art",
    positive: "pixel-art {prompt} . low-res, blocky, pixel art style, 8-bit graphics",
    negative: "sloppy, messy, blurry, noisy, highly detailed, ultra textured, photo, realistic"
  },
]);


  if(typeof window !== 'undefined') {
    prompt.set(localStorage.getItem('prompt') || '');
    negativePrompt.set(localStorage.getItem('negativePrompt') || '');
    aspectRatio.set(localStorage.getItem('aspectRatio') || '1024x1024');
    steps.set(localStorage.getItem('steps') || '30');

    prompt.subscribe(value => {
      localStorage.setItem('prompt', value)
    });
    negativePrompt.subscribe(value => {
      localStorage.setItem('negativePrompt', value)
    });
    aspectRatio.subscribe(value => {
      localStorage.setItem('aspectRatio', value)
    });
    steps.subscribe(value => {
      localStorage.setItem('steps', value)
    });
  }

  selectedStyle.subscribe(value => {
  let style = $styles.find(style => style.style === value);
  if (style) {
    let currentPrompt = $prompt; // Capture the current prompt
    prompt.set(style.positive.replace("{prompt}", currentPrompt)); // Replace "{prompt}" with the captured prompt
    negativePrompt.set(style.negative); // Replace the entire negativePrompt with the style's negative text
  }
});

</script>

<div class="h-screen flex flex-col items-center justify-center tracking-widest text-gray-300 bg-gray-900 px-4 lg:px-0">
  <h1 class="font-mono text-2xl lg:text-4xl font-bold tracking-wider w-full text-center">BreadImagine</h1>
  <p class="mb-4 hidden md:block">Generate image using <span class="font-semibold">SDXL</span> from Stable Horde.</p>

  <div class="p-4 lg:p-10 rounded-lg bg-gray-700 mt-4 text-gray-300 w-full lg:w-3/4 mx-auto flex flex-col lg:flex-row">
    <form class="flex flex-col mt-4 space-y-4 w-full lg:w-1/2 mr-0 lg:mr-4">
      <label>
        <textarea rows="3" required 
                  bind:value={$prompt}
                  class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full" 
                  placeholder="Prompt (Required)"></textarea>
      </label>

      <label>
        <textarea rows="3" 
                  bind:value={$negativePrompt}
                  class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full"
                  placeholder="Negative Prompt (Optional)"></textarea>
      </label>
      
      <div class="flex space-x-4">
        <label class="w-full">
          Aspect Ratio:
          <select bind:value={$aspectRatio} 
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
          <input bind:value={$steps} 
                 type="number" 
                 min="20" 
                 max="50" 
                 class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full" 
                >
        </label>
        
        <label class="w-full">
          Style:
          <select bind:value={$selectedStyle} 
                  class="p-2 rounded border border-gray-600 bg-gray-800 text-white mt-1 w-full">
            {#each $styles as style (style.style)}
              <option>{style.style}</option>
            {/each}
          </select>
        </label>
      </div>
      
      <button type="submit" class="w-full p-2 mt-4 bg-blue-500 text-white rounded hover:bg-blue-700">Generate</button>
    </form>
    
    <div class="ml-3 mt-4 lg:mt-0 bg-gray-600 w-full lg:w-1/2 h-64 lg:h-full rounded-lg flex items-center justify-center text-lg font-semibold text-center">
      Image you generate will appear here
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
</style>
