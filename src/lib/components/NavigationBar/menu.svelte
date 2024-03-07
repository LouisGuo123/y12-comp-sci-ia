<script lang="ts">
    import MenuSublink from "./menu-sublink.svelte";

    export let labelHrefList: string[][] = [];
</script>

<div>
    <ul>
        {#each labelHrefList as [label, href], i (i)}
            <MenuSublink label={label} href={href} />
        {/each}
    </ul>
</div>

<style>
    ul {
        display: flex;
        flex-direction: column;
        
        background: #2a2632ff;
        border-radius: 1vh;
    }
</style>