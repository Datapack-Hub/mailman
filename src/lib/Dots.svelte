<script lang="ts">
    export let dots: string[];
</script>

{#if dots.includes("dph")}
<span class="text-orange-600">●</span>
{/if}
{#if dots.includes("mod")}
<span class="text-green-600">●</span>
{/if}
{#if dots.includes("smi")}
<span class="text-blue-500">●</span>
{/if}