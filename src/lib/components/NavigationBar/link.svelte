<script lang="ts">
    import { base } from '$app/paths';

    export let label = "???";
    export let href = "https://www.youtube.com/watch?v=dQw4w9WgXcQ";
</script>

<li>
    <a href="{base}{href}">{label}</a>
</li>

<style>
    li {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;

        width: auto;
        height: calc(2.5vw + 1.8vh);
        padding: 0 calc(.5vw + .6vh);
    }

    a {
        color: white;
    }
</style>