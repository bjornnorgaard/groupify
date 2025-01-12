<script lang="ts">
    import '../app.css';
    import { autoModeWatcher } from '@skeletonlabs/skeleton';

    let { children } = $props();
</script>

<svelte:head>{@html '<script>(' + autoModeWatcher.toString() + ')();</script>'}</svelte:head>

<div class="p-4">
    <header>
        <hgroup>
            <h1 class="h1">Groupify</h1>
        </hgroup>
    </header>

    <main class="mx-auto my-8 max-w-screen-xl">
        {@render children()}
    </main>
</div>
