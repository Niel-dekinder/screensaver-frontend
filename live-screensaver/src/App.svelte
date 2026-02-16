<script lang="ts">
  import { onMount } from "svelte";
  import { BASE_URL } from "./api";

  interface Notification {
    type: string;
    text: string;
  }

  let notifications: Notification[] = [];
  let error = "";

  async function loadNotifications() {
    try {
      error = "";
      const res = await fetch(`${BASE_URL}/notifications`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      notifications = await res.json();
    } catch (e) {
      notifications = [];
    }
  }

  onMount(() => {
    loadNotifications();
    const t = setInterval(loadNotifications, 5000);
    return () => clearInterval(t);
  });
</script>

<div class="container">
  {#if notifications.length === 0}
    <div class="message">No notifications</div>
  {:else}
    {#each notifications as n (n.type ?? n.text ?? n)}
      <div class="message">
        type: {typeof n === "string" ? n : n.type} -- 
        text: {typeof n === "string" ? n : n.text}
      </div>
    {/each}
  {/if}
</div>

<style>
  :global( html, body) {
    margin: 0;
    height: 100%;
    overflow: hidden;
    background: url('./background.png') center center / cover no-repeat fixed;
  }

  .container {
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 20px;
  }

  .message {
    color: white;
    font-size: 40px;
    font-family: Arial, sans-serif;
    text-align: center;
  }
</style>
