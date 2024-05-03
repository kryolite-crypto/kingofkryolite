<script>
  import Navbar from "./lib/navbar.svelte";
  import Modal from './lib/Modal.svelte';

  const contractAddress = "kryo:aeckjgbfj5u8grv35ngqkyh85caekb5zregctx4t";
  const url = "https://testnet-1.kryolite.io";

  let response = "";
  let contract;
  let showModal = false;
  let params;
  let name;

  const evtSource = new EventSource(`${url}/contract/${contractAddress}/listen`);

  evtSource.onmessage = (event) => {
    var ev = JSON.parse(event.data)

    if (ev.Type === "Custom" && ev.Event.EventData[0] === "0") {
      load();
    }
  };

  load();

  async function post(url, path, body) {
    return new Promise((resolve, reject) => {
      fetch(`${url}${path}`, {
        body: JSON.stringify(body),
        headers: {
          "Content-Type": "application/json",
        },
        method: "POST",
      })
        .then((response) =>
          response.json().catch((error) => {
            console.error(error);
          })
        )
        .then((resObj) => {
          resolve(resObj);
        })
        .catch((error) => {
          reject(error);
        });
    });
  }

  async function load() {
    post(url, `/contract/${contractAddress}/call`, { method: "GetState" })
      .then((contractObj) => {
        response = JSON.stringify(contractObj);
        contractObj["address"] = contractAddress;
        contract = contractObj;
        contract.kings = contract.kings.reverse()
        console.log(contract);
      })
      .catch((error) => {
        console.error(error);
      });
  }

  const stringToHex = (str) => {
    let hex = '';
    for (let i = 0; i < str.length; i++) {
      const charCode = str.charCodeAt(i);
      const hexValue = charCode.toString(16);

      // Pad with zeros to ensure two-digit representation
      hex += hexValue.padStart(2, '0');
    }
    return hex;
  };

  export function updateParams()
  {
      return stringToHex(JSON.stringify({"method": "ClaimThrone", params: [name]}));
  }
</script>

<body>
  <Navbar />
  {#if contract}
  <div class="main">
    <div class="info">
      <p class="title">Info</p>
      <div class="container">
          <div class="column">
            <div class="row">
              <p class="header">Contract address</p>
              <p class="text">{contractAddress}</p>
            </div>
          </div>
      </div>

      <p class="title">Current state of kingdom</p>
      <div class="container">
          <div class="column">
            <div class="row">
              <p class="header">Current King</p>
              <p class="text">{contract.kings[0].name}</p>
              <a href="https://kryolite-crypto.github.io/explorer/ledger/{contract.kings[0].address}">{contract.kings[contract.kings.length - 1].address}</a>
            </div>
            <div class="row">
              <p class="header">Reign began</p>
              <p class="text">{new Date(contract.kings[0].timestamp).toLocaleString()}</p>
            </div>
            <div class="row">
              <p class="header">Amount required to overthrow the King</p>
              <p class="text">{(contract.claimAmount / 1_000_000).toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 6})} kryo</p> <!--add 1 kryo to always round up-->
            </div>
            <div style="display: flex; justify-content: center">
              <button class="btn-5" on:click={() => (showModal = true)}>CLAIM THRONE</button>
            </div>
          </div>
      </div>
    </div>

    <div class="instructions">
      <p class="title">Instructions</p>
      <div class="container fill-height">
          <!--div class="column">
            <div style="margin-bottom: 10px;">This is a demo application utilizing Kryolite Standard Token interface.</div>
            <div>To participate in this lottery you need to buy a token using Kryolite Wallet.</div>
            <ul>
              <li>Open wallet and go to Send tab</li>
              <li>Select wallet where to send funds from</li>
              <li>Copy {contractAddress} to recipient field, this is the contract address</li>
              <--li>Amount to send: {(contract.ticket_price / 1_000_000)}</li->
              <li>From the next dropdown select <i>Buy ticket</i></li>
              <li>Click Send</li>
              <li>Wait for transaction to complete</li>
              <li>Your wallet should receive token named <i>Lottery Ticket #X</i></li>
            </ul>
            <div style="margin-bottom: 10px;">
              Multiple tickets can be bought for same draw. If you don't see the dropdown which lets you select <i>Buy ticket</i> your walelt might not have fully synchronized yet.
            </div>
            <div>
              After winner has been drawn old tokens in your wallet will show up as consumed and they will not participate in future draws.
            </div>
          </div-->
      </div>
    </div>
  </div>
  
  <p class="title">Past Kings</p>
  <div class="container">
    <div class="column">
      <p class="header">Name</p>
      {#each contract.kings.slice(1) as king, i}
      <p class="text">
        <a href="https://kryolite-crypto.github.io/explorer/ledger/{king.address}">{king.name}</a>
      </p>
    {/each}
    </div>
    <div class="column">
      <p class="header">Address</p>
      {#each contract.kings.slice(1) as king, i}
      <p class="text">
        <a href="https://kryolite-crypto.github.io/explorer/ledger/{king.address}">{king.address}</a>
      </p>
    {/each}
    </div>
    <div class="column">
      <p class="header">Reign began</p>
      {#each contract.kings.slice(1) as king, i}
      <p class="text">
        {new Date(king.timestamp).toLocaleString()}
      </p>
    {/each}
    </div>
    <div class="column">
      <p class="header">Claim amount</p>
      {#each contract.kings.slice(1) as king, i}
      <p class="text">
        {(king.claimAmount / 1_000_000).toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 6})} kryo
      </p>
    {/each}
    </div>
    <div class="column">
      <p class="header">Profit made</p>
      {#each contract.kings.slice(1) as king, i}
      <p class="text">
        {(king.profit / 1_000_000).toLocaleString(undefined, {minimumFractionDigits: 2, maximumFractionDigits: 6})} kryo
      </p>
    {/each}
    </div>
  </div>
  {/if}

  <Modal bind:showModal let:dialog>
    {#if contract}
    Name for your king
    <br />
    <input bind:value={name} on:change={() => params = updateParams()}/>
    <div style="margin: 10px; margin-top: 15px; display: flex; justify-content: center">
      {#if name}
      <a class="btn-5" href="kryolite:call/{contractAddress}/{contract.claimAmount}/{params}" on:click={() => dialog.close()}>CLAIM THRONE</a>
      {:else}
      <a class="btn-5" style="pointer-events: none; opacity: 0.5; cursor: not-allowed;" href="kryolite:call/{contractAddress}/{contract.claimAmount}/{params}" on:click={() => dialog.close()}>CLAIM THRONE</a>
      {/if}
    </div>
    {/if}
  </Modal>
</body>

<style>
    .main {
        display:flex;
        align-content: space-between;
        margin-bottom: 80px;
    }

    .info {
        min-width: 50%;
        margin-right: 20px;
        height: 100%;
        margin-bottom: -55px;
    }

    .instructions {
        width: 50%;
    }

    .row {
      margin-bottom: 20px;
    }

    .fill-height {
      height: 100%;
    }
</style>
