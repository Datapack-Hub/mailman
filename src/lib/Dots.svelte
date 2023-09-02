<script lang="ts">
    export let dots: string[];
</script>

<span class="select-none inline-flex">
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
