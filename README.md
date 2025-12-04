# Sai’s Banking & Money System (MTA:SA)

**Join our Discord server to Buy this script** - https://discord.gg/C6yGdAEWEa 

**Buy From Gumroad** - https://saimusick.gumroad.com/l/huiwr

A complete money and banking system for MTA:SA Roleplay / RPG servers, including:

- **Persistent money system** (wallet + bank)
- **Configurable bank location & keybind**
- **Modern CEF bank UI** with:
  - View balances
  - Withdraw to hand
  - Deposit from hand
  - Transfer to other accounts by name
  - Account name + formatted 12‑digit account number
- **Automatic paydays** directly to bank balance

---

## Features

### 1. Money System

- Stores money **per-account** using MTA’s account data.
- Syncs account money with:
  - `setPlayerMoney` (wallet)
  - `elementData`:
    - `money` – current cash in hand
    - `bank` – current bank balance
- Automatically:
  - Saves money and bank on **resource stop**
  - Saves player money/bank on **quit**
  - Restores money/bank on **resource start** for all logged players

**Exported functions (server-side):**

From `money.lua`:

```lua
-- Wallet (hand) money
setMoney(account, amount)
getMoney(account)
giveMoney(account, amount)
takeMoney(account, amount)

-- Bank money
setBank(account, amount)
getBank(account)
giveBank(account, amount)
takeBank(account, amount)
```

You can use these from other resources via `call` / `exports`:

```lua
local bankRes = getResourceFromName("[players]/money")
exports["[players]/money"]:giveBank(account, 5000)
```

There’s also a `/balance` command to quickly print wallet + bank to the player’s chat.

---

### 2. Bank System

- **Bank marker**:
  - Position, interior & dimension configurable in `config.lua`.
  - When you enter, you see a notification:
    - `Press E to access bank` (uses your notification system: `add:notification`).
- **Keybind**:
  - Default interaction key: `E`.
  - Press E near the marker to open the bank UI.
- **Modern CEF UI**:
  - Shows:
    - Cash in hand
    - Bank balance
    - Account name
    - 12‑digit account number (based on player ID, zero‑padded)
  - Actions:
    - **Withdraw:** move from bank → wallet.
    - **Deposit:** move from wallet → bank.
    - **Transfer:** bank → another account by account name.
  - Close via **X button** or leaving the marker.

The UI is responsive and only renders the glass card (transparent background so game world is visible).

---

### 3. Payday System

From `payday.lua`:

- Configurable **interval** (currently 1 hour) using real time.
- Pays a fixed `PAYDAYAMOUNT` into each account’s **bank balance** if:
  - Player is online or
  - Logged in within the last period.
- Logs all payments to server log.
- Uses the same `giveBank` function as the main bank system.

---

## Files Overview

### Core Scripts

- `meta.xml`  \
  Resource manifest & exports.

- `money.lua`  \
  - Wallet & bank logic.\
  - Exports for other resources.\
  - Load/save on resource start/stop & player join/quit.\
  - `/balance` & `/set` (admin/debug) commands.

- `payday.lua`  \
  - Periodic payday logic.\
  - Uses `giveBank` to credit bank balance.

### Bank Logic

- `config.lua`  \
  Shared configuration for bank marker & keybind:

  ```lua
  bankConfig = {
      -- Bank marker position (edit to move)
      position = {
          x = 1545.6,
          y = -1675.6,
          z = 13.6,
          interior = 0,
          dimension = 0,
      },

      -- Marker visuals
      markerType  = "cylinder",
      markerSize  = 1.5,
      markerColor = { r = 0, g = 150, b = 255, a = 150 },

      -- Key used to open the bank near marker
      interactionKey = "e",
  }
  ```

- `bank_server.lua`  \
  - Creates and manages the bank marker.\
  - On marker hit: 
    - Triggers client event to enable UI opening.\
    - Shows `Press E to access bank` notification.\
  - Bank operations: 
    - `handleGetBalance(player)`\
    - `handleWithdraw(player, amount)`\
    - `handleDeposit(player, amount)`\
    - `handleTransfer(player, targetAccountName, amount)`\
  - Validates amounts and target accounts.\
  - Sends updated balances and notifications back to client.

- `bank_client.lua`  \
  - Client-side logic: 
    - Tracks `nearBank` and `bankOpen`.\
    - Listens to `money:bankEnter` / `money:bankLeave`.\
    - Binds `E` to open bank UI when near marker.\
    - Creates/destructs `guiCreateBrowser` full-screen CEF.\
    - Sends bank actions to server on button clicks.\
    - Receives updated balances from server and forwards to JS.

- `bank_events.lua`  \
  - Registers custom events: 
    - `money:bankEnter`\
    - `money:bankLeave`  
    (Required so server can trigger them.)

### HTML / UI

Path: `html/bank/`

- `index.html`  \
  Main bank panel layout.

- `style.css`  \
  Modern glassmorphism styling.

- `app.js`  \
  Data binding and event bridge: 
  - Receives `updateBankInfo(hand, bank, accountName, accountNumber)` from Lua.\
  - Sends withdraw/deposit/transfer/close requests back to Lua using `mta.triggerEvent`.

---

## Installation

1. **Place resource**

   Copy the whole `[players]/money` folder into your server’s `resources` directory.

2. **Ensure dependencies**

   - MTA:SA 1.5+ with CEF/browser enabled.
   - Your notification system must define:
     - `add:notification` client event (used to show hints & messages).

3. **meta.xml**

   Already configured:

   ```xml
   <meta>
       <info author="sai ram kishan" type="script" name="money and payday" version="0.1.0" />

       <script src="config.lua" type="shared" />

       <script src="money.lua" type="server" />
       <script src="payday.lua" type="server" />
       <script src="bank_server.lua" type="server" />

       <script src="bank_events.lua" type="client" />
       <script src="bank_client.lua" type="client" />

       <file src="html/bank/index.html" type="client" />
       <file src="html/bank/style.css" type="client" />
       <file src="html/bank/app.js" type="client" />
   </meta>
   ```

4. **Start the resource**

   In console or ACL-authorized chat:

   ```
   refresh
   start money
   ```

   Or via admin panel.

---

## Configuration

### Bank Location & Marker

Edit `config.lua`:

- **Position**:

  ```lua
  bankConfig.position = {
      x = 1545.6,
      y = -1675.6,
      z = 13.6,
      interior = 0,
      dimension = 0,
  }
  ```

  Change to any world coordinates / interior / dimension.

- **Keybind**:

  ```lua
  bankConfig.interactionKey = "e"
  ```

  You can change `"e"` to any valid MTA key string (`"f"`, `"enter"`, etc.).

- **Marker style**:

  - `markerType` – any valid marker type.
  - `markerSize` – radius/size.
  - `markerColor` – RGBA.

### Payday Settings

In `payday.lua`:

- **Interval** (currently every hour):

  ```lua
  local timeInterval = 1000 * 60 * 60 -- 1 hour in ms
  local oneDay = 60 * 60              -- 1 hour in seconds
  ```

- **Amount**:

  ```lua
  local PAYDAYAMOUNT = 1000
  ```

Adjust to fit your economy.

---

## Usage for Players

- Walk into the **bank marker** (e.g., configured outside LSPD).
- You see a hint: **Press E to access bank**.
- Press **E**:
  - Banking panel opens.
  - You see:
    - Cash in Hand
    - Money in Bank
    - Account name
    - 12-digit account number.
- Use:
  - **Withdraw**:
    - Type amount → click **Withdraw**.
  - **Deposit**:
    - Type amount → click **Deposit**.
  - **Transfer**:
    - Enter target account name and amount → click **Transfer**.
- Close via:
  - **X button** in top-right of panel, or
  - Walking out of the marker.

---

