<script lang="ts">
    import Dropzone from "svelte-file-dropzone/Dropzone.svelte";

    function dropHandler(ev: DragEvent) {
        console.log("File(s) dropped");

        // Prevent default behavior (Prevent file from being opened)
        ev.preventDefault();

        if (ev.dataTransfer?.items) {
            // Use DataTransferItemList interface to access the file(s)
            [...ev.dataTransfer.items].forEach((item, i) => {
            // If dropped items aren't files, reject them
            if (item.kind === "file") {
                const file = item.getAsFile();
                console.log(`… file[${i}].name = ${file.name}`);
            }
            });
        } else {
            // Use DataTransfer interface to access the file(s)
            [...ev.dataTransfer.files].forEach((file, i) => {
            console.log(`… file[${i}].name = ${file.name}`);
            });
        }
        }
</script>

<div class="flex h-screen justify-around items-center">
    <div class="flex flex-col items-center space-y-10">
        <div class="flex flex-col items-center space-y-2">
            <div class="flex items-center">
                <img src="/dph.png" class="h-20 mr-2">
                <h1><b class="text-7xl">Mailman</b> <span class="text-base">by Datapack Hub</span></h1>
            </div>
            <p>Upload your datapack versions to 
                <a class="text-orange-600 hover:underline" href="https://datapackhub.net">Datapack Hub</a>, 
                <a class="text-green-600 hover:underline" href="https://modrinth.com">Modrinth</a>, and 
                <a class="text-blue-500 hover:underline" href="https://smithed.dev">Smithed</a> at the same time!</p>
        </div>
        <input name="upload" id="upload" type="file" class="hidden" />
        <label for="upload" class="bg-zinc-950 rounded-xl border-zinc-800 border-2 p-3 space-y-2 cursor-pointer">
            <p class="p-5 px-10"><b>Click</b> to upload a datapack.</p>
            <!-- <div class="flex justify-around space-x-2 w-fit">
                <div class="bg-zinc-950 w-96 rounded-xl border-zinc-800 border-2 p-3 space-y-2">
                    <div class="flex items-center space-x-2">
                        <img src="/dph.png" class="h-8">
                        <b>Datapack Hub</b>
                    </div>
                    <input 
                    class="w-full bg-zinc-900 rounded-md h-8 focus:outline-orange-600 focus:outline focus:outline-2 p-1 placeholder:text-zinc-600" 
                    placeholder="datapackhub.net API token" />
                </div>
                <div class="bg-zinc-950 w-96 rounded-xl border-zinc-800 border-2 p-3 space-y-2">
                    <div class="flex items-center space-x-2">
                        <img src="/modrinth.svg" class="h-8">
                        <b>Modrinth</b>
                    </div>
                    <input 
                    class="w-full bg-zinc-900 rounded-md h-8 focus:outline-green-600 focus:outline focus:outline-2 p-1 placeholder:text-zinc-600" 
                    placeholder="modrinth.com API token" />
                </div>
                <div class="bg-zinc-950 w-96 rounded-xl border-zinc-800 border-2 p-3 space-y-2">
                    <div class="flex items-center space-x-2">
                        <img src="/smithed.svg" class="h-8 rounded-full bg-[#1b48c4]">
                        <b>Smithed</b>
                    </div>
                    <input 
                    class="w-full bg-zinc-900 rounded-md h-8 focus:outline-blue-500 focus:outline focus:outline-2 p-1 placeholder:text-zinc-600" 
                    placeholder="smithed.dev API token" />
                </div>
            </div> -->
        </label>
    </div>
</div>
