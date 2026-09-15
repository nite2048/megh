<script>
    import one from "./1.svelte";
    import two from "./2.svelte";
    import three from "./3.svelte";
    import four from "./4.svelte";
    import Router from 'svelte-spa-router';
   	import { onMount } from 'svelte';


    const audio = new URL('./assets/audio.weba', import.meta.url).href;

    const routes = {
        '/': one,
        '/2': two,
        '/3': three,
        '/4': four
    };

    let music;

    function startMusic() {
        if (music) {
            music.play().catch(() => {});
        }
    }

    onMount(() => {
        startMusic();
    });     
</script>

<svelte:window on:click={startMusic} on:touchstart={startMusic} />

<Router {routes}/>

<audio bind:this={music} loop>
    <source src={audio} type="audio/webm">
    Your browser does not support the HTML5 audio element.
</audio>