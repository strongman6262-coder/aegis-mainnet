# راهنمای جامع راه‌اندازی و همرسانی سرور محلی (Local Node & P2P Deployment Guide)

این راهنما برای اجرای نود اصلی روی کامپیوتر شخصی شما (به عنوان Seed Node / Validator) و نحوه اتصال نودهای دیگر جهت اطمینان از همرسانی ۱۰۰٪ سالم (Sync) تهیه شده است.

---

## ۱. پیش‌نیازها روی سیستم شما
1. نصب کامپایلر Rust و ابزار Cargo:
   - لینوکس/مک: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
   - ویندوز: دانلود و نصب از [rustup.rs](https://rustup.rs)
2. پورت‌های مورد نیاز در فایروال سیستم یا مودم:
   - پورت **`30303`** (ارتباط P2P و همرسانی نودها)
   - پورت **`8545`** (JSON-RPC برای والت‌ها و صرافی‌ها)
   - پورت **`8080`** (Coinbase Rosetta API)

---

## ۲. کامپایل نهایی باینری نود (Release Build)
در پوشه اصلی پروژه دستور زیر را اجرا کنید:

```bash
cargo build --release -p aegis-node
```
باینری اجرایی در مسیر `./target/release/aegis-node` (یا `aegis-node.exe` در ویندوز) ساخته می‌شود.

---

## ۳. راه‌اندازی کامپیوتر شما به عنوان نود اصلی (Bootnode / Validator)
برای اینکه کامپیوتر شما شروع به تولید بلاک‌ها کند و نقش سرور همرسانی را برای بقیه نودها ایفا کند:

```bash
# لینوکس و مک
./target/release/aegis-node --validator --config config.toml

# ویندوز در PowerShell
.\target\release\aegis-node.exe --validator --config config.toml
```

**خروجی مورد انتظار در ترمینال شما:**
```text
============================================================
    AEGIS PROTOCOL - Modular Privacy Layer-1 Full Node      
============================================================
[Node] Validator Mode Enabled: Keypair generated
[Node] Validator Public Key: 366f705b...
[Node] Validator Address:    0x929145f6...
[Genesis] Network: Aegis Mainnet (ChainID: 714261)
[Genesis] Initial Allocations: 5 accounts
[Node] Initialized Genesis Block: 069c145a...
[Consensus] Starting PoS Block Production Loop...
[Block #1] Mined successfully! Hash: ff347240... | TxCount: 0
[Block #2] Mined successfully! Hash: d49ffa10... | TxCount: 0
[RPC] JSON-RPC 2.0 Engine ready at 0.0.0.0:8545
[P2P] Listening for incoming peers on 0.0.0.0:30303
============================================================
```

---

## ۴. نحوه تست اتصال نود دوم (Sync Verification)
برای اینکه مطمئن شوید مشکل همرسانی فورک مونرو اینجا تکرار نمی‌شود، نود دوم (روی یک لپ‌تاپ دیگر یا در ترمینال دوم همان کامپیوتر با پورت دیگر) با دستور زیر به نود شما متصل می‌شود:

```bash
# اجرای نود پیر و معرفی IP نود اصلی شما به عنوان Bootnode
./target/release/aegis-node --peer "YOUR_IP_ADDRESS:30303"
```

### چک‌لیست جلوگیری از خطای همگام‌سازی (Zero Sync Errors):
1. **تطابق Genesis State Root**: هر دو نود باید در خط `Initialized Genesis Block` مقدار هش یکسان `069c145a...` را چاپ کنند.
2. **بررسی پورت فایروال**: پورت `30303` روی کامپیوتر سرور باید در حالت `Listen 0.0.0.0` باز باشد (نه `127.0.0.1`).
3. **مکانیزم Magic Byte**: هندشیک شبکه با هدر `0x41454749` تایید می‌کند که هر دو نود در حال خواندن زنجیره واقعی هستند و بسته‌های نامعتبر ریجکت نمی‌شوند.
