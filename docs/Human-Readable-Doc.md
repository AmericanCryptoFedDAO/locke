# Token Smart Contract Documentation (Token.sol)

## Overview

This document explains how the smart contract dynamically manages Locke Token distribution to enforce daily claim limits efficiently. It includes examples, definitions, and details, as well as handling caching, transitions, skipped days, partial claims, and gas fee responsibilities.

## Key Features

### 1. Token Supply Limits

#### Units Abbreviation:
- **Trillion:** T
- **Million:** M

#### Maximum Supply:
The total supply of Locke tokens is capped at **1 Trillion tokens** or (1T), ensuring a fixed limit on the total number of tokens that can ever exist on the Ethereum Blockchain.

#### Max Daily Claim Limit:
The **Max Daily Claim Limit** determines the maximum number of Locke Tokens that contributors can claim in a single day. This value is recalculated dynamically based on the **Base Claim Limit** and the **Total Previous Day Claims.** This ensures contributors can claim a fair amount each day. It is calculated using a fixed, algorithmic formula that scales predictably with the claims from the previous day.

- **Formula:**
  ```
  Max Daily Claim Limit = Base Claim Limit (50M) + Total Previous Day Claims
  ```
- **Base Claim Limit:** 50 million (50M) Locke tokens are available as a fixed capacity per day.
- **Total Previous Day Claims:** The actual number of tokens claimed during the previous day.

#### Examples:
- **Example 1:**
  - **Day 1:** Base Claim Limit 50 million (50M) Locke tokens are available, while Contributors claim 10M tokens on Day1.
    - **Total Previous Day Claims:** 0M.
    - **Max Daily Claim Limit on Day 1:** 50M (Base Claim Limit).
    - **Total Current Day Claims**: 10M
    - **The Actual Number of Token Claimed on Day 1**: 10M
    - **Unfilled Claims**: 0M.

- **Example 2:**
  - **Day 2:** Base Claim Limit 50 million (50M) Locke tokens are available, while Contributors claim 0M tokens on Day 2.
    - **Total Previous Day Claims:** 10M.
    - **Max Daily Claim Limit on Day 2:** 50M (Base Claim Limit) + 10M = 60M.
    - **Total Current Day Claims**: 0M
    - **The Actual Number of Token Claimed on Day 2**: 0M
    - **Unfilled Claims**: 0M.

- **Example 3:**
  - **Day 3:** Base Claim Limit 50 million (50M) Locke tokens are available, while
Contributors claim 25M tokens on Day 3.
    - **Total Previous Day Claims:** 0M.
    - **Max Daily Claim Limit on Day 3:** 50M (Base Claim Limit).
    - **Total Current Day Claims**: 25M
    - **The Actual Number of Token Claimed on Day 3**: 25M
    - **Unfilled Claims**: 0M.

- **Example 4:**
  - **Day 4:** Base Claim Limit 50 million (50M) Locke tokens are available, while Contributors claim 95M tokens on Day 4.
    - **Total Previous Day Claims:** 25M.
    - **Max Daily Claim Limit on Day 4:** 50M (Base Claim Limit) + 25M = 75M.
    - **Total Current Day Claims:** 95M.
    - **The Actual Number of Token Claimed on Day 4:** 75M
    - **Unfilled Claims:** 20M.

- **Example 5:**
  - **Day 5:** Base Claim Limit 50 million (50M) Locke tokens are available, while Contributors claim 120M tokens on Day 5.
    - **Total Previous Day Claims:** 75M.
    - **Max Daily Claim Limit on Day 5:** 50M (Base Claim Limit) + 75M = 125M
    - **Total Current Day Claims:** 120M.
    - **The Actual Number of Token Claimed on Day 5:** 120M
    - **Unfilled Claims:** 0M.

- **Example 6:**
  - **Day 6:** Base Claim Limit 50 million (50M) Locke tokens are available, while Contributors claim 200M tokens on Day 6.
    - **Total Previous Day Claims:** 120M.
    - **Max Daily Claim Limit on Day 6:** 50M (Base Claim Limit) + 120M = 175M
    - **Total Current Day Claims:** 200M.
    - **The Actual Number of Token Claimed on Day 6:** 175M
    - **Unfilled Claims:** 25M.

- **Example 7:**
  - **Day 7:** Base Claim Limit 50 million (50M) Locke tokens are available, while Contributors claim 210M tokens on Day 7.
    - **Total Previous Day Claims:** 175M.
    - **Max Daily Claim Limit on Day 7:** 50M (Base Claim Limit) + 175M = 225M
    - **Total Current Day Claims:** 210M.
    - **The Actual Number of Token Claimed on Day 7:** 210M
    - **Unfilled Claims:** 0M.
---

### 2. Contributor Management (Contributor's Identifier Account)

#### Identification:
Contributors are identified by an identifier (e.g., individual name and Wyoming Driver License or entity name and Wyoming entity registration number) linked to a maximum of 3 wallet eth addresses (public keys).

#### Immutable Data for Contributors Only:
- Contributors cannot update or delete their identifiers or wallets to prevent fraud.
- Owner of the smart contract (CryptoFed) can add, update **Contributor's Identifier Account** including 1-3 wallet eth addresses (public keys), but CryptoFed never knows the private keys for any of the three wallets.
  - **Contributor's Identifier Account** must have at least 1 wallet eth address for the account to be established or added.

#### Token Allocation:
Tokens allocated to a contributor are shared across their registered wallets.

#### Example:
- Contributor has 3 wallets and is allocated 30 tokens.
  - Wallet #1 claims 7 tokens. Remaining balance: 23 tokens.
  - Wallet #2 claims 8 tokens. Remaining balance: 15 tokens.
  - Wallet #3 claims 6 tokens. Remaining balance: 9 tokens.

### 2A. Get Contributor Claim Information (`getContributorClaimInfo`):
This function, when called upon, provides real-time information about a contributor's claim allocation, the system-wide remaining claim availability for the day, and wallet-specific claim details. It is designed for contributors to understand their claimable tokens and the system's current state at any given time.

The **Get Contributor Claim Information (`getContributorClaimInfo`)** function retrieves the following real-time details for a specific contributor:

1. **Max Daily Claim Limit**: The total number of tokens available for claims across all contributors on a given day.
2. **Total Remaining Day Claim**: The number of tokens still available to be claimed system-wide for the day.
3. **Wallet-Specific Claim Information**:
   - The number of wallets registered for the contributor.
   - The number of tokens already claimed for each wallet.
4. **Claim Allocation**: The maximum tokens allocated to the contributor based on their registration details.
5. **Current Estimated Available Allocated Claim**:
   - Defined as the minimum of the contributor's allocation and the remaining system-wide availability:
     ```
     Current Estimated Available Allocated Claim = Min(Your Claim Allocation, Total Remaining Day Claim)
     ```
#### Key Notes

- This function **does not reserve tokens** for the contributor. The information provided reflects the state at the time of the call.
- The **Estimated Available Allocated Claim** is dynamic and subject to change as other contributors process their claims. It is not guaranteed to remain the same when the claim is processed.
- **Input Requirement**: The function can be called using the contributor's wallet address or identifier.

#### Function Output Breakdown

When called, the function returns the following (Output Breakdown):

1. **Max Daily Claim Limit** (System-wide): The maximum tokens available to claim for all contributors on the current day.
   - Sample output:
   ```
      Max Daily Claim Limit: 100M tokens
   ```
2. **Total Remaining Day Claim** (System-wide): The number of tokens still available for the day.
   - Sample output:
   ```
      Total Remaining Day Claim: 10M tokens
   ```   
3. **Wallet Claim Details**:
   - Number of registered wallets.
   - Tokens claimed by each wallet eth address.

   - Sample output:
   ```
      Registered Wallets: 3
      Tokens Claimed:
      Wallet 1 (0x1234...abcd): 5M tokens
      Wallet 2 (0x5678...efgh): 3M tokens
      Wallet 3 (0x9abc...ijkl): 2M tokens
   ```
   
 4. **Total Claim Allocation** (Contributor-Specific): The maximum number of tokens allocated to the calling contributor.
    - Sample output:
    ```
      Total Claim Allocation: 25M tokens
    ```
 5. **Unclaimed Allocation** (Contributor-Specific): The maximum number of tokens unclaimed to the calling contributor.
    - Sample output:
    ```
      Unclaimed Allocation: 15M tokens
    ```
6. **Current Estimated Available Allocated Claim** (Contributor-Specific): The maximum number of tokens the contributor can claim at that moment, considering both their allocation and the system-wide remaining tokens.
   - Sample output:
   ```
     Current Estimated Available Allocated Claim: 10M tokens
   ```

#### Example: Contributor Claim Query
  
  - **Scenario**:
      - Contributor calls `getContributorClaimInfo` at 2:00 PM MT
        - with contributor's wallet eth address
       
      - **Result**:
      ```
          System-Wide Information:
          Max Daily Claim Limit: 100M tokens
          Total Remaining Day Claim: 10M tokens
          
          Contributor Information:
          Registered Wallets: 3
          Tokens Claimed:
          Wallet 1 (0x1234...abcd): 5M tokens
          Wallet 2 (0x5678...efgh): 3M tokens
          Wallet 3 (0x9abc...ijkl): 2M tokens

          Total Claim Allocation: 25M tokens
          Unclaimed Allocation: 15M tokens
          Current Estimated Available Allocated Claim: 10M tokens
      ```
---

### 3. Claiming Tokens

#### Claim Limits:
Contributors can claim tokens dynamically, subject to the current calculated **Max Daily Claim Limit.**

#### Same Day Claims:
- **How it works**: 
   - Multiple claims made on the same day are added to the **Total Current Day Claims**.
   - The **Total Previous Day Claims** remains **cached** and is not updated during the same day.

#### New Day Transition:
- **How it works**: 
   - When a claim is made on a new day:
     - The **Total Current Day Claims** is copied to the **Total Previous Day Claims**.
     - The **Total Current Day Claims** is reset to 0.
   - This update happens only once per day during the first claim of the new day.

#### Skipped Days:
- **How it works**: 
   - If no claims occur for one or more days:
     - The **Total Previous Day Claim** is updated to 0 during the next claim, dynamically reflecting no claims for skipped days.
     - The **Total Current Day Claim** resets for the new day.
    
#### Partial Claims:
- **How it works**: 
   - If a claim exceeds the Max Daily Claim Limit, only the allowable amount is processed.
   - Unfulfilled claims must be manually re-requested. They are not carried over to the next day automatically.

#### Example:
- **Max Daily Claim Limit:** 80M tokens.
- **Claimant 1 Request**: 20M tokens → Processed: 20M tokens
- **Remaining Daily Limit**: 60M tokens
####
- **Claimant 2 Request**: 90M tokens → Processed: 60M tokens
- **Unfilled Claim**: 30M tokens (cannot be carried over to the next day)

##### Output Messages:
- **Full Claim:**
  - “Congratulations, your claim of X tokens is successful. You have Y tokens remaining.”
- **Partial Claim:**
  - “Partial claim processed: X tokens claimed. Y tokens could not be processed due to daily limits.”

#### More Examples:

- #### A. Transitioning Total Current Day Claim to Total Previous Day Claim
  - **Scenario**:
    - **Date**: November 2, 2024 (Mountain Time).
    - **Base Claim Limit**: 50M tokens.
    - **Previous Day Claims**: 30M tokens.
    - **Max Daily Claim Limit**:
      ```
      50M (Base) + 30M = 80M
      ```

  **Process**:
  - A contributor claims 40M tokens during the day:
    - Add to **Total Current Day Claim**: `{date: November 2, value: 40M}`.
    - **Total Previous Day Claim** remains cached: `{date: November 1, value: 30M}`.
  - The same contributor claims another 20M tokens later in the day:
    - Add to **Total Current Day Claim**: `{date: November 2, value: 60M}`.
  - On November 3:
    - The first claim triggers the update:
      - **Total Previous Day Claim** is updated with the value of **Total Current Day Claim**:
        ```
        {date: November 2, value: 60M}
        ```
      - **Total Current Day Claim** is reset:
        ```
        {date: November 3, value: 0}
        ```

  **Result**:
  - **Total Previous Day Claim** is updated to reflect the total claims made on November 2.
  - **Total Current Day Claim** starts fresh for November 3.

- #### B. Fully Claimed Day
  **Date**: November 2, 2024 (Mountain Time).
  - **Scenario**:
    - Base Claim Limit = 50M tokens.
    - Total Previous Day Claims = 0M tokens.
    - **Max Daily Claim Limit**:
      ```
      50M + 0M = 50M
      ```
  
  **Process**:
  1. A contributor requests a claim for **50M tokens**, which matches the **Max Daily Claim Limit** for November 2.
     - **Result**:
       - **Total Current Day Claim**: `{date: November 2, value: 50M}`.
       - No more claims are allowed for November 2 since the **Max Daily Claim Limit** has been reached.
  
  2. **On November 3**:
     - The first claim of the day triggers the update:
       - **Total Previous Day Claim** updates to:
         ```
         {date: November 2, value: 50M}
         ```
       - **Total Current Day Claim** resets to:
         ```
         {date: November 3, value: 0}
         ```
     - **Max Daily Claim Limit for November 3**:
       ```
       50M (Base) + 50M (Previous Day Claims) = 100M
       ```
  
  **Result**:
  - November 3 begins with a Max Daily Claim Limit of **100M tokens**, reflecting the full claims made on November 2.

- #### C. Skipped Days
  **Scenario**:
    - Last claim: November 2, 2024 (40M tokens).
    - No claims on November 3 or 4, 2024.
    - First claim on November 5, 2024 (50M tokens).
  
  **Process**:
  - On November 5, the system detects skipped days:
    - **Total Previous Day Claim** is updated once:
      ```
      {date: November 4, value: 0}
      ```
    - **Total Current Day Claim** is reset:
      ```
      {date: November 5, value: 0}
      ```
  - Process the new claim:
    - Add 50M to **Total Current Day Claim**: `{date: November 5, value: 50M}`.
  - **Max Daily Claim Limit for November 5**:
    ```
    50M (Base) + 0M (Previous Day Claims) = 50M
    ```
  
  **Result**:
  - Skipped days dynamically reset the **Total Previous Day Claim** to 0.

#### Key Considerations:

##### Efficient Updates:
- The **Total Previous Day Claim** is updated only once per day, during the first claim of a new day or after skipped days.
- Cached for multiple claims on the same day.

##### Gas Responsibility:
- Contributors must pay gas fees for each claim or minting operation.
- The CryptoFed or the owner of the smart contract does not cover gas fees.

##### Handling Partial Claims:
- Unfulfilled amounts are not automatically carried forward. Contributors must re-request their claim on subsequent days.

---

### 4. Minting Mechanism

#### Purpose:
- Tokens are minted directly to contributors’ selected Ethereum wallet addresses on demand.
- The minting process ensures that tokens are created only when contributors claim them, complying with the **Max Daily Claim Limit**.
- **Total Token Supply:**
  - Tracks the total number of tokens minted so far.
  - **Purpose:**
    1. **ERC-20 Compliance:** The `totalSupply()` function is required by the ERC-20 standard, providing transparency about the total number of tokens in circulation.
    2. **Max Supply Enforcement:** Ensures the total tokens minted never exceed the defined maximum limit (e.g., 1 trillion tokens).
    
#### Dynamic Calculation of Total Previous Day Claims:
- The **`daysElapsed`** value determines whether the **Total Previous Day Claims** should be carried forward or reset to `0` based on the time elapsed since the last claim or mint.

- **Formula to Determine Days Elapsed:**
  - **1 day = 86400 seconds**
  - **daysElapsed = (Current Time - Last Claim Time) / 1 Day**

- **Status Labels for `daysElapsed`:**
  - **`daysElapsed == 0 (status: Same Day)`**: The current day is the same as the last claim day, so the **Total Previous Day Claims** stay unchanged.
  - **`daysElapsed == 1 (status: Consecutive Day)`**: The current day is directly after the last claim day; the **Total Previous Day Claims** remain valid.
  - **`daysElapsed > 1 (status: Skipped Days)`**: Skipped days occurred; the **Total Previous Day Claims** are reset to `0` because no claims occurred during those days.

#### Examples:

##### Example 1: Same Day
- **Day 1 Claims:** 40M tokens in the morning.
- **Day 1 Claims (Afternoon):** Contributor claims another 20M tokens.
  - **daysElapsed = 0 (status: Same Day)**.
  - **Total Previous Day Claims = 0 (as no claims occurred on Day 0)**.
  - **Max Daily Claim Limit:** 50M (Base Claim Limit) + 0 (Total Previous Day Claims) = 50M.

##### Example 2: Consecutive Day
- **Day 1 Claims:** 30M tokens.
- **Day 2 Claims:** 15M tokens.
  - **daysElapsed = 1 (status: Consecutive Day)**.
  - **Total Previous Day Claims = 30M (from Day 1)**.
  - **Max Daily Claim Limit:** 50M (Base Claim Limit) + 30M (Total Previous Day Claims) = 80M.

##### Example 3: Skipped Days
- **Day 1 Claims:** 25M tokens.
- **Day 2-4:** No claims.
- **Day 5 Claims:** 20M tokens.
  - **daysElapsed = 4 (status: Skipped Days)**.
  - **Total Previous Day Claims = 0 (reset due to skipped Days 2-4)**.
  - **Max Daily Claim Limit:** 50M (Base Claim Limit) + 0 (Total Previous Day Claims) = 50M.

---

### 5. Daylight Savings and Timing

Ethereum uses **UTC timestamps** by default because it is a global standard and simplifies blockchain operations. Mountain Time (MT) tracking, including **Daylight Savings Time (DST)** adjustments, is implemented by converting UTC to MT.

#### **How to Convert UTC to Mountain Time**

Mountain Time operates on two predefined offsets, designed to optimize gas efficiency:
- **DST (Summer):** UTC-6
- **Standard Time:** UTC-7

To convert:
1. **Predefined DST Offset Rules (UTC to Mountain Time)**
  - The smart contract **automatically adjusts** for Daylight Saving Time (DST) based on predefined rules:
    - If DST is active, subtract **6 hours** from UTC.
    - If not, subtract **7 hours** from UTC.

  - **Key Notes:**
    - **No Gas Cost for Reading Information**:
      - When contributors or users access information (e.g., checking claim details), the smart contract performs the computation without incurring gas costs.
    - **Minimal Gas Impact for Transactions:**  
      - The computation for converting time (e.g., from UTC to Mountain Time) is included in the overall transaction process and has a minimal effect on gas fees. This ensures that the conversion does not significantly increase the cost of claiming tokens or other operations.

2. **Manual Override:** The smart contract includes a **manual override** for DST as a fail-safe. While the system automatically adjusts for DST based on predefined rules, the override provides flexibility for unexpected changes, such as the government removing or modifying DST policies. If DST remains unchanged, the smart contract automatically handles DST without requiring the override.

#### **Calculating `daysElapsed`**

`daysElapsed` measures full days between two timestamps based on Mountain Time:

##### **Formula**:
```
1 day = 86400 seconds
daysElapsed = (toMountainTime(currentTimestamp) / 1 day) - (toMountainTime(lastClaimTimestamp) / 1 day)
```

##### **Steps**:
1. Convert UTC timestamps to MT.
2. Divide each by 1 day (86400 seconds).
3. Subtract to get the difference.

##### **Example**
- **Last Total Previous Claim Timestamp:** November 2, 2024, 11:00 PM UTC → November 2, 2024, 5:00 PM MDT (Mountain Daylight Time).
- **Current Claim Timestamp:** November 4, 2024, 2:00 AM UTC → November 3, 2024, 7:00 PM MST (Mountain Standard Time).
- **Result:** `daysElapsed = 2`.
- **Total Previous Day Claims = 0 (reset due to 1 skipped Day)**.
- **Max Daily Claim Limit:** 50M (Base Claim Limit) + 0 (Total Previous Day Claims) = 50M.

#### **Key Points**
- **Mountain Time Conversion:** Adjusts UTC to local time (-7 or -6 hours).
- **Cost gas for Mountain Time Conversion:** Everytime converting UTC to Mountain Time with DST Rule on-chain cost gas to the contributor.
- **`daysElapsed` Calculation:** Ensures accurate tracking of daily operations based on Mountain Time.
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

The **Token.sol** smart contract combines advanced token minting, dynamic claim limit calculations, contributor management, and streamlined claiming logic. Tokens are minted only on demand when contributors claim them, and the contract dynamically calculates the **Max Daily Claim Limit** based on the **Base Claim Limit** and the **Total Previous Day Claims.**
