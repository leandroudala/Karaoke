<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import '../styles.css';
	import { onMount } from 'svelte';

	interface Musica {
		nome: String;
		interprete: String;
		codigo: Number;
		trecho: String;
		idioma: String;
		index: String;
	}

	let musicas: Musica[] = [];
	let encontradas: Musica[] = [];

	let search = '';
	let hasText = false;
	let searchTimeout = setTimeout(() => 0, 1);
	let searching = false;

	onMount(() => {
		const query = new URLSearchParams($page.url.searchParams.toString());
		search = query.get('q') ?? '';

		if (search !== '') {
			onBuscaChanged();
		}
	});

	function removerAcentuacao(texto: string): string {
		return texto.normalize('NFD').replace(/[\u0300-\u036f]/g, '');
	}

	function corrigirEspacamento(str: string): string {
		return str.replace(/\s{2,}/g, ' ');
	}

	function limparString(texto: string): string {
		return corrigirEspacamento(removerAcentuacao(texto).toLowerCase().trim());
	}

	function procurarMusica() {
		const termo = limparString(search);

		encontradas = musicas.filter((musica) => musica.index.indexOf(termo) != -1);

		const query = new URLSearchParams($page.url.searchParams.toString());
		query.set('q', termo);
		searching = false;
		goto(`?${query.toString()}`);
	}

	let isPlaying = false;
	let backgroundMusic: HTMLAudioElement;

	function tocarSom(codigo: Number) {
		console.log(backgroundMusic);
		const cod = ('' + codigo).padStart(5, '0');
		const audioUrl = `https://ivideoke.com.br/static/mp3/${cod}.mp3`;

		const currentUri = backgroundMusic.src;
		const sameAudio = currentUri == audioUrl;

		if (isPlaying && sameAudio) {
			backgroundMusic.pause();
			isPlaying = false;
			return;
		}

		if (!isPlaying && sameAudio) {
			backgroundMusic.play();
			isPlaying = true;
			return;
		}

		backgroundMusic.src = audioUrl;
		backgroundMusic.play();
		isPlaying = true;
	}

	function onBuscaChanged() {
		hasText = search.length != 0;
		clearTimeout(searchTimeout);
		encontradas = [];

		if (!hasText) {
			return;
		}

		searching = true;
		searchTimeout = setTimeout(procurarMusica, 500);
	}

	async function loadMusicas() {
		const response = await fetch('/songs_min.json');
		if (!response.ok) {
			alert('Não consegui baixar as músicas...');
			return;
		}

		const rawMusicas: Musica[] = await response.json();
		musicas = rawMusicas.map((musica) => {
			musica.index = limparString(
				(musica.interprete + '_').trim() + (musica.nome + '_').trim() + (musica.trecho + '').trim()
			);

			return musica;
		});
		console.log('Músicas encontradas:', musicas.length);
	}

	onMount(() => {
		loadMusicas();
	});
</script>

<svelte:head>
	<title>Karaokê do Sandro</title>
	<!-- Google tag (gtag.js) -->
	<script async src="https://www.googletagmanager.com/gtag/js?id=G-7779Y1S13B"></script>
	<script>
	window.dataLayer = window.dataLayer || [];
	function gtag(){dataLayer.push(arguments);}
	gtag('js', new Date());

	gtag('config', 'G-7779Y1S13B');
	</script>

	<!-- Google Tag Manager -->
	<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
	new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
	j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
	'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
	})(window,document,'script','dataLayer','GTM-MSMBV72K');</script>
	<!-- End Google Tag Manager -->
</svelte:head>
<!-- Google Tag Manager (noscript) -->
<noscript>
	<iframe src="https://www.googletagmanager.com/ns.html?id=GTM-MSMBV72K"
	height="0" width="0" style="display:none;visibility:hidden" title="Google Tag Manager"></iframe>
</noscript>
	<!-- End Google Tag Manager (noscript) -->
<main>
	<audio src="" bind:this={backgroundMusic} />
	<div class:flex-container={!hasText}>
		<div class="header">
			<div class="container center bg-white" class:logo={!hasText}>
				<img
					src="/favicon.png"
					class:image-small={hasText}
					alt="Videokê Logo - Leão cantando e segurando o microfone com a mão direita."
				/>
			</div>
			{#if hasText}
				<div class="container center">
					<div class="title">
						Karaokê<br />
						do<br />
						Sandro
					</div>
					<div>
						{musicas.length} músicas
					</div>
				</div>
			{/if}
		</div>
		<div class="container search-container mb-4">
			<div class="p-2">
				<input
					type="search"
					bind:value={search}
					on:input={onBuscaChanged}
					class="form-control w-100"
					placeholder="Buscar por Músicas, Intérpretes"
				/>
			</div>
		</div>
	</div>
	<div class="container container-fill" class:hidden={!hasText}>
		<div class="scrollable-content">
			<table class="table table-striped">
				<thead class="thead-green">
					<tr>
						<th scope="col">Código</th>
						<th scope="col">Intérprete</th>
						<th scope="col">Título</th>
						<th scope="col">Início da Letra</th>
					</tr>
				</thead>
				<tbody>
					{#if encontradas.length}
						{#each encontradas as musica}
							<tr on:click={() => tocarSom(musica.codigo)}>
								<td>{musica.codigo}</td>
								<td>{musica.interprete}</td>
								<td>{musica.nome}</td>
								<td>{musica.trecho}</td>
							</tr>
						{/each}
					{:else}
						<tr>
							<td class="center" colspan="4">
								{#if searching}
									Procurando...
								{:else}
									Nenhuma música encontrada.
								{/if}
							</td>
						</tr>
					{/if}
				</tbody>
			</table>
		</div>
	</div>
</main>
