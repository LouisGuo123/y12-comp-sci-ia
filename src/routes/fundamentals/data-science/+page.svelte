<script lang="ts">
    import ScatterHistogram from '$lib/components/Figures/fundamentals/data-science/scatter_histogram.svelte';
    import BarChart from "$lib/images/bar-chart.png";
    import BoxAndWhiskers from "$lib/images/box-and-whiskers.png";
</script>

<p class="article-p">Data science is the field of finding, processing, analyzing and presenting data. To find patterns and trends in data, data scientists first need to gather the data. This can come in the form of numerical data, like weather readings or population demographics, ordinal numbers, like movie rankings, text, and much more. Data can come from measurement instruments, like sensors and website statistics, human observation, like during experiments, or human responses, via interviews, surveys and the like.</p>
<p class="article-p">Data then needs to be processed. This is most commonly done through code, processing the raw data for use when analyzing it. This involves converting all the data to the same format, and removing any erroneous and unuseful data points.</p>
<p class="article-p">Arguable the central part of data science is <em>exploratory data analysis</em>. This is the part where data scientists try a variety of visualization tools to see what trends they can find in the data, graphing correlations between different variables, filtering the data to find trends in subsets of data. Here's a list of visualizations you may want to use:</p>
<div class="image-wrapper">
    <img src={BarChart} alt="Bar Chart">
    <caption>Bar Chart</caption>
</div>
<div class="image-wrapper">
    <img src={BoxAndWhiskers} alt="Box and Whiskers Plot">
    <caption>Box and Whiskers Plot</caption>
</div>
<ScatterHistogram />
<style>
    .image-wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;

        color: white;
    }

    img {
        margin: 1.4em 0;
        width: calc(25vw + 180px);
        height: auto;
    }
</style>