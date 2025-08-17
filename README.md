# ZKVM with Circom and SnarkJS (Simple Stack Based zkVM)

This repository demonstrates how to build and test a simple **Zero-Knowledge Virtual Machine (ZKVM)** using [Circom](https://docs.circom.io/) for circuit design and [SnarkJS](https://github.com/iden3/snarkjs) for generating and verifying zk-SNARK proofs.

---

## 📦 Prerequisites

Ensure you have the following installed:

* **Node.js (>= 16.x)**
* **npm** (comes with Node.js)
* **Circom v2** ([installation guide](https://docs.circom.io/getting-started/installation/))
* **SnarkJS** (install globally)

  ```bash
    Sudo npm install -g snarkjs
  ```
* **Vim or any text editor** (for editing `input.json`)

---

## ⚙️ Project Setup

Clone this repo and install dependencies:

```bash
git clone <your-repo-url>
cd <your-repo>
```

### Project Structure

```
zkvm.circom       # The main Circom circuit file
input.json        # Input values for witness generation
README.md         # This documentation
```

---

## 📝 Step 1: Write Circuit (`zkvm.circom`)

Example `zkvm.circom`:

```circom
include "node_modules/circomlib/comparators.circom";

// Simple stack-based VM logic here
component main = ... ;
```

⚠️ If you use external libraries (like `circomlib`), make sure they are installed:

```bash
npm install circomlib
```

---

## 📝 Step 2: Define Inputs (`input.json`)

Create an `input.json` file with Vim or another editor:

```bash
vim input.json
```

Example content:

```json
{
  "instr": [1,3,1,6,1,2,3,0,3,0]
}
```

Save and exit with `:wq`.

---

## ⚡ Step 3: Compile the Circuit

```bash
circom zkvm.circom --r1cs --wasm --sym -l node_modules/circomlib/circuits
```

This generates:

* `zkvm.r1cs` → circuit constraints
* `zkvm_js/` → witness generation code
* `zkvm.sym` → symbols file

---

## 🛠️ Step 4: Generate Witness

Run:

```bash
node zkvm_js/generate_witness.js zkvm_js/zkvm.wasm input.json witness.wtns
```

---

## 🔑 Step 5: Trusted Setup

### 5.1 Powers of Tau

Initialize:

```bash
snarkjs powersoftau new bn128 12 pot12_0000.ptau -v
```

Contribute:

```bash
snarkjs powersoftau contribute pot12_0000.ptau pot12_0001.ptau --name="First contribution" -v
```

Prepare for phase 2:

```bash
snarkjs powersoftau prepare phase2 pot12_0001.ptau pot12_final.ptau -v
```

### 5.2 Circuit-specific Setup

```bash
snarkjs groth16 setup zkvm.r1cs pot12_final.ptau zkvm_0000.zkey
```

Add contribution:

```bash
snarkjs zkey contribute zkvm_0000.zkey zkvm_final.zkey --name="Key Contributor" -v
```

Export verification key:

```bash
snarkjs zkey export verificationkey zkvm_final.zkey verification_key.json
```

---

## ✅ Step 6: Generate Proof

```bash
snarkjs groth16 prove zkvm_final.zkey witness.wtns proof.json public.json
```

---

## 🔍 Step 7: Verify Proof

```bash
snarkjs groth16 verify verification_key.json public.json proof.json
```

Expected output:

```
OK!
```

---

## 🌐 Step 8: Export Solidity Verifier (Optional)

```bash
snarkjs zkey export solidityverifier zkvm_final.zkey Verifier.sol
```

Compile and deploy `Verifier.sol` to Ethereum or compatible chains to perform on-chain verification.

---

## 📚 Troubleshooting

* **`EACCES: permission denied, mkdir '/usr/local/lib/node_modules'`** → Install with `sudo` or configure npm prefix:

  ```bash
  mkdir ~/.npm-global
  npm config set prefix '~/.npm-global'
  export PATH=~/.npm-global/bin:$PATH
  ```
* **`The file circomlib/comparators.circom not found`** → Add include path `-l node_modules/circomlib/circuits`
* **`[ERROR] snarkJS: Powers of tau is not prepared`** → Run Phase 1 & Phase 2 trusted setup first.

---

## 📖 References

* [Circom Documentation](https://docs.circom.io/)
* [SnarkJS GitHub](https://github.com/iden3/snarkjs)
* [Awesome Zero-Knowledge](https://github.com/matter-labs/awesome-zero-knowledge-proofs)

---

🚀 You now have a working ZKVM setup with Circom + SnarkJS!
