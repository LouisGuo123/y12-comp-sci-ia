<script lang="ts">
    import Link from './link.svelte';
    import MenuLink from './menu-link.svelte';
    import img from '$lib/images/banner-dark.svg';
    import { base } from '$app/paths';
</script>

<nav>
    <a href="{base}/"><object title="site-banner" data={img} type="image/svg+xml"></object></a>
    <ul>
        <MenuLink label="Fundamentals" href="/fundamentals" labelHrefList={[["Linear Algebra", "/fundamentals/linear-algebra"], ["Calculus", "/fundamentals/calculus"]]} />
        <Link label="Resources" href="/resources" />
        <Link label="Further Reading" href="/further-reading" />
    </ul>
</nav>

<style>
    nav {
        display: flex;
        align-items: flex-start;
        flex-direction: row;
        flex-wrap: wrap;

        background-color: #312b3bff;
        padding: calc((3vw + 12vh) * .2) 0;
        border-bottom: .5vh solid white;
    }

    a {
        display: flex;
        align-items: flex-end;

        height: calc(2.6vw + 10vh);
        padding-left: 2%;
    }

    object {
        width: auto;
        height: 100%;

        pointer-events: none;
    }

    ul {
        display: flex;
        flex-direction: row;
        flex-wrap: wrap;

        padding: 0 calc(.5vw + .6vh);
        color: white;
        font: normal 100 calc(.13vw + 1.6vh) 'Roboto', sans-serif;
        letter-spacing: .05em;
    }
</style>