<script>
  import "../app.css";
  const metaDescription = "Sprint point planning - Enter your team and their activities and get a rough estimate for the remainder of the week"
  const title = "Scrum Team Planning"
  const canon = "https://sprint-planning.hjf.io/"
  const meta = [
        {
          name: "description",
          content: metaDescription,
        },
        {
          name: "canonical",
          content: canon,
        },
        {
          property: "og:title",
          content: title,
        },
        {
          property: "og:description",
          content: metaDescription,
        },
        {
          property: "og:type",
          content: "website",
        },
        {
          name: "twitter:card",
          content: "summary_large_image",
        },
    /*
        {
          property: "og:image",
          content: seoImg,
        },
        {
          name: "twitter:image",
          content: seoImg,
        },
        */
        {
          name: "twitter:creator",
          content: "@hjfitz"
        },
        {
          name: "twitter:title",
          content: title,
        },
        {
          name: "twitter:description",
          content: metaDescription,
        },
      ]
</script>

<svelte:head>
  <title>{title}</title>
  {#each meta as tag}
    <meta name={tag.name} content={tag.content} />
  {/each}


</svelte:head>
<slot />
