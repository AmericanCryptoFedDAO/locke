# Token Smart Contract Documentation (Token.sol) (Human Readable)

## Key Features

### 1. Token Supply Limits

#### Maximum Supply:
The total supply of Locke tokens is capped at **1 trillion tokens**, ensuring a fixed limit on the total number of tokens that can ever exist on the Ethereum Blockchain.

#### Max Daily Supply Limit:
- **Max Daily Supply Limit = Base Supply:** + **Previous Day Claims:**
  - **Base Supply:** 50 million Locke tokens are available daily.
  - **Previous Day Claims:** Tokens claimed the previous day are added to the base supply.

#### Examples:
- **Example 1:**
  - **Day 1:** Daily supply = 50M tokens, with 40M claimed.
  - **Day 2:** Daily supply = 10M left; to maintain 50M tokens, 40M tokens will be automatically minted at 12:00 AM Mountain Time.

- **Example 2:**
  - **Day 1:** Daily supply = 50M tokens, with 40M claimed.
  - **Day 2:** Max daily supply limit = 50M + 40M (previous day’s claims).

#### Reset Mechanism:
If fewer tokens are claimed than available in a day, the difference is not carried forward. Daily limits reset every day at 12:00 AM Mountain Time.

---

### 2. Contributor Management

#### Identification:
Contributors are identified by an identifier (e.g., name, registration number) linked to a maximum of 3 wallet addresses.

#### Immutable Data:
Contributors cannot update or delete their identifiers or wallets to prevent fraud.

#### Token Allocation:
Tokens allocated to a contributor are shared across their registered wallets.

#### Example:
- Contributor has 3 wallets and is allocated 30 tokens.
  - Wallet #1 claims 7 tokens. Remaining balance: 23 tokens.
  - Wallet #2 claims 8 tokens. Remaining balance: 15 tokens.
  - Wallet #3 claims 6 tokens. Remaining balance: 9 tokens.

---

### 3. Claiming Tokens

#### Claim Limits:
Contributors can claim tokens daily, subject to limits.

#### Partial Claims:
If the requested amount exceeds the daily limit, only the available amount is processed.

#### Example:
- **Daily limit:** 75M tokens.
- **Request:** 80M tokens.
- **Processed:** 75M tokens. Remaining 5M is unfilled and cannot be carried over.

#### Output Messages:
- **Full Claim:**
  - “Congratulations, your claim of X tokens is successful. You have Y tokens remaining.”
- **Partial Claim:**
  - “Partial claim processed: X tokens claimed. Y tokens could not be processed due to daily limits.”

---

### 4. Minting Mechanism

#### Purpose:
Ensures the daily base supply (50M tokens) is maintained.
Mint on demand if claims exceed the current supply available but stay within the maximum daily supply.

#### Minting Formula 1 (Maintaining 50M daily supply):
- **Minted Tokens = Base Supply (50M) - Current Available Tokens.**

#### Examples:
1. **Day Start:** 40M tokens available.
   - **Minting Required:** 10M tokens.
   - **Result:** 50M tokens available for claims during the day.

2. **Claiming Above Supply:**
   - **Previous day claims:** 40M tokens.
   - **Current available tokens:** 40M.
   - **Base supply:** 50M.
   - **Maximum daily supply:** 50M + 40M = 90M.
   - **Minted Tokens:** 60M - 50M = 10M additional tokens.
   - **Total Daily Supply:** 60M.
   - **Claimed Tokens:** 60M.

---

### 5. Daylight Savings and Timing

#### Time Adjustments:
The contract adjusts for Mountain Time (MT) and accounts for Daylight Savings Time (DST).

#### Manual Override:
The owner can manually override DST settings if needed.

#### Next Day Calculation:
**Cutoff Time:** 11:59 PM MT.

#### Example:
- Current day ends at 11:59 PM MT.
- Next day starts at midnight MT.

---

## Additional Examples

### Maximum Daily Token Supply Limit Calculation:
- **Day 1:**
  - Base Supply = 50M.
  - No previous day claims (first day).
  - **Maximum Daily Supply Limit = 50M.**
- **Day 2:**
  - Base Supply = 50M.
  - Previous Day Claims = 15M.
  - **Maximum Daily Supply Limit = 50M + 15M = 65M.**
- **Day 3:**
  - Base Supply = 50M.
  - Previous Day Claims = 20M.
  - **Maximum Daily Supply Limit = 50M + 20M = 70M.**

### Token Claim by Contributor:
- **Contributor Allocation:** 30 tokens across 3 wallets.
  - Wallet #1 claims 7 tokens.
    - **Remaining Allocation:** 23 tokens.
  - Wallet #2 claims 8 tokens.
    - **Remaining Allocation:** 15 tokens.
  - Wallet #3 claims 6 tokens.
    - **Remaining Allocation:** 9 tokens.

---

## Error Messages

- **Unauthorized Wallet:**
  - “Unauthorized wallet.”
- **Exceeds Allocation:**
  - “Exceeds allocation.”
- **No Tokens Available Today:**
  - “No tokens available today.”
- **Partial Claim:**
  - “Partial claim processed: X tokens claimed. Y tokens could not be processed due to daily limits.”
- **Successful Claim:**
  - “Congratulations, your claim of X tokens is successful. You have Y tokens remaining.”

---

## Summary

The **Token.sol** smart contract combines advanced token minting, daily supply limits, contributor management, and streamlined claiming logic. It ensures fairness, security, and efficiency in token distribution while maintaining compliance with pre-defined limits and adjusting for mountain time with daylight savings logic.
