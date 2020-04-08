<script>
    export let url;
    export let date;

    import moment from 'moment';

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
    background-repeat: no-repeat;
    background-size: 200px;
    padding: 10px 10px 10px 240px;
    border: 1px solid #ddd;
    border-radius: 3px;
}
.linkTitle {
    margin: 0 0 12px;
    padding: 0;
    font-size: 24px;
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

<div class="linkContainer">
    {#await getOpenGraphData}
        ...loading link
    {:then data}
        <a class="link" href="{url}" target="_blank" style="background-image: url({data.image})">
            <h1 class="linkTitle">
                {#if data.title}
                   {data.title}
                {:else}
                    {url}
                {/if}
                {#if data.site_name}
                    - {data.site_name}
                {/if}
            </h1>
            {#if data.description}<p class="linkDescription">{data.description}</p>{/if}
            <div class="linkDate">{formatDate(date)}</div>
        </a>
    {:catch error}
        <a href="{url}" target="_blank">{url}</a><br>
        {formatDate(date)}
    {/await}
</div>