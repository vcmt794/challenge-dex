---

# 🏗️ Simple DEX (ETH ↔ BAL) – Hướng dẫn chạy theo từng Checkpoint

Tài liệu này hướng dẫn **từng bước chạy và hoàn thành các checkpoint** của challenge **Simple Decentralized Exchange (DEX)** sử dụng **Scaffold-ETH**.

DEX cho phép:

* Swap **ETH ↔ BAL (ERC20 Balloons)**
* Add / Remove Liquidity
* Áp dụng mô hình **AMM (x * y = k)** với **fee 0.3%**

---

## 📦 Yêu cầu môi trường

Cài sẵn các công cụ sau:

* **Node.js >= v20.18.3**
* **Yarn** (v1 hoặc v2+)
* **Git**

Kiểm tra nhanh:

```bash
node -v
yarn -v
git --version
```

---

## 🚀 Khởi tạo dự án

Tạo project từ template challenge:

```bash
npx create-eth@1.0.2 -e challenge-dex challenge-dex
cd challenge-dex
yarn install
```

---

## ✅ Checkpoint 0 – Environment 📦

### Mục tiêu

* Chạy blockchain local
* Deploy smart contracts
* Chạy frontend

### Các bước

Mở **3 terminal riêng biệt**:

**Terminal 1 – Local blockchain**

```bash
yarn chain
```

**Terminal 2 – Deploy contracts**

```bash
yarn deploy
```

**Terminal 3 – Frontend**

```bash
yarn start
```

Mở trình duyệt:

```
http://localhost:3000
```

> 🔁 Khi sửa contract:
> Dừng `yarn chain` → chạy lại `yarn chain` → `yarn deploy --reset`

---

## ✅ Checkpoint 1 – Contract Structure 🔭

### Mục tiêu

Hiểu cấu trúc project và các contract chính.

### Các contract

| Contract       | Vai trò                                        |
| -------------- | ---------------------------------------------- |
| `Balloons.sol` | ERC20 token (BAL) – mint 1000 BAL cho deployer |
| `DEX.sol`      | AMM DEX – swap, price, liquidity               |

### Việc cần làm

Mở file:

```
packages/hardhat/contracts/DEX.sol
```

* Đọc các function rỗng
* Hiểu mối liên hệ:

  * `init`
  * `price`
  * `ethToToken`
  * `tokenToEth`
  * `deposit`
  * `withdraw`

---

## ✅ Checkpoint 2 – Reserves & Liquidity ⚖️

### Mục tiêu

* Track tổng liquidity
* Track liquidity của từng user
* Initialize pool ETH + BAL

### Việc cần implement

Trong `DEX.sol`:

* Khai báo:

  ```solidity
  uint256 public totalLiquidity;
  mapping(address => uint256) public liquidity;
  ```
* Implement:

  * `init(uint256 tokens)`
  * `getLiquidity(address)`

### Deploy lại

```bash
yarn deploy --reset
```

### Test

* Approve BAL cho DEX
* Call `init()` với:

  * ETH: ví dụ `0.01`
  * BAL: `0.01`

👉 DEX phải có **ETH + BAL** balance.

---

## ✅ Checkpoint 3 – Pricing Function 🤑

### Mục tiêu

* Implement công thức **AMM x * y = k**
* Áp dụng **fee 0.3%**

### Function cần code

```solidity
function price(
  uint256 xInput,
  uint256 xReserves,
  uint256 yReserves
) public pure returns (uint256)
```

### Công thức chuẩn

```text
xInputWithFee = xInput * 997
yOutput = (xInputWithFee * yReserves) / (xReserves * 1000 + xInputWithFee)
```

### Kiến thức cần nắm

* Không có số thập phân trong Solidity
* Fee = `997 / 1000`
* Slippage tăng khi trade lớn

---

## ✅ Checkpoint 4 – Trading 🤝

### Mục tiêu

* Swap ETH → BAL
* Swap BAL → ETH

### Functions cần implement

* `ethToToken()`
* `tokenToEth(uint256 tokenInput)`

### Lưu ý quan trọng

* `ethToToken`:

  * `msg.value` là input
* `tokenToEth`:

  * Phải `approve` BAL trước
* Emit event sau mỗi swap

### Test

* Swap ETH → BAL
* Approve BAL → Swap BAL → ETH
* Kiểm tra reserve thay đổi đúng

---

## ✅ Checkpoint 5 – Liquidity Pool 🌊

### Mục tiêu

* User bất kỳ có thể add liquidity
* Withdraw liquidity theo tỷ lệ

### Functions cần implement

* `deposit()`
* `withdraw(uint256 amount)`

### Logic cốt lõi

* Deposit:

  * ETH gửi vào
  * BAL lấy theo **tỷ lệ pool**
  * Mint LP tokens
* Withdraw:

  * Nhận ETH + BAL theo % liquidity
  * Có thể **lãi từ fee**

### Test

* Deposit nhiều lần
* Swap → tạo fee
* Withdraw → nhận nhiều hơn ban đầu

---

## ✅ Checkpoint 6 – UI & Events 🖼

### Mục tiêu

* UI update realtime
* Chart hiển thị slippage
* Event hiển thị swap / liquidity

### Side Quest (optional)

File:

```
packages/nextjs/app/events/page.tsx
```

* Emit event cho `approve()`
* Hiển thị approve status rõ ràng

---

## ⚠️ Test Contract

Chạy test tự động:

```bash
yarn test
```

🎯 Mục tiêu: **All tests pass (màu xanh)**

---

## 🛰️ Checkpoint 7 – Deploy Testnet (Sepolia)

### Chuẩn bị account

```bash
yarn generate
yarn account
```

### Yêu cầu ETH

* **≥ 0.1 ETH Sepolia** (nếu init 0.1 ETH)
* Faucet: Alchemy / Infura / Chainlink

### Deploy

```bash
yarn deploy --network sepolia
```

⚠️ Nếu lỗi *insufficient funds* → giảm ETH init trong `00_deploy_dex.ts`

---

## 🚢 Checkpoint 8 – Deploy Frontend

### Cấu hình network

File:

```
packages/nextjs/scaffold.config.ts
```

```ts
targetNetwork: chains.sepolia
```

### Deploy Vercel

```bash
yarn vercel
yarn vercel --prod
```

Lưu lại URL frontend public.\
Front end được deploy và submit:
```
https://nextjs-k8eaxm1am-lab01s-projects.vercel.app
```

---

## 📜 Checkpoint 9 – Verify & Submit

### Verify contract

```bash
yarn verify --network sepolia
```

### Submit

* Copy link **Etherscan DEX contract**
* Submit frontend URL lên:

```
https://speedrunethereum.com
```

* url đã được submit:
```
https://sepolia.etherscan.io/address/0x27a144244a5b654416b469CF634E616445619B2E
```
---

## 🎯 Hoàn thành 🎉
