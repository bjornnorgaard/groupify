<script lang="ts">
    let { title, children } = $props();
</script>

<div class="card p-4">
    <h2 class="h2 pb-4">{title}</h2>
    {@render children()}
</div>
