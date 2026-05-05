<script lang="ts">
  import { onMount, afterUpdate, tick } from "svelte";
  import { BASE_URL } from "./api";

  interface Notification {
    id: number;
    type: string;
    text: string;
    expiration_date: Date;
  }

  let notifications: Notification[] = [];
  let error = "";
  let columnCount = 1;
  let notifWrapper: HTMLElement | null = null;
  let isRecalculating = false;

  let showForm = false;
  let editingId: number | null = null;
  let editingExpiration = "";
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
    editingId = null;
    editingExpiration = "";
    type = "default";
    text = "";
    expiration_date = "";
  }

  function openForm() {
    editingId = null;
    type = "default";
    text = "";
    expiration_date = "";
    showForm = true;
  }

  function openEditForm(n: Notification) {
    editingId = n.id;
    type = n.type;
    text = n.text;
    editingExpiration = String(n.expiration_date);
    expiration_date = "";
    showForm = true;
  }

  function get_expiration_date(days: string) {
    const date = new Date();
    date.setDate(date.getDate() + Number(days));
    const y = date.getFullYear();
    const m = String(date.getMonth() + 1).padStart(2, "0");
    const d = String(date.getDate()).padStart(2, "0");
    return `${y}-${m}-${d} 23:59:59`;
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
      const body = {
        type,
        text,
        expiration_date: expiration_date
          ? get_expiration_date(expiration_date)
          : editingExpiration,
      };
      if (editingId !== null) {
        await fetch(`${BASE_URL}/notifications/${editingId}`, {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body),
        });
      } else {
        await fetch(`${BASE_URL}/post`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(body),
        });
      }
      closeForm();
      await loadNotifications();
    } catch (e) {
      console.error(e);
    }
  }

  async function deleteNotification(id: number) {
    try {
      await fetch(`${BASE_URL}/notifications/${id}`, { method: "DELETE" });
      await loadNotifications();
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

  async function recalcColumns() {
    if (!notifWrapper || isRecalculating) return;
    isRecalculating = true;
    columnCount = 1;
    await tick();
    const available = window.innerHeight - 80;
    while (
      notifWrapper.scrollHeight > available &&
      columnCount < notifications.length
    ) {
      columnCount++;
      await tick();
    }
    isRecalculating = false;
  }

  afterUpdate(() => {
    recalcColumns();
  });

  onMount(() => {
    loadNotifications();
    const t = setInterval(loadNotifications, 5000);
    window.addEventListener("resize", recalcColumns);
    return () => {
      clearInterval(t);
      window.removeEventListener("resize", recalcColumns);
    };
  });
</script>

<div class="container">
  <button class="top-button" on:click={openForm}>Voeg toe</button>
  {#if notifications.length === 0}
    <div class="message">No notifications</div>
  {:else}
    <div
      class="notif-wrapper"
      style="columns: {columnCount}"
      bind:this={notifWrapper}
    >
      {#each notifications as n (n.id)}
        <div class="message">
          <span title={n.type}>{getEmoji(n.type)}</span>
          <span class="message-text">{n.text}</span>
          <span class="row-actions">
            <button class="icon-btn" title="Bewerk" on:click={() => openEditForm(n)}>✏️</button>
            <button class="icon-btn" title="Verwijder" on:click={() => deleteNotification(n.id)}>🗑️</button>
          </span>
        </div>
      {/each}
    </div>
  {/if}
  {#if showForm}
    <button type="button" class="backdrop" on:click={closeForm} aria-label="button"></button>
    <div
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-label="Add notification"
    >
      <div class="modal-title">{editingId !== null ? "Bewerk tekst" : "Voeg tekst toe"}</div>

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
            Aantal dagen{editingId !== null ? " (leeg = ongewijzigd)" : ""}
            <input bind:value={expiration_date} placeholder={editingId !== null ? "ongewijzigd" : "3"} />
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

  :global(body) {
    display: block;
  }

  :global(#app) {
    width: 100%;
    height: 100%;
    max-width: none;
    margin: 0;
    padding: 0;
    text-align: initial;
  }

  .container {
    position: relative;
    height: 100vh;
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  .top-button {
    position: absolute;
    top: 20px;
    background-color: #cf2a1e;
    border: none;
    padding: 10px 20px;
    cursor: pointer;
  }

  .notif-wrapper {
    column-gap: 60px;
    padding: 0 40px;
    width: min(100%, 1800px);
    box-sizing: border-box;
  }

  .message {
    color: white;
    font-size: 40px;
    font-family: Arial, sans-serif;
    text-align: center;
    display: flex;
    align-items: flex-start;
    justify-content: center;
    gap: 12px;
    break-inside: avoid;
    margin-bottom: 20px;
    width: 100%;
    max-width: 100%;
    min-width: 0;
    box-sizing: border-box;
    line-height: 1.2;
    white-space: normal;
  }

  .message-text {
    min-width: 0;
    max-width: 100%;
    overflow-wrap: anywhere;
    word-break: break-word;
  }

  .row-actions {
    display: inline-flex;
    flex: 0 0 auto;
    gap: 4px;
    opacity: 0;
    transition: opacity 0.15s;
  }

  .message:hover .row-actions {
    opacity: 1;
  }

  .icon-btn {
    background: rgba(0, 0, 0, 0.45);
    border: none;
    border-radius: 6px;
    font-size: 24px;
    cursor: pointer;
    padding: 2px 6px;
    line-height: 1;
  }

  .icon-btn:hover {
    background: rgba(0, 0, 0, 0.7);
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
