<script lang="ts">
    type Platform = "dph" | "mod" | "smi"
    import 'tippy.js/dist/tippy.css';
	import tippy from "tippy.js";

    export let dots: Platform[];

    const formatArray = (arr: string[]): string => arr.length === 1 ? arr[0].replace('dph', 'Datapack Hub').replace('smi', 'Smithed').replace('mod', 'Modrinth') : `${arr.slice(0, -1).map(item => item.replace('dph', 'Datapack Hub').replace('smi', 'Smithed').replace('mod', 'Modrinth')).join(', ')} and ${arr.slice(-1)[0].replace('dph', 'Datapack Hub').replace('smi', 'Smithed').replace('mod', 'Modrinth')}`;

</script>

<span class="select-none inline-flex" use:tippy={{content:"Supports " + formatArray(dots)}}>
    {#if dots.includes("dph")}
    <span class="text-orange-500">●</span>
    {/if}
    {#if dots.includes("mod")}
    <span class="text-green-500">●</span>
    {/if}
    {#if dots.includes("smi")}
    <span class="text-blue-500">●</span>
    {/if}
</span>
