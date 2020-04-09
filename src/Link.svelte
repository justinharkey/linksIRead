<script>
    export let url;
    export let date;

    import moment from 'moment';
    import { decodeHtmlEntities } from './utils.js';

    const formatDate = (date) => {
		return moment(date).format('MMMM Do, YYYY');
	}

    function parseOpenGraphData(htmlData) {
        // might be able to use Pocket API for all this
        const metaTagRegex = /<meta([^>]*)\/?>/gi;
        let metaTagMatches = [...htmlData.matchAll(metaTagRegex)];

        let ogTags = {};
        if (metaTagMatches.length > 0) {
            metaTagMatches.map((metaTag) => {
                if (metaTag[0].includes('og:')) {
                    const propertySearch = /property="([^\"]*)"/gi;
                    const contentSearch = /content="([^\"]*)"/gi;
                    const property = propertySearch.exec(metaTag[0]);
                    const content = contentSearch.exec(metaTag[0]);
                    ogTags[property[1]] = content[1];
                }
            });
        }

        if (!ogTags["og:title"]) {
            const titleTagRegex = /<title>([^<]*)<\/title>/gi;
            const titleTag = titleTagRegex.exec(htmlData);
            ogTags["og:title"] = titleTag[1];
        }

		return ogTags;
	}
    
    const getOpenGraphData = (async () => {
        const response = await fetch(`https://cors-anywhere.herokuapp.com/${url}`, { cache: 'force-cache' });
        const htmlData = await response.text();
        return await parseOpenGraphData(htmlData);
    })();
</script>

<style>
.link {
    display: block;
    text-decoration: none;
    color: inherit;
    margin-bottom: 20px;
    border: 1px solid #ddd;
    border-radius: 3px;
}
.link__container {
    background-repeat: no-repeat;
    background-size: 400px;
    padding: 14px 24px 14px 240px;
    position: relative;
    z-index: 0;
}
.link__container::after {
    content: '';
    position: absolute;
    height: 100%;
    width: calc(100% - 220px);
    top: 0;
    left: 220px;
    display: block;
    background-color: #fff;
    z-index: -1;
}
.link__title {
    margin: 0 0 4px;
    padding: 0;
    font-size: 16px;
    line-height: 1.3;
    color: #369;
}
.link:visited .link__title {
    color: darkslateblue;
}
.link__description {
    margin: 0 0 6px;
    padding: 0;
    font-size: 12px;
    line-height: 1.5;
}
.link__date {
    font-size: 10px;
    font-style: italic;
    color: #555;
}
</style>

<a class="link" href="{url}" target="_blank">
    {#await getOpenGraphData}
        <div class="link__container">...loading open graph data</div>
    {:then data}
        <div class="link__container" style="background-image: url({data["og:image"]})">
            <h1 class="link__title">
                {#if data["og:title"]}
                   {decodeHtmlEntities(data["og:title"])}
                {:else}
                    {url}
                {/if}
                {#if data["og:site_name"]}
                    - {data["og:site_name"]}
                {/if}
            </h1>
            {#if data["og:description"]}<p class="link__description">{decodeHtmlEntities(data["og:description"])}</p>{/if}
            <div class="link__date">{formatDate(date)}</div>
        </div>
    {:catch error}
        <div class="link__container">
            <h1 class="link__title">{url}</h1>
            <div class="link__date">{formatDate(date)}</div>
        </div>
    {/await}
</a>