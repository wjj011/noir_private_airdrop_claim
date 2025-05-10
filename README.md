# Noir ZK Proof for Private Airdrop Claim

This project demonstrates a simplified Zero-Knowledge proof using Noir for enabling private and verifiable airdrop claims. Users can prove their eligibility for an airdrop based on certain criteria (represented by a commitment) using a private `eligibility_secret`, and generate a unique nullifier to prevent double-claiming, all without revealing the secret itself.

## Core Concept

An airdrop campaign has public `criteria_commitment` defining eligibility. A user eligible for the airdrop:
1.  Possesses an `eligibility_secret` (e.g., related to a past action or holding).
2.  Knows the `airdrop_id` for the specific campaign.
3.  Wishes to claim the airdrop to their `recipient_address` in the current `claim_epoch`.

The ZK circuit allows the user to generate a proof demonstrating:
*   They can form a valid `eligibility_proof_commitment` = `hash(eligibility_secret, airdrop_id, criteria_commitment)`. This links their secret to the specific airdrop and its rules.
*   They can form a unique `claim_nullifier` = `hash(eligibility_secret, airdrop_id, claim_epoch)`.

An airdrop smart contract or backend system would:
1.  Verify the ZK proof.
2.  Ensure the `criteria_commitment` matches the current campaign.
3.  (In a full system) Potentially check if the `eligibility_proof_commitment` corresponds to a pre-verified or more detailed eligibility proof. Our circuit simplifies this linkage.
4.  Check if the `claim_nullifier` has already been spent for this `airdrop_id` and `claim_epoch`.
5.  If all checks pass, distribute the airdrop to `recipient_address` and record the `claim_nullifier` as spent.

## Project Structure

(Standard Noir project structure)

## Circuit Logic (`src/main.nr`)

The Noir circuit takes:
*   **Private Inputs**: `eligibility_secret`, `airdrop_id`.
*   **Public Inputs**: `recipient_address`, `claim_epoch`, `criteria_commitment`, `claimed_eligibility_proof_commitment`, `claimed_claim_nullifier`.

It constrains:
1.  `claimed_eligibility_proof_commitment` equals `pedersen_hash([eligibility_secret, airdrop_id, criteria_commitment])`.
2.  `claimed_claim_nullifier` equals `pedersen_hash([eligibility_secret, airdrop_id, claim_epoch])`.

## Prerequisites

*   **Nargo**: Noir's package manager. Install from [official documentation](https://noir-lang.org/docs/getting_started/installation/).

## How to Run

1.  **Setup Project**: Create files and structure as outlined.
2.  **Crucial: Update Hash Placeholders**: The `claimed_eligibility_proof_commitment` and `claimed_claim_nullifier` in `Prover.toml` and `Verifier.toml` are **placeholders**. Recompute these using the private inputs in `Prover.toml` and the `pedersen_hash` logic.
3.  **Compile**: `nargo compile`
4.  **Generate Proof**: `nargo prove`
5.  **Verify Proof**: `nargo verify`

## Simplifications & Extensions

*   **Eligibility Detail**: This circuit uses a simplified eligibility proof commitment. A full implementation might involve a sub-circuit proving that `eligibility_secret` unlocks data satisfying complex criteria (e.g., Merkle proof of past transaction, proof of token balance).
*   **Airdrop Amount**: Assumes a fixed airdrop amount distributed externally. Variable amounts based on eligibility could be added.

This project provides a foundation for building privacy-preserving airdrop mechanisms that are resistant to Sybil attacks and double-claiming.