<script lang="ts">
import Dots from "$lib/Dots.svelte";
import autoAnimate from "@formkit/auto-animate";

let page = 0;

let fileInput: HTMLInputElement;
</script>

<div class="flex justify-around items-center min-h-screen h-full py-3">
    <div class="flex flex-col items-center space-y-10">
        <div class="flex flex-col items-center space-y-2">
            <div class="flex items-center">
                <img src="/dph.png" class="h-20 mr-2" alt="logo">
                <h1><b class="text-5xl md:text-7xl">Mailman</b> <span class="text-base hidden md:inline">by Datapack Hub</span></h1>
            </div>
            <p>Upload your datapack versions to 
                <a class="text-orange-600 hover:underline" href="https://datapackhub.net">Datapack Hub</a>, 
                <a class="text-green-600 hover:underline" href="https://modrinth.com">Modrinth</a>, and 
                <a class="text-blue-500 hover:underline" href="https://smithed.dev">Smithed</a> at the same time!</p>
        </div>
        <div>
            <div class="bg-zinc-950 rounded-xl border-zinc-800 border-2 p-3" use:autoAnimate>
                <input name="upload" id="upload" type="file" class="hidden" on:change={() => page = 1} />
                {#if page == 0}
                <label for="upload" class="cursor-pointer">
                    <p class="p-5 px-10"><b>Click</b> to upload a datapack.</p>
                </label>
                {:else if page == 1}
                <p class="font-light text-sm mb-2"><b class="font-bold text-base">Authenticate and choose your projects. </b>If you leave one blank, the version won't be posted to that site.</p>
                <div class="flex flex-col md:flex-row justify-around space-y-2 md:space-y-0 md:space-x-2 w-full">
                    <div class="bg-zinc-950 w-full rounded-xl border-zinc-800 border-2 p-3 space-y-2">
                        <div class="flex items-center space-x-2">
                            <img src="/dph.png" class="h-8" alt="logo">
                            <b>Datapack Hub</b>
                        </div>
                        <input 
                        class="focus:outline-orange-600" 
                        placeholder="datapackhub.net API token" />
                    </div>
                    <div class="bg-zinc-950 w-full rounded-xl border-zinc-800 border-2 p-3 space-y-2">
                        <div class="flex items-center space-x-2">
                            <img src="/modrinth.svg" class="h-8" alt="logo">
                            <b>Modrinth</b>
                        </div>
                        <input 
                        class="focus:outline-green-600" 
                        placeholder="modrinth.com API token" />
                    </div>
                    <div class="bg-zinc-950 w-full rounded-xl border-zinc-800 border-2 p-3 space-y-2">
                        <div class="flex items-center space-x-2">
                            <img src="/smithed.svg" class="h-8 rounded-full bg-[#1b48c4]" alt="logo">
                            <b>Smithed</b>
                        </div>
                        <input 
                        class="focus:outline-blue-500" 
                        placeholder="smithed.dev API token" />
                    </div>
                </div>
                {:else if page == 2}
                <h1 class="font-bold">Enter version details. </h1>

                <h2 class="text-sm mt-3">Title <Dots dots={["mod","dph"]} /></h2>
                <input placeholder="The Snails Update" />

                <h2 class="text-sm mt-3">Version Code <Dots dots={["mod","smi","dph"]} /></h2>
                <input placeholder="v1.2.3" />

                <h2 class="text-sm mt-3">Changelog <Dots dots={["mod","dph"]} /></h2>
                <textarea placeholder="This version adds a snail you can worship, among other bug fixes" />
                {/if}
                {#if page != 0}
                <p class="text-sm text-zinc-500 mt-2">Nothing entered here is sent to us. All code is client-side, which means that we can't view anything you enter here, including API tokens.</p>
                {/if}
            </div>
            {#if page != 0}
            <div class="mt-1">
                <button class="bg-zinc-700 rounded-md p-1 px-2 text-lg" on:click={() => {
                    if(page > 0) page--
                    if(page == 0) fileInput.files = null;
                }}>Back</button>
                <button class="bg-orange-600 rounded-md p-1 px-2 float-right text-lg font-semibold" on:click={() => page++}>Next</button>
            </div>
            {/if}
        </div>
    </div>
</div>