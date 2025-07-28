<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
  <link href="https://fonts.googleapis.com/css2?family=Manrope:wght@200;300;400;500;600;700;800&display=swap" rel="stylesheet">
</svelte:head>

<script lang="ts">
  import { onMount } from 'svelte';
  import Button from '@/lib/components/ui/button.svelte';
  import Card from '@/lib/components/ui/card.svelte';
  import Input from '@/lib/components/ui/input.svelte';
  import Label from '@/lib/components/ui/label.svelte';
  import { cn } from '@/lib/utils';

  let fileInput: HTMLInputElement;
  let password: string = '';
  let selectedFile: File | null = null;
  let showPassword: boolean = false;
  let isDragOver: boolean = false;
  let isProcessing: boolean = false;
  let isDarkMode: boolean = false;

  function handleFileChange(event: Event): void {
    const target = event.target as HTMLInputElement;
    selectedFile = target.files?.[0] || null;
  }

  function togglePasswordVisibility(): void {
    showPassword = !showPassword;
  }

  function handleDragOver(event: DragEvent): void {
    event.preventDefault();
    isDragOver = true;
  }

  function handleDragLeave(event: DragEvent): void {
    event.preventDefault();
    isDragOver = false;
  }

  function handleDrop(event: DragEvent): void {
    event.preventDefault();
    isDragOver = false;
    const files = event.dataTransfer?.files;
    if (files && files.length > 0 && files[0]?.type === 'application/pdf') {
      selectedFile = files[0];
    }
  }

  async function handleSubmit(): Promise<void> {
    if (!selectedFile || !password) return;

    isProcessing = true;
    const formData = new FormData();
    formData.append('file', selectedFile);
    formData.append('password', password);
    
    try {
      const response = await fetch('https://tools.thesoorajsingh.me/unlock', {
        method: 'POST',
        body: formData
      });

      if (response.ok) {
        const blob = await response.blob();
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `unlocked_${selectedFile.name}`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        console.log('PDF unlocked successfully');
      } else {
        console.error('Failed to unlock PDF:', response.statusText);
      }
    } catch (error) {
      console.error('Error unlocking PDF:', error);
    } finally {
      isProcessing = false;
    }
  }

  function removeFile(): void {
    selectedFile = null;
    if (fileInput) fileInput.value = '';
  }

  function toggleDarkMode(): void {
    isDarkMode = !isDarkMode;
    document.documentElement.classList.toggle('dark', isDarkMode);
    localStorage.setItem('darkMode', isDarkMode.toString());
  }

  onMount(() => {
    const savedMode = localStorage.getItem('darkMode');
    if (savedMode !== null) {
      isDarkMode = savedMode === 'true';
    } else {
      isDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
    }
    document.documentElement.classList.toggle('dark', isDarkMode);
  });
</script>

<div class="min-h-screen bg-background text-foreground transition-colors">
  <!-- Theme Toggle -->
  <Button
    variant="ghost"
    size="sm"
    on:click={toggleDarkMode}
    className="fixed top-4 right-4 z-50 h-9 w-9 p-0"
  >
    {#if isDarkMode}
      <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
      </svg>
    {:else}
      <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" />
      </svg>
    {/if}
  </Button>

  <main class="container mx-auto max-w-md px-4 py-16">
    <!-- Header -->
    <div class="text-center mb-8">
      <div class="text-4xl mb-4">🔓</div>
      <h1 class="text-2xl font-semibold mb-2">PDF Unlocker</h1>
      <p class="text-muted-foreground">Remove password protection from your PDFs</p>
    </div>

    <!-- Main Card -->
    <Card className="p-6 space-y-6">
      <!-- File Upload -->
      <div class="space-y-2">
        <Label>PDF File</Label>
        <div
          class={cn(
            "border-2 border-dashed rounded-lg p-6 text-center cursor-pointer transition-colors",
            isDragOver ? "border-primary bg-primary/5" : "border-border hover:border-primary/50",
            selectedFile && "border-green-500 bg-green-500/5"
          )}
          on:dragover={handleDragOver}
          on:dragleave={handleDragLeave}
          on:drop={handleDrop}
          role="button"
          tabindex="0"
        >
          <input
            bind:this={fileInput}
            type="file"
            accept=".pdf"
            on:change={handleFileChange}
            class="absolute inset-0 opacity-0 cursor-pointer"
          />

          {#if selectedFile}
            <div class="flex items-center gap-3 text-left">
              <div class="text-2xl">📄</div>
              <div class="flex-1 min-w-0">
                <div class="font-medium truncate">{selectedFile.name}</div>
                <div class="text-sm text-muted-foreground">
                  {(selectedFile.size / 1024 / 1024).toFixed(2)} MB
                </div>
              </div>
              <Button
                variant="ghost"
                size="sm"
                on:click={removeFile}
                className="h-8 w-8 p-0 text-muted-foreground hover:text-destructive"
              >
                ✕
              </Button>
            </div>
          {:else}
            <div>
              <div class="text-3xl mb-2">📁</div>
              <div class="text-sm font-medium mb-1">Drop your PDF here</div>
              <div class="text-xs text-muted-foreground">or click to browse</div>
            </div>
          {/if}
        </div>
      </div>

      <!-- Password Input -->
      <div class="space-y-2">
        <Label for="password">Password</Label>
        <div class="relative">
          <Input
            id="password"
            type={showPassword ? "text" : "password"}
            bind:value={password}
            placeholder="Enter PDF password"
            className="pr-10"
          />
          <Button
            variant="ghost"
            size="sm"
            on:click={togglePasswordVisibility}
            className="absolute right-0 top-0 h-10 w-10 p-0"
            type="button"
          >
            {#if showPassword}
              <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
              </svg>
            {:else}
              <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.878 9.878L3 3m6.878 6.878L21 21" />
              </svg>
            {/if}
          </Button>
        </div>
      </div>

      <!-- Submit Button -->
      <Button
        on:click={handleSubmit}
        disabled={isProcessing || !selectedFile || !password}
        className="w-full"
        size="lg"
      >
        {#if isProcessing}
          <svg class="animate-spin -ml-1 mr-2 h-4 w-4" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          Processing...
        {:else}
          Unlock PDF
        {/if}
      </Button>
    </Card>
  </main>
</div>

<style>
  .font-manrope {
    font-family: 'Manrope', -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
  }

  .circuit-background {
    background-color: #f8fafc;
    background-image: 
      linear-gradient(90deg, rgba(59, 130, 246, 0.05) 1px, transparent 1px),
      linear-gradient(180deg, rgba(59, 130, 246, 0.05) 1px, transparent 1px),
      radial-gradient(circle at 20px 20px, rgba(59, 130, 246, 0.1) 1px, transparent 1px),
      radial-gradient(circle at 60px 60px, rgba(147, 51, 234, 0.1) 1px, transparent 1px);
    background-size: 40px 40px, 40px 40px, 40px 40px, 80px 80px;
    animation: circuitMove 20s linear infinite;
  }

  .dark .circuit-background {
    background-color: #0f172a;
    background-image: 
      linear-gradient(90deg, rgba(59, 130, 246, 0.1) 1px, transparent 1px),
      linear-gradient(180deg, rgba(59, 130, 246, 0.1) 1px, transparent 1px),
      radial-gradient(circle at 20px 20px, rgba(59, 130, 246, 0.15) 1px, transparent 1px),
      radial-gradient(circle at 60px 60px, rgba(147, 51, 234, 0.15) 1px, transparent 1px);
    background-size: 40px 40px, 40px 40px, 40px 40px, 80px 80px;
  }

  @keyframes circuitMove {
    0% { background-position: 0px 0px, 0px 0px, 0px 0px, 0px 0px; }
    100% { background-position: 40px 40px, 40px 40px, 40px 40px, 80px 80px; }
  }

  .font-manrope {
    font-family: 'Manrope', -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
  }

  .subtle-gradient {
    background: linear-gradient(-45deg, #f8fafc, #f1f5f9, #e2e8f0, #cbd5e1);
    background-size: 400% 400%;
    animation: subtleShift 20s ease infinite;
  }

  @keyframes subtleShift {
    0% {
      background-position: 0% 50%;
    }
    50% {
      background-position: 100% 50%;
    }
    100% {
      background-position: 0% 50%;
    }
  }

  .moving-gradient {
    background: linear-gradient(-45deg, #667eea, #764ba2, #f093fb, #f5576c, #4facfe, #00f2fe);
    background-size: 400% 400%;
    animation: gradientShift 15s ease infinite;
  }

  @keyframes gradientShift {
    0% {
      background-position: 0% 50%;
    }
    50% {
      background-position: 100% 50%;
    }
    100% {
      background-position: 0% 50%;
    }
  }

  .border-3 {
    border-width: 3px;
  }

  .hover\:scale-102:hover {
    transform: scale(1.02);
  }

  .shadow-3xl {
    box-shadow: 0 35px 60px -12px rgba(0, 0, 0, 0.25);
  }
</style>
