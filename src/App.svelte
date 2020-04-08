<script>
	import { handleStatus, toJson } from './utils';
	import { masterRef, links } from './stores';
	import Link from './Link.svelte';

	const apiBase = `https://justinharkey.prismic.io/api/v2`;
	const linksQuery = `[[at(document.type,+"links_i_read")]]`;

	const getLinksIRead = () => {
		let fetchUrl = `${apiBase}/documents/search?ref=${$masterRef}&q=${escape(linksQuery)}`;
		fetch(fetchUrl, { method: 'get' })
		.then(handleStatus)
		.then(toJson)
		.then((data) => {
			let linksData = [];
			data.results.map((linkData) => {
				linksData.push({
					linkDate: linkData.data.link_date,
					linkUrl: linkData.data.link_url.url
				});
			});
			// sort by date asc
			linksData.sort((a,b) => (a.linkDate < b.linkDate) ? 1 : -1);
			links.set(linksData);
		})
		.catch((error) => {
			console.log('Request failed', error);
		});
	}

	const getMasterRef = (() => {
		fetch(apiBase, { method: 'get' })
		.then(handleStatus)
		.then(toJson)
		.then((data) => {
			masterRef.set(data.refs[0].ref);
			getLinksIRead();
		})
		.catch((error) => {
			console.log('Request failed', error);
		});
	})();
</script>

{#each $links as link}
	<Link url={link.linkUrl} date={link.linkDate} />
{/each}