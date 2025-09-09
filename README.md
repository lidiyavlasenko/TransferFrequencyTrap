# TransferFrequencyTrap
# TransferFrequencyTrap — Drosera Smart Trap

The trap captures any decrease in the balance of the observed wallet and provides details about how much and when the decrease occurred.

---

## 🔍 What This Trap Does

Determining a balance drop
If the balance has decreased by at least dropThreshold (in the code, this is the minimum value of 1 wei), the trap considers this a significant event.

---

## 📁 Files

### `src/TransferFrequencyTrap.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface ITrap {
    function collect() external view returns (bytes memory);
    function shouldRespond(bytes[] calldata data) external view returns (bool, bytes memory);
}

contract TransferFrequencyTrap is ITrap {
    /// The wallet address we are tracking (test)
    address public constant target = 0x5dE1b3767000000000006C7a00eba66438F6890;

    /// Drop threshold in wei
    uint256 public immutable dropThreshold = 1; // 1 wei

    /// Data collection: returns the current balance
    function collect() external view override returns (bytes memory) {
        return abi.encode(target.balance);
    }

    /// Check: if the balance drops below threshold or more, we trigger
    function shouldRespond(bytes[] calldata data) external pure override returns (bool, bytes memory) {
        if (data.length < 2) {
            return (false, abi.encode("Insufficient data"));
        }

        uint256 current = abi.decode(data[0], (uint256));
        uint256 previous = abi.decode(data[1], (uint256));

        if (previous == 0) {
            return (false, abi.encode("Previous balance is zero"));
        }

        if (previous > current && (previous - current) >= dropThreshold) {
            uint256 drop = previous - current;

            // Returning arguments for logTransferAlert(address,uint256,uint256,uint256,string)
            return (
                true,
                abi.encode(
                    target,        // address
                    previous,      // uint256
                    current,       // uint256
                    drop,          // uint256
                    "Balance dropped" // string
                )
            );
        }

        return (false, abi.encode("No significant drop"));
    }
} 

```

---

### `src/LogAlertReceiver.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LogAlertReceiver {
    event Alert(
        address indexed target,
        uint256 prevBalance,
        uint256 currBalance,
        uint256 drop,
        string message
    );

    function logTransferAlert(
        address target,
        uint256 prevBalance,
        uint256 currBalance,
        uint256 drop,
        string calldata message
    ) external {
        emit Alert(target, prevBalance, currBalance, drop, message);
    }
}
```

---

## ⚙️ drosera.toml Configuration

```toml
ethereum_rpc = "https://rpc.ankr.com/eth_hoodi/7d76ec4753f686fd*****************fee849faf82277de80688a850951a6"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaF********e056F5a9b1F14bb06e5D"

[traps]

[traps.mytrap]
path = "out/TransferFrequencyTrap.sol/TransferFrequencyTrap.json"
response_contract = "0x1e71923035********6Ac3BCb44Ce55669781"
response_function = "logTransferAlert(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["0x5dE1b3767*******06C7a00eba66438F6890"]
address = "0x9A80B5237de*********2c7a6aEA12fB197365bA"
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
Deployer: 0x5dE1b376754*******e006C7a00eba66438F6890
Deployed to: 0x1e7192303**********2Ab96Ac3BCb44Ce55669781
Transaction hash: 0xcd2304104b8fce9338***********31ec5dfacd9f7d473330fdbf5caf4e5
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
