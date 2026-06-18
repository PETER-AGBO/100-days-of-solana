# Creating a Compliance-Gated Token with Default Frozen Accounts

# Step 1: Create a mint with default frozen accounts
spl-token create-token \
  --program-id TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb \
  --enable-freeze \
  --default-account-state frozen
# Save your mint address from the output
# Example: MINT_ADDRESS=[your-mint-address]
# Step 2: Create two token accounts
spl-token create-account [MINT_ADDRESS]
# Generate a second keypair to simulate a second user
solana-keygen new --outfile ~/second-wallet.json --no-bip39-passphrase --force
SECOND_WALLET=$(solana-keygen pubkey ~/second-wallet.json)
spl-token create-account [MINT_ADDRESS] --owner $SECOND_WALLET --fee-payer ~/.config/solana/id.json
# Step 3: Try to mint to your (frozen) account — this will FAIL
spl-token mint [MINT_ADDRESS] 100
# You should see an error: "Account is frozen"
# Step 4: Thaw your own token account
spl-token thaw [YOUR_TOKEN_ACCOUNT_ADDRESS]
# Step 5: Mint tokens to your now-thawed account
spl-token mint [MINT_ADDRESS] 100
# Step 6: Try to transfer to the second (still frozen) account — this will FAIL
spl-token transfer [MINT_ADDRESS] 50 $SECOND_WALLET --allow-unfunded-recipient
# You should see an error because the destination is frozen
# Step 7: Thaw the second account, then transfer
spl-token thaw [SECOND_TOKEN_ACCOUNT_ADDRESS]
spl-token transfer [MINT_ADDRESS] 50 $SECOND_WALLET --allow-unfunded-recipient
# Verify balances
spl-token accounts --owner $SECOND_WALLET