<script lang="ts">
    import Menu from "./menu.svelte";
    import arrow from "$lib/images/down-arrow.svg";
    import { base } from '$app/paths';

    export let label = "???";
    export let href = "https://www.youtube.com/watch?v=dQw4w9WgXcQ";
    export let labelHrefList: string[][] = [];

    let isMenuVisible = false;
    function toggleMenuVisible() {
        isMenuVisible = !isMenuVisible;
    }
</script>

<li>
    <div>
        <a href="{base}{href}">{label}</a>
        <button on:click={toggleMenuVisible}><object title="dropdown-arrow" data={arrow} type="image/svg+xml"></object></button>
    </div>
    {#if isMenuVisible}
        <Menu labelHrefList={labelHrefList} />
    {/if}
</li>

<style>
    li {
        display: flex;
        flex-direction: column;

        height: calc(2.5vw + 1.8vh);
        border: none;
        margin: 0;
        z-index: 10;
    }

    div {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;

        width: auto;
        min-height: 100%;
        padding: 0 calc(.5vw + .6vh);
    }

    a {
        color: white;
    }

    button {
        display: flex;
        justify-content: center;
        align-items: center;

        width: calc(.6vw + 2vh);
        height: 100%;
        cursor: pointer;
    }

    object {
        width: 40%;
        height: auto;
        pointer-events: none;
    }

</style>