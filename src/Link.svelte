<script>
    export let url;
    export let date;

    import moment from 'moment';
    import { decodeHtmlEntities } from './utils.js';

    const formatDate = (date) => {
		return moment(date).format('MMMM Do, YYYY');
	}

    function parseOpenGraphData(htmlData) {
        const regex = /<meta(?=.*property="og:([^\"]*)")(?=.*content="([^\"]*)").*\/?>/gi;
        // const regex = /property="(og:[^\"]*)"(?= content="([^\"]*)")/gi;
        let matchesArray = [...htmlData.matchAll(regex)];
		let ogTags = {};
		if (matchesArray.length > 0) {
			matchesArray.map((match) => {
				ogTags[match[1]] = match[2];
			});
        }
		return ogTags;
	}
    
    const getOpenGraphData = (async () => {
        const response = await fetch(`https://cors-anywhere.herokuapp.com/${url}`);
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
.linkContainer {
    background-repeat: no-repeat;
    background-size: 400px;
    padding: 14px 24px 14px 240px;
    position: relative;
    z-index: 0;
}
.linkContainer::after {
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
.linkTitle {
    margin: 0 0 12px;
    padding: 0;
    font-size: 18px;
    line-height: 1.3;
    color: #369;
}
.link:visited .linkTitle {
    color: darkslateblue;
}
.linkDescription {
    margin: 0 0 12px;
    padding: 0;
    font-size: 14px;
    line-height: 1.5;
}
.linkDate {
    font-size: 10px;
    font-style: italic;
}
</style>

<a class="link" href="{url}" target="_blank">
    {#await getOpenGraphData}
        <div class="linkContainer">...loading open graph data</div>
    {:then data}
        <div class="linkContainer" style="background-image: url({data.image})">
            <h1 class="linkTitle">
                {#if data.title}
                   {decodeHtmlEntities(data.title)}
                {:else}
                    {url}
                {/if}
                {#if data.site_name}
                    - {data.site_name}
                {/if}
            </h1>
            {#if data.description}<p class="linkDescription">{decodeHtmlEntities(data.description)}</p>{/if}
            <div class="linkDate">{formatDate(date)}</div>
        </div>
    {:catch error}
        <div class="linkContainer">
            <h1 class="linkTitle">{url}</h1>
            <div class="linkDate">{formatDate(date)}</div>
        </div>
    {/await}
</a>