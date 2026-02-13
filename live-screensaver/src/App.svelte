<script>
  let messages = [
    { type: "logistics", text: "📦 new Milwaukee arrived" },
    { type: "delivery", text: "🚚 Bosch delayed - 22 feb" },
    { type: "promo", text: "🟢 New Promo tomorrow" },
  ];

</script>

<div class="container">
  {#each messages as message (message)}
    <div class="message">
      {message.text}
    </div>
  {/each}
</div>

<style>
  :global( body) {
    margin: 0;
    background: black;
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
