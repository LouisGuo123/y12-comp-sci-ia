<script lang="ts">
    import NavigationBar from '$lib/components/NavigationBar/navigation-bar.svelte';
    import "$lib/styles/article.css";
</script>

<NavigationBar />

<main>
    <article>
        <slot></slot>
    </article>
</main>