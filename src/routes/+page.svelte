<script lang="ts">
    import "../styles.css";
    import { onMount } from "svelte";

    interface Musica {
        nome: String;
        interprete: String;
        codigo: Number;
        trecho: String;
        idioma: String;
    };

    let musicas: Musica[] = [];
    let encontradas: Musica[] = []

    let search = "";
    let hasText = false;
    let searchTimeout = setTimeout(() => 0, 1);
    let searching = false;

    function procurarMusica() {
        const termo = search.toLowerCase()
        
        encontradas = musicas.filter(musica => {
            return musica.nome.toLowerCase().indexOf(termo) != -1 ||
            musica.interprete.toLowerCase().indexOf(termo) != -1 ||
            musica.trecho.toLowerCase().indexOf(termo) != -1
        });

        searching = false;
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
        const response  = await fetch("/songs_min.json");
        if (!response.ok) {
            alert("Não consegui baixar as músicas...");
            return
        }

        const rawMusicas: Musica[] = await response.json()
        musicas = rawMusicas.map(musica => {
            musica.interprete += '';
            musica.nome += '';
            musica.trecho += '';

            return musica;
        });
        console.log("Músicas encontradas:", musicas.length);
    }

    onMount(() => {
        loadMusicas();
    });
</script>
<svelte:head>
    <title>Karaokê do Sandro</title>
</svelte:head>

<main>
    <div class:flex-container={!hasText}>
        <div class="container center bg-white" class:logo={!hasText}>
            <img src="/videoke-logo.png" class:image-small="{hasText}" alt="Videokê Logo - Leão cantando e segurando o microfone com a mão direita."/>
        </div>
        <div class="container search-container mb-4">
            <div class="p-2">
                <input type="search" bind:value={search} on:input={onBuscaChanged} class="form-control w-100" placeholder="Buscar por Músicas, Intérpretes">
            </div>
        </div>
    </div>
    <div class="container container-fill" class:hidden="{!hasText}">
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
                        {#if encontradas.length }
                        {#each encontradas as musica}
                        <tr>
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
