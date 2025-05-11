<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Token Migration</title>
  <script src="https://cdn.jsdelivr.net/npm/web3@1.10.0/dist/web3.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background: #f5f7fa;
      padding: 50px;
    }
    h2 {
      color: #333;
    }
    input, button {
      padding: 10px;
      margin: 10px;
      font-size: 16px;
    }
    button {
      cursor: pointer;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 5px;
    }
    button:hover {
      background-color: #0056b3;
    }
    #status {
      margin-top: 20px;
      color: #444;
    }
    .container {
      background: white;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
      display: inline-block;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Token Migration<br><small>(17 → 18 Decimals)</small></h2>

    <button onclick="connectWallet()">🔌 Connect Wallet</button>
    <p id="wallet"></p>

    <input type="number" id="amount" placeholder="Amount in whole tokens" step="any" />
    <br>
    <button onclick="approve()">1. Approve</button>
    <button onclick="migrate()">2. Migrate</button>

    <p id="status"></p>
  </div>

  <script>
    const oldTokenAddress = "0x0cf8e180350253271f4b917ccfb0accc4862f262"; // 17 decimals
    const migratorAddress = "0x36990197bf1a33ba7d682b21e1059d6bb7335b3d";

    const oldTokenABI = [
      { "constant": false, "inputs": [ { "name": "_spender", "type": "address" }, { "name": "_value", "type": "uint256" } ], "name": "approve", "outputs": [ { "name": "", "type": "bool" } ], "type": "function" },
      { "constant": true, "inputs": [], "name": "decimals", "outputs": [{ "name": "", "type": "uint8" }], "type": "function" }
    ];

    const migratorABI = [
      { "inputs": [{ "internalType": "uint256", "name": "oldAmount", "type": "uint256" }], "name": "migrate", "outputs": [], "stateMutability": "nonpayable", "type": "function" }
    ];

    let web3;
    let account;
    let oldDecimals = 17;

    async function connectWallet() {
      if (window.ethereum) {
        web3 = new Web3(window.ethereum);
        const accounts = await ethereum.request({ method: "eth_requestAccounts" });
        account = accounts[0];
        document.getElementById("wallet").innerText = "Connected: " + account;

        const token = new web3.eth.Contract(oldTokenABI, oldTokenAddress);
        oldDecimals = await token.methods.decimals().call();
      } else {
        alert("MetaMask not found");
      }
    }

    function toUnits(amount, decimals) {
      return web3.utils.toBN(web3.utils.toWei(amount.toString(), "ether"))
                      .div(web3.utils.toBN(10).pow(web3.utils.toBN(18 - decimals)));
    }

    async function approve() {
      const amount = document.getElementById("amount").value;
      if (!amount || amount <= 0) return alert("Enter a valid amount");

      const token = new web3.eth.Contract(oldTokenABI, oldTokenAddress);
      const amountInUnits = toUnits(amount, oldDecimals);

      try {
        await token.methods.approve(migratorAddress, amountInUnits.toString()).send({ from: account });
        document.getElementById("status").innerText = "✅ Approval successful!";
      } catch (err) {
        document.getElementById("status").innerText = "❌ Approval failed: " + err.message;
      }
    }

    async function migrate() {
      const amount = document.getElementById("amount").value;
      if (!amount || amount <= 0) return alert("Enter a valid amount");

      const migrator = new web3.eth.Contract(migratorABI, migratorAddress);
      const amountInUnits = toUnits(amount, oldDecimals);

      try {
        await migrator.methods.migrate(amountInUnits.toString()).send({ from: account });
        document.getElementById("status").innerText = "✅ Migration successful!";
      } catch (err) {
        document.getElementById("status").innerText = "❌ Migration failed: " + err.message;
      }
    }
  </script>
</body>
</html>
