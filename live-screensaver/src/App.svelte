<script lang="ts">
  import { onMount } from "svelte";
  import { BASE_URL } from "./api";

  interface Notification {
    id: number;
    type: string;
    text: string;
    expiration_date: Date;
  }

  let notifications: Notification[] = [];
  let error = "";

  let showForm = false;
  let type = "default";
  let text = "";
  let expiration_date = "";

  let symbol_table = [
    { value: "default", emoji: "🔴" },
    { value: "verlof", emoji: "🌴" },
    { value: "ziekte", emoji: "🤒" },
    { value: "verjaardag", emoji: "🎂" },
    { value: "feestdag", emoji: "🎉" },
  ];

  function closeForm() {
    showForm = false;
  }

  function openForm() {
    showForm = true;
  }

  function get_expiration_date(days: string) {
    const date = new Date();
    date.setUTCDate(date.getUTCDate() + Number(days));
    return date.toISOString().slice(0, 19).replace("T", " ");
  }

  function getEmoji(type: string): string {
    const match = symbol_table.find(
      (item) => item.value === type?.toLowerCase(),
    );
    return match?.emoji ?? "🔴";
  }

  async function submitForm() {
    if (!text) {
      closeForm();
      return;
    }
    try {
      await fetch(`${BASE_URL}/post`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          type: type || "default",
          text: text || "Geen tekst ingevoerd",
          expiration_date: get_expiration_date(expiration_date),
        }),
      });
      closeForm();
    } catch (e) {
      console.error(e);
    }
  }

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
  <button class="top-button" on:click={openForm}>Voeg toe</button>
  {#if notifications.length === 0}
    <div class="message">No notifications</div>
  {:else}
    {#each notifications as n (n.id)}
      <div class="message">
        <span title={n.type} style="cursor: pointer;">
          {getEmoji(n.type)}
        </span>
        {n.text}
      </div>
    {/each}
  {/if}
  {#if showForm}
    <div class="backdrop" on:click={closeForm}></div>
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-label="Add notification"
    >
      <div class="modal-title">Voeg tekst toe</div>

      <form on:submit|preventDefault={submitForm}>
        <label>
          Type
          <select bind:value={type}>
            {#each symbol_table as opt (opt.value)}
              <option value={opt.value}>{opt.value + " " + opt.emoji}</option>
            {/each}
          </select>

          <label>
            Tekst
            <textarea
              bind:value={text}
              rows="3"
              placeholder="De Kinder is goe beizg... ;)"
            ></textarea>
          </label>

          <label>
            Aantal dagen
            <input bind:value={expiration_date} placeholder="3" />
          </label>

          <div class="actions">
            <button class="form-btn" type="button" on:click={closeForm}
              >Cancel</button
            >
            <button class="form-btn" type="submit">Save</button>
          </div>
        </label>
      </form>
    </div>
  {/if}
</div>

<style>
  :global(html, body) {
    margin: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: url("/background.png") center center / cover no-repeat fixed;
  }

  .container {
    position: relative;
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 20px;
  }

  .top-button {
    position: absolute;
    top: 20px;
    background-color: #cf2a1e;
    border: none;
    padding: 10px 20px;
    cursor: pointer;
  }

  .message {
    color: white;
    font-size: 40px;
    font-family: Arial, sans-serif;
    text-align: center;
  }

  .backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.45);
  }

  .modal {
    position: fixed;
    top: 15%;
    left: 50%;
    transform: translateX(-50%);
    width: min(520px, calc(100% - 32px));
    background: white;
    border-radius: 12px;
    padding: 16px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.25);
  }

  form {
    display: grid;
    gap: 10px;
  }
  label {
    text-align: left;
    display: grid;
    color: rgb(11, 9, 9);
    gap: 6px;
  }
  input,
  select,
  textarea {
    padding: 8px 10px;
    border: 1px solid #ccc;
    color: black;
    background-color: white;
    border-radius: 8px;
    font: inherit;
  }

  .actions {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    width: 100%;
    margin-top: 6px;
  }

  .modal-title {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 12px;
    color: #00283f;
  }

  .form-btn {
    padding: 8px 12px;
    background-color: #cf2a1e;
    width: 50%;
    border: none;
    border-radius: 8px;
    cursor: pointer;
  }
</style>
