# 🏦 Bank Transaction System (Backend Ledger)

A production-style **banking backend** built on a **double-entry accounting ledger**. Instead of storing a mutable `balance` field on each account, every rupee that moves is recorded as two **immutable** ledger entries (a `DEBIT` and a `CREDIT`), and balances are *derived* on demand. This guarantees that money is never created or destroyed — the books always balance.

Built with **Node.js, Express 5, and MongoDB (Mongoose)**.

---

## ✨ Features

- **JWT Authentication** — Register, login, and logout with HTTP-only cookie + Bearer token support. Passwords are hashed with `bcryptjs`.
- **Token Blacklisting** — Logged-out tokens are blacklisted and auto-expire after 3 days using a MongoDB TTL index.
- **Double-Entry Ledger** — Every transaction writes one `DEBIT` and one `CREDIT` entry. Ledger entries are **immutable** — updates and deletes are blocked at the schema level.
- **Derived Balances** — Account balance is computed from ledger entries via a MongoDB aggregation (`totalCredit - totalDebit`), never stored directly.
- **Idempotent Transfers** — Each transaction requires a unique `idempotencyKey`, so retrying a request never double-charges a user.
- **ACID Transactions** — Transfers run inside a MongoDB session/transaction, so a debit and credit either both commit or both roll back.
- **Account State Machine** — Accounts can be `ACTIVE`, `FROZEN`, or `CLOSED`; only `ACTIVE` accounts can transact.
- **System / Funding Route** — A privileged "system user" route to inject initial funds into the ledger.
- **Email Notifications** — Welcome and transaction emails sent via `nodemailer` (Gmail OAuth2).

---

## 🛠️ Tech Stack

| Layer        | Technology                          |
| ------------ | ----------------------------------- |
| Runtime      | Node.js                             |
| Framework    | Express 5                           |
| Database     | MongoDB + Mongoose                  |
| Auth         | JSON Web Tokens (`jsonwebtoken`)    |
| Password     | `bcryptjs`                          |
| Email        | `nodemailer` (Gmail OAuth2)         |
| Config       | `dotenv`                            |
| Cookies      | `cookie-parser`                     |

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (v18+ recommended)
- A MongoDB instance — **a replica set is required** for multi-document transactions (e.g. MongoDB Atlas, or a local replica set)
- A Gmail account with OAuth2 credentials (for email notifications)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/Deepanshuubnare/Bank_Transaction_System.git
cd Bank_Transaction_System

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env
# then open .env and fill in your own values

# 4. Run the server
npm run dev      # development (nodemon, auto-reload)
# or
npm start        # production
```

The server starts on **http://localhost:3000**.

---

## 🔄 The Transfer Flow

Every transfer follows a deterministic, fail-safe sequence:

1. **Validate request** — all required fields present, accounts exist
2. **Validate idempotency key** — short-circuit if this request was already processed
3. **Check account status** — both accounts must be `ACTIVE`
4. **Derive sender balance** from the ledger and check for sufficient funds
5. **Create transaction** record with status `PENDING`
6. **Write the `DEBIT`** ledger entry (sender)
7. **Write the `CREDIT`** ledger entry (receiver)
8. **Mark transaction `COMPLETED`**
9. **Commit the MongoDB session** (atomic — all or nothing)
10. **Send an email notification**

If any step inside the session fails, the entire transaction is rolled back, leaving the books untouched.

---

## 📝 Notes

- Transfers currently include a **15-second simulated processing delay** between the debit and credit writes (in `transaction.controller.js`). This is useful for demonstrating the `PENDING` state and idempotency behavior, and can be removed for production use.
- Multi-document transactions require MongoDB to be running as a **replica set**; they will fail on a standalone `mongod`.

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to open an issue or submit a pull request.

---


## 👤 Author

**Deepanshu Ubnare**  
GitHub: [@Deepanshuubnare](https://github.com/Deepanshuubnare)
