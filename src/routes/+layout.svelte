<script lang="ts">
    import '../app.css';
    import { autoModeWatcher } from '@skeletonlabs/skeleton';

    let { children } = $props();
</script>

<svelte:head>{@html '<script>(' + autoModeWatcher.toString() + ')();</script>'}</svelte:head>

<header>
    <hgroup>
        <h1 class="h1">Groupify</h1>
    </hgroup>
</header>

<main class="mx-auto max-w-screen-md p-4">
    {@render children()}
</main>
