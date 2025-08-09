<script lang="ts">
  import { onMount } from "svelte";
  import { Mistral } from "@mistralai/mistralai";
  import { z } from "zod";

  let video: HTMLVideoElement;
  let screen = $state<"wait" | "loading" | "result">("wait");

  let isPizzaOnPhoto = $state(false);
  let isCoffeeOnPhoto = $state(false);
  let isColaOnPhoto = $state(false);
  let isFriesOnPhoto = $state(false);

  onMount(() => {
    navigator.mediaDevices
      .getUserMedia({
        video: {
          aspectRatio: 1 / 1,
        },
      })
      .then((stream) => {
        video.srcObject = stream;
        video.play();
      });
  });

  async function buttonClick() {
    screen = "loading";
    const apiKey = localStorage.getItem("mistralKey")!;
    const mistral = new Mistral({ apiKey });
    const canvas = document.createElement("canvas");
    canvas.width = 640;
    canvas.height = 360;
    const ctx = canvas.getContext("2d")!;
    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
    const res = await mistral.chat.parse({
      model: "pixtral-12b-2409",
      messages: [
        {
          role: "user",
          content: [
            {
              type: "text",
              text: "Check for the objects from image",
            },
            {
              type: "image_url",
              imageUrl: canvas.toDataURL("image/jpeg"),
            },
          ],
        },
      ],
      responseFormat: z.object({
        isPizzaOnPhoto: z.boolean(),
        isCoffeeOnPhoto: z.boolean(),
        isColaOnPhoto: z.boolean(),
        isFriesOnPhoto: z.boolean(),
      }),
    });
    if (!res.choices) return;
    isPizzaOnPhoto = res.choices[0].message?.parsed.isPizzaOnPhoto;
    isCoffeeOnPhoto = res.choices[0].message?.parsed.isCoffeeOnPhoto;
    isFriesOnPhoto = res.choices[0].message?.parsed.isFriesOnPhoto;
    isColaOnPhoto = res.choices[0].message?.parsed.isColaOnPhoto;
    screen = "result";
    setTimeout(() => {
      screen = "wait";
    }, 3000);
  }
</script>

<div class="h-full relative">
  <!-- svelte-ignore a11y_media_has_caption -->
  <video bind:this={video} class="aspect-video h-full object-cover rounded-3xl"
  ></video>
  {#if screen == "wait"}
    <div class="absolute top-0 left-0 w-full h-full flex flex-col justify-end">
      <div class="flex justify-center p-9">
        <button
          class="bg-amber-600 px-6 py-3 rounded-full text-white text-xl active:scale-90 transition-transform"
          onclick={buttonClick}
        >
          Сканировать
        </button>
      </div>
    </div>
  {:else if screen == "loading"}
    <div
      class="absolute rounded-3xl top-0 left-0 w-full h-full flex items-center justify-center bg-black/70"
    >
      <div class="text-white text-3xl">Происходит магия...</div>
    </div>
  {:else}
    <div
      class="absolute rounded-3xl top-0 left-0 w-full h-full flex items-center justify-center bg-black/70"
    >
      <div class="bg-white p-7 w-1/2 rounded-xl text-xl space-y-3">
        <div class="text-3xl font-bold">Я распознал</div>
        <ul class="list-disc px-5">
          {#if isPizzaOnPhoto}
            <li>Пицца 400 ₽</li>
          {/if}
          {#if isCoffeeOnPhoto}
            <li>Кофе 200 ₽</li>
          {/if}
          {#if isColaOnPhoto}
            <li>Кола 80 ₽</li>
          {/if}
          {#if isFriesOnPhoto}
            <li>Картофель фри 180 ₽</li>
          {/if}
        </ul>
      </div>
    </div>
  {/if}
</div>
