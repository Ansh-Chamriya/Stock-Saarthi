<script lang="ts">
  import { Send } from "lucide-svelte";
  import Button from "./components/ui/button/button.svelte";
  import Input from "./components/ui/input/input.svelte";
  import { enhance } from "$app/forms";
  import { db } from "./db";
  import toast from "svelte-french-toast";
  import ChatAnimation from "./ChatAnimation.svelte";
  export let iswaiting: boolean;
  let prompt = "";
  async function getRes() {
    iswaiting = true;
    const req = await fetch("/predict", {
      method: "POST",
      body: JSON.stringify({ prompt }),
      headers: {
        "Content-Type": "application/json",
      },
    });

    if (req.status === 201) {
      // const { responseText } = await req.json();
      const data = await req.json();
      const { responseText } = data;
      console.log(JSON.parse(responseText));

      db.transaction("rw", db.chats, async () => {
        await db.chats.add({
          prompt: prompt,
          response: JSON.parse(responseText).answer,
          cretaedAt: new Date(),
        });
      })
        .then(() => {
          iswaiting = false;
          prompt = "";
        })
        .catch((e) => {
          toast.error("Try again");
          console.error(e);
        });
    }
  }
</script>

<ChatAnimation {iswaiting} />
<div>
  <form on:submit={getRes}>
    <div class="flex gap-3">
      <Input type="text" placeholder="Type a message..." bind:value={prompt} />
      <Button type="submit">
        {#if !iswaiting}
          <Send></Send>
        {:else}
          <div
            class="size-4 animate-spin rounded-full border-b-2 border-t-2 border-white"
          ></div>
        {/if}
      </Button>
    </div>
  </form>
</div>
