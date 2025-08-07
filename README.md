# TransferFrequencyTrap
# TransferFrequencyTrap — Drosera Smart Trap

This repository contains a working Drosera-compatible trap called `TransferFrequencyTrap`, which monitors the balance of a specific wallet and triggers an alert when a drop of **at least 0.03 ETH** is detected. It also includes the deployed alert receiver contract.

---

## 🔍 What This Trap Does

This trap observes a target Ethereum wallet. If the balance falls by **0.03 ETH or more** between two collected samples, `shouldRespond` returns `true`, and an alert is emitted through the configured `LogAlertReceiver`.

---

## 📁 Files

### `src/TransferFrequencyTrap.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface ITrap {
    function collect() external returns (bytes memory);
    function shouldRespond(bytes[] calldata data) external view returns (bool, bytes memory);
}

contract TransferFrequencyTrap is ITrap {
    /// Адрес кошелька, за которым следим
    address public constant target = 0x5dE1b376***********006C7a00eba66438F6890;

    /// Порог падения в wei (0.03 ETH)
    uint256 public constant dropThreshold = 0.03 ether;

    /// Сбор данных: возвращает текущий баланс
    function collect() external view override returns (bytes memory) {
        return abi.encode(target.balance);
    }

    /// Проверка: если баланс упал на 0.03 ETH или больше — срабатываем
    function shouldRespond(bytes[] calldata data) external pure override returns (bool, bytes memory) {
        if (data.length < 2) return (false, "Insufficient data");

        uint256 current = abi.decode(data[0], (uint256));
        uint256 previous = abi.decode(data[1], (uint256));

        if (previous > current && (previous - current) >= dropThreshold) {
            return (true, abi.encode("Balance dropped by at least 0.03 ETH"));
        }

        return (false, "");
    }
}
```

---

### `src/LogAlertReceiver.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LogAlertReceiver {
    event Alert(string message);

    function logTransferAlert(string calldata message) external {
        emit Alert(message);
    }
}
```

---

## ⚙️ drosera.toml Configuration

```toml
ethereum_rpc = "https://rpc.ankr.com/eth_hoodi/7d76ec4753f686fd*****************fee849faf82277de80688a850951a6"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.mytrap]
path = "out/TransferFrequencyTrap.sol/TransferFrequencyTrap.json"
response_contract = "0x1e71923035a2cA2812Ab96Ac3BCb44Ce55669781"
response_function = "logTransferAlert(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["0x5dE1b3767540357B38e006C7a00eba66438F6890"]
address = "0x9A80B5237de6E1d120762c7a6aEA12fB197365bA"
```

---

## 🚀 Deployment Commands

```bash
cd $HOME/my-drosera-trap

# Compile
forge build

# Deploy LogAlertReceiver
forge create \
  --rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --broadcast \
  --private-key <YOUR_PRIVATE_KEY> \
  src/LogAlertReceiver.sol:LogAlertReceiver
```

Output:

```
Deployer: 0x5dE1b3767540357B38e006C7a00eba66438F6890
Deployed to: 0x1e71923035a2cA2812Ab96Ac3BCb44Ce55669781
Transaction hash: 0xcd2304104b8fce9338df7550fa36be6a731ec5dfacd9f7d473330fdbf5caf4e5
```

---

## 🧪 Drosera Apply

```bash
# Load environment
source /root/.bashrc

# Apply trap
DROSERA_PRIVATE_KEY=<YOUR_PRIVATE_KEY> drosera apply
```

---

## ✅ Example Successful Execution (journalctl logs)

```
INFO drosera_services::operator::enzyme::runner: ShouldRespond='true'
INFO drosera_services::operator::attestation::runner: Generated attestation
INFO drosera_services::operator::attestation::runner: Reached signature threshold
INFO drosera_services::operator::submission::runner: Aggregated attestation result is 'shouldRespond=true'
INFO drosera_services::operator::submission::runner: Estimated gas: 743956
```

---

## 📡 Addresses

| Component             | Address |
|----------------------|---------|
| Trap contract         | `0x9A80B5237de6E1d120762c7a6aEA12fB197365bA` |
| LogAlertReceiver      | `0x1e71923035a2cA2812Ab96Ac3BCb44Ce55669781` |
| Trap operator wallet  | `0x5dE1b3767540357B38e006C7a00eba66438F6890` |

---

## 👨‍💻 Author

Trap created and deployed by **Lidiya Vlasenko** using Drosera + Forge stack on Ethereum Hoodi testnet.
