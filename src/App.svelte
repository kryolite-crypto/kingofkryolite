<script>
  import Navbar from "./lib/navbar.svelte";

  const contractAddress = "kryo:wea3prgphv2dmxjh46sadh77r57pv62wg2m7dt8v8a";
  const url = "http://localhost:5000";
  const KRYO_DIVISION = 1000000;

  let response = "";
  let contract;

  // By default give this address:
  addressInputted({
    detail: { data: contractAddress },
  });

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

  async function addressInputted(event) {
    let address = event.detail.data;

    if (!address) return;
    if (!address.startsWith("kryo:")) return;

    post(url, `/contract/${address}/call`, { method: "GetState" })
      .then((contractObj) => {
        response = JSON.stringify(contractObj);
        contractObj["address"] = address;
        contract = contractObj;
        contract.kings = contract.kings.reverse()
        console.log(contract);
      })
      .catch((error) => {
        console.error(error);
      });
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
      <div class="container fill-height">
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
              <p class="text">{((contract.claimAmount + 1_000_000) / 1_000_000).toFixed(0)} kryo</p> <!--add 1 kryo to always round up-->
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
        {(king.claimAmount / 1_000_000).toFixed(2)}
      </p>
    {/each}
    </div>
    <div class="column">
      <p class="header">Profit made</p>
      {#each contract.kings.slice(1) as king, i}
      <p class="text">
        {(king.profit / 1_000_000).toFixed(2)}
      </p>
    {/each}
    </div>
  </div>
  {/if}
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
        margin-bottom: -59px;
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
