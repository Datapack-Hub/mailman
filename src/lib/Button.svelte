<script lang="ts">
    export let click: string | (() => void) | (() => Promise<void>) | undefined;

    let disabled = false
    
    async function onclick() {
        if (!disabled) {
            disabled = true
            if (click !== undefined && typeof click != "string") Promise.resolve(click()).then(() => (disabled = false));
        }
    }
</script>

<button class="items-center flex space-x-1 w-fit bg-zinc-900 rounded-md h-8 p-2 px-2 {!disabled ? "cursor-pointer text-zinc-400 hover:text-zinc-300" : "cursor-wait text-zinc-700" } mb-1" on:click={onclick}><slot /></button>