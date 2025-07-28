<script lang="ts">
  import { onDestroy } from 'svelte';
  export let message: string;
  export let type: 'default' | 'destructive' = 'default';
  export let visible: boolean = false;
  export let duration: number = 4000;
  export let onClose: () => void = () => {};

  let timeout: ReturnType<typeof setTimeout>;

  $: if (visible) {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      onClose();
    }, duration);
  }

  onDestroy(() => {
    clearTimeout(timeout);
  });
</script>

{#if visible}
  <div
    class="fixed top-6 left-1/2 transform -translate-x-1/2 z-50 px-4 py-2 rounded shadow-lg font-semibold transition
      {type === 'destructive' ? 'bg-red-600 text-white' : 'bg-gray-800 text-white'}"
    role="alert"
    aria-live="assertive"
  >
    {message}
  </div>
{/if}
