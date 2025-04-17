# MovieDOMToken (MDOM) Smart Contract Documentation

**Version:** 1.0  
**Contract Address:** (Deploy address TBD)  
**Source Code:** `contracts/MovieDOMTokenv2.sol`  
**Audit:** Hacken EEA EthTrust Report (April 4th, 2025) - M-Level Compliant

## 1. Overview and Purpose
The **MovieDOMToken (MDOM)** contract implements the core utility token for the **Moviedom** decentralized entertainment ecosystem. It serves as the primary medium for transactions related to content funding, distribution, monetization, NFT interactions, and access to services within the platform.  

This contract is designed as a standard, secure, and efficient ERC20 token, incorporating extensions for enhanced user experience and functionality.

## 2. Contract Logic and Mechanisms
The **MovieDOMToken.sol** contract builds upon audited and widely-used OpenZeppelin libraries to provide the following functionalities:

### 2.1. Core Standard: ERC20
The contract fully adheres to the **EIP-20** standard, ensuring compatibility with wallets, exchanges, and other decentralized applications within the Ethereum ecosystem.

It implements the following standard functions:
- `name()`: Returns the token name (e.g., "Moviedom").
- `symbol()`: Returns the token symbol (e.g., "MDOM").
- `decimals()`: Returns `18` (the standard number of decimal places).
- `totalSupply()`: Returns the total amount of MDOM tokens in existence.
- `balanceOf(address account)`: Returns the token balance of a specific account.
- `transfer(address recipient, uint256 amount)`: Transfers tokens from the caller's account to a recipient.
- `allowance(address owner, address spender)`: Returns the amount the spender is allowed to withdraw from the owner's account.
- `approve(address spender, uint256 amount)`: Allows the spender to withdraw tokens from the caller's account, up to the specified amount.
- `transferFrom(address sender, address recipient, uint256 amount)`: Transfers tokens from a sender's account to a recipient, provided the caller has sufficient allowance.

**Events:**  
It emits the standard `Transfer` and `Approval` events upon successful execution of the respective actions.

### 2.2. Gasless Approvals: ERC20Permit (EIP-2612)
The contract inherits `ERC20Permit` from OpenZeppelin, implementing the **EIP-2612** standard for gasless approvals.

- Users approve token spending via off-chain signatures instead of an on-chain `approve` transaction.
- A user signs a message containing details like the owner, spender, amount, nonce, and deadline, according to the **EIP-712** standard.
- This signed message can be relayed to the contract by any third party (the "relayer" or potentially the spender).
- The `permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)` function verifies the signature against the **EIP-712** domain separator and the signer's nonce.

Includes:
- `nonces(address owner)` to prevent signature replay attacks.
- `DOMAIN_SEPARATOR()` for EIP-712 signature verification context.

### 2.3. Token Burning
The contract provides two methods for burning tokens, effectively removing them from circulation:

- `burn(uint256 amount)`:  
  Allows any token holder to destroy a specified amount of their own tokens, decreasing their balance and the total supply.

- `burnFrom(address account, uint256 amount)`:  
  Allows an address (_msgSender()) to burn tokens on behalf of another account, provided the caller has been granted sufficient allowance by that account.

**Events:**
- Emits a `Transfer` event to the zero address and an `Approval` event for the reduced allowance when burning tokens.

### 2.4. Initial Supply and Minting
- The contract defines a fixed **INITIAL_SUPPLY** of **1,000,000,000 MDOM tokens** (1 billion).
- This is represented as `1000000000 * 10 ** 18` due to 18 decimals.
- Upon deployment, the constructor mints the entire **INITIAL_SUPPLY** to the owner address provided during deployment.
- **No further minting functionality** exists in the contract.

### 2.5. Security and Inheritance
- The contract leverages **OpenZeppelin Contracts (v4.x or later)**, which are industry-standard and heavily audited.
- It uses **Solidity version 0.8.28**, which includes default checks for arithmetic overflow and underflow, enhancing security (except where explicitly and safely bypassed using `unchecked`).

## 3. Contract Specifications
The **MovieDOMToken.sol** contract MUST adhere to the following specifications:

### General
- **SPEC-GEN-01**: MUST use Solidity compiler version `0.8.28`.
- **SPEC-GEN-02**: MUST use the **MIT** license identifier.
- **SPEC-GEN-03**: MUST rely on OpenZeppelin's audited implementations for ERC20 and ERC20Permit base functionality.

### ERC20 Compliance
- **SPEC-ERC20-01**: MUST implement all functions and events defined in the EIP-20 standard interface.
- **SPEC-ERC20-02**: `decimals()` MUST return `18`.
- **SPEC-ERC20-03**: `transfer` MUST revert if the caller's balance is less than the amount.
- **SPEC-ERC20-04**: `transferFrom` MUST revert if the sender's balance is less than the amount.
- **SPEC-ERC20-05**: `transferFrom` MUST revert if the caller's allowance for the sender is less than the amount.
- **SPEC-ERC20-06**: MUST emit `Transfer` event upon successful `transfer`, `transferFrom`, `_mint`, and `_burn` operations.
- **SPEC-ERC20-07**: MUST emit `Approval` event upon successful `approve` and allowance updates in `transferFrom` / `burnFrom`.

### ERC20Permit (EIP-2612) Compliance
- **SPEC-PERMIT-01**: MUST implement the EIP-2612 interface (`permit`, `nonces`, `DOMAIN_SEPARATOR`).
- **SPEC-PERMIT-02**: `permit` function MUST validate the provided signature using **EIP-712** verification.
- **SPEC-PERMIT-03**: `permit` function MUST revert if the signature is invalid or does not match the owner.
- **SPEC-PERMIT-04**: `permit` function MUST revert if the deadline has passed.
- **SPEC-PERMIT-05**: `permit` function MUST update the allowance mapping for the owner and spender upon successful validation.
- **SPEC-PERMIT-06**: `permit` function MUST increment the owner's nonce upon successful validation to prevent replay attacks.
- **SPEC-PERMIT-07**: MUST provide a valid `DOMAIN_SEPARATOR` based on **EIP-712** requirements.

### Constructor and Initial Supply
- **SPEC-INIT-01**: The constructor MUST accept `name` (string), `symbol` (string), and `owner` (address) as arguments.
- **SPEC-INIT-02**: The constructor MUST set the token's name and symbol as provided.
- **SPEC-INIT-03**: The constructor MUST set the **EIP-712 domain name** used by ERC20Permit based on the provided name.
- **SPEC-INIT-04**: The constructor MUST mint the **INITIAL_SUPPLY** of `1,000,000,000 * 10 ** 18` tokens.
- **SPEC-INIT-05**: The constructor MUST assign the entire **INITIAL_SUPPLY** to the owner address provided.
- **SPEC-INIT-06**: **INITIAL_SUPPLY** MUST be a constant value.
- **SPEC-INIT-07**: There MUST be no other function capable of minting new tokens.

### Burning Functions
- **SPEC-BURN-01**: `burn(uint256 amount)` MUST decrease the caller's (`_msgSender()`) balance by amount.
- **SPEC-BURN-02**: `burn(uint256 amount)` MUST decrease the total supply by amount.
- **SPEC-BURN-03**: `burn(uint256 amount)` MUST revert if the amount is greater than the caller's balance.
- **SPEC-BURN-04**: `burnFrom(address account, uint256 amount)` MUST decrease the account's balance by amount.
- **SPEC-BURN-05**: `burnFrom(address account, uint256 amount)` MUST decrease the total supply by amount.
- **SPEC-BURN-06**: `burnFrom` MUST revert if the caller's (`_msgSender()`) allowance for account is less than amount.
- **SPEC-BURN-07**: `burnFrom` MUST decrease the caller's allowance for account by amount upon successful burn.
- **SPEC-BURN-08**: `burnFrom` MUST revert if the amount is greater than the account's balance.

### Security
- **SPEC-SEC-01**: Arithmetic operations MUST be safe from overflow/underflow (guaranteed by Solidity >=0.8.0, except for the specified safe use of `unchecked` in `burnFrom` after allowance validation).
- **SPEC-SEC-02**: The contract MUST NOT be vulnerable to reentrancy attacks on its implemented functions (inherent protection from standard OpenZeppelin ERC20 patterns).
