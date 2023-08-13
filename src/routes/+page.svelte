<script>
  import { writable } from 'svelte/store';
  
  let prompt = writable('');  // Add this line
  let negativePrompt;
  let aspectRatio;
  let steps; 

  if(typeof window !== 'undefined') {
    prompt = writable(localStorage.getItem('prompt') || '');  // And this line
    negativePrompt = writable(localStorage.getItem('negativePrompt') || '');
    aspectRatio = writable(localStorage.getItem('aspectRatio') || '1024x1024');
    steps = writable(localStorage.getItem('steps') || '30');
   
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
  } else {
    prompt = writable('');  // And this line
    negativePrompt = writable('');
    aspectRatio = writable('1024x1024');
    steps = writable('30');
  }
</script>

<div class="h-screen flex flex-col items-center justify-center tracking-widest text-gray-300 bg-gray-900 px-4 lg:px-0">
  <h1 class="font-mono text-2xl lg:text-4xl font-bold tracking-wider w-full text-center">BreadImagine</h1>
  <p class="mb-4 hidden md:block">Generate image using <span class="font-semibold">SDXL</span> from Stable Horde.</p>

  <div class="p-4 lg:p-10 rounded-lg bg-gray-700 mt-4 text-gray-300 w-full lg:w-3/4 mx-auto flex flex-col lg:flex-row">
    <form class="flex flex-col mt-4 space-y-4 w-full lg:w-1/2 mr-0 lg:mr-4">
      <label>
        <textarea rows="3" required 
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
            <option value="1024x1024" selected>1:1 - 1024x1024</option>
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
        
        
      </div>
      <button type="submit" class="w-full p-2 mt-4 bg-blue-500 text-white rounded hover:bg-blue-700">Generate</button>
    </form>
    
    <div class="mt-4 lg:mt-0 bg-gray-600 w-full lg:w-1/2 h-64 lg:h-full rounded-lg flex items-center justify-center text-lg font-semibold text-center">
      Image you generate will appear here
    </div>
  </div>
</div>
