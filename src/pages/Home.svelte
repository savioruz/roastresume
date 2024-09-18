<script lang="ts">
  import "$src/app.css";
  import ISO6391 from 'iso-639-1';
  import { ModeWatcher } from "mode-watcher";
  import { Bolt, LoaderCircle } from "lucide-svelte";
  import { Button } from "$lib/components/ui/button";
  import { Card, CardHeader, CardTitle, CardContent } from "$lib/components/ui/card";
  import { Input } from "$lib/components/ui/input";
  import { Label } from "$lib/components/ui/label";
  import { Skeleton } from "$lib/components/ui/skeleton";
  import * as Collapsible from "$lib/components/ui/collapsible";
  import * as Select from "$lib/components/ui/select";
  import Theme from "$lib/widgets/Theme.svelte";
  import Footer from '$lib/widgets/Footer.svelte';

  const api = "https://roastgithub-api.vercel.app/api/v1/roast/resume";
  let key : string | null = null;
  let lang : string | null = null;
  let generated_content : string | null = null;
  let loading : boolean = false;
  let errorMessage : string | null = null;

  const langs = ['id', 'en'];
  const languages = langs.map((lang) => ISO6391.getName(lang));

  function getApiUrl(): string {
    const randomIndex = Math.floor(Math.random() * api.length);
    return api[randomIndex];
  }

  async function handleSubmit() {
    const file = (document.getElementById("file") as HTMLInputElement).files?.[0];
    if (!file) {
      errorMessage = "Please upload a file";
      return;
    }

    if (document.getElementById("lang")) {
      lang = (document.getElementById("lang") as HTMLInputElement).value;
    }

    if (document.getElementById("key")) {
      key = (document.getElementById("key") as HTMLInputElement).value;
    }

    loading = true;

    const formData = new FormData();
    formData.append("file", file);
    if (lang) {
      formData.append("lang", lang);
    } else if (key) {
      formData.append("key", key);
    }

    try {
      const response = await fetch(getApiUrl(), {
        method: "POST",
        body: formData,
      });

      if (!response.ok) {
        throw new Error("Failed to roast your resume");
      }

      const data = await response.json();
      generated_content = data.data.generated_content.replace(/\n/g, "<br>").replace(/\*/g, "");
    } catch (error) {
      errorMessage = error as string;
    } finally {
      loading = false;
    }
  }
</script>

<ModeWatcher />
<main class="flex-grow container mx-auto px-4 py-16 max-w-4xl min-h-[90vh]">
  <Card class="mb-12 shadow-lg">
    <CardHeader>
      <CardTitle>Roast your resume</CardTitle>
    </CardHeader>
    <CardContent>
      <div class="flex flex-col space-y-4">
        <Label for="file">Upload your resume</Label>
        <Input id="file" type="file" accept=".pdf" />
        {#if loading}
          <Button disabled>
            <LoaderCircle class="mr-2 h-4 w-4 animate-spin" />
            Please wait
          </Button>
        {:else}
          <Button on:click={handleSubmit}>Roast</Button>
        {/if}
      </div>
      {#if loading}
        <div class="p-4 my-3 space-y-5">
          <Skeleton class="h-[35px] max-w-[150px] md:max-w-sm" />
          <div class="space-y-3">
            <Skeleton class="h-[25px] max-w-[200px] md:max-w-lg" />
            <Skeleton class="h-[25px]" />
            <Skeleton class="h-[25px]" />
          </div>
        </div>
      {:else if generated_content}
        <div class="p-4 my-3">
          <p>{@html generated_content}</p>
        </div>
      {:else if errorMessage}
        <div class="bg-red-100 text-red-800 rounded-sm mb-4 mt-4 p-3 w-fit">
          <p class="text-sm">{errorMessage}</p>
        </div>
      {/if}
    </CardContent>
  </Card>
  <div class="flex">
    <Collapsible.Root class="space-y-3 basis-full">
      <div class="flex items-center space-x-4 px-4">
        <Collapsible.Trigger asChild let:builder>
          <Button builders={[builder]} variant="ghost">
            <Bolt class="mr-2 h-4 w-4" />
            Settings
          </Button>
        </Collapsible.Trigger>
      </div>
      <Collapsible.Content class="space-y-3 px-4">
        <Select.Root portal={null}>
          <Select.Trigger class="w-[180px]">
            <Select.Value placeholder="Select a language" />
          </Select.Trigger>
          <Select.Content>
            <Select.Group>
              <Select.Label>Language</Select.Label>
              {#each languages as l}
                <Select.Item value={l} label={l}>
                  {l}
                </Select.Item>
              {/each}
            </Select.Group>
          </Select.Content>
          <Select.Input id="lang" name="lang" />
        </Select.Root>
        <Input id="key" bind:value={key} type="text" placeholder="Gemini Api Key (Optional)" class="w-full" />
      </Collapsible.Content>
    </Collapsible.Root>
    <div class="basis-0 mr-4">
      <Theme />
    </div>
  </div>
</main>
<Footer />
