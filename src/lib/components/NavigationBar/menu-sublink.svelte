<script>
    import { base } from "$app/paths";

    export let label = "???";
    export let href = "https://www.youtube.com/watch?v=dQw4w9WgXcQ";
</script>

<li>
    <a href={href}>{label}</a>
</li>

<style>
    li {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;

        width: 100%;
        padding: 0 10%;
        min-height: 8vh;
        box-sizing: border-box;
    }

    a {
        width: 100%;
        color: white;
        text-align: left;
    }
</style>