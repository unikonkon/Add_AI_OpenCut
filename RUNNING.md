# คู่มือการติดตั้งและรัน OpenCut

เอกสารนี้อ้างอิงจากโครงสร้างและไฟล์ตั้งค่าปัจจุบันของ repository นี้ โดยโปรเจกต์อยู่ระหว่างการเขียนใหม่และแบ่งเป็น 3 แอปที่รันแยกกัน:

| แอป | เทคโนโลยีหลัก | คำสั่งรัน | ตำแหน่งเริ่มต้น |
| --- | --- | --- | --- |
| Web | React 19, TanStack Start, Vite, Cloudflare | `moon run web:dev` | <http://localhost:5173> |
| API | Elysia, Cloudflare Workers | `moon run api:dev` | <http://localhost:8787> |
| Desktop | Rust, GPUI | `moon run desktop:dev` | เปิดเป็น native window |

> Web, API และ Desktop ยังไม่ได้เชื่อมต่อกันในโค้ดปัจจุบัน จึงเลือกเปิดเฉพาะแอปที่ต้องการพัฒนาได้

## 1. สิ่งที่ต้องติดตั้ง

ใช้ `proto` เพื่อให้เวอร์ชันเครื่องมือตรงกับไฟล์ `.prototools` ของโปรเจกต์:

- Moon `2.3.3`
- Bun `1.3.11`
- Rust `1.97.0`

ติดตั้ง proto บน Linux, macOS หรือ WSL:

```sh
bash <(curl -fsSL https://moonrepo.dev/install/proto.sh)
```

บน Windows PowerShell:

```powershell
irm https://moonrepo.dev/install/proto.ps1 | iex
```

หาก PowerShell ไม่อนุญาตให้รัน shim ให้ตั้งค่าสำหรับ user ปัจจุบันหนึ่งครั้ง:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

## 2. เตรียมโปรเจกต์ครั้งแรก

รันจาก root ของ repository:

```sh
proto use
bun install --cwd apps/web
bun install --cwd apps/api
```

เหตุผลที่ต้องติดตั้ง dependency สองครั้งคือ repository นี้ไม่มี JavaScript workspace ที่ root; `apps/web` และ `apps/api` มี `package.json` ของตัวเอง ส่วน Desktop ใช้ Cargo และจะดาวน์โหลด dependency ตอนสั่ง build/run ครั้งแรก

ตรวจว่าเครื่องมือพร้อมใช้งาน:

```sh
moon --version
bun --version
rustc --version
```

## 3. รันแต่ละแอป

คำสั่งทั้งหมดด้านล่างรันจาก root ของ repository

### Web

```sh
moon run web:dev
```

เปิด <http://localhost:5173> ใน browser เส้นทางที่มีในปัจจุบันคือ:

- `/` — หน้าเริ่มต้น
- `/editor` — หน้า Editor ที่ยังเป็น placeholder

คำสั่งอื่น:

```sh
moon run web:build      # production build ไปที่ apps/web/dist
moon run web:test       # รัน Vitest แบบครั้งเดียว
```

### API

```sh
moon run api:dev
```

Wrangler จะแสดง URL ที่ใช้งานจริงใน terminal; ค่าเริ่มต้นโดยทั่วไปคือ <http://localhost:8787> ทดสอบ endpoint ปัจจุบันได้ด้วย:

```sh
curl http://localhost:8787/
curl http://localhost:8787/health
curl -X POST http://localhost:8787/echo \
  -H 'Content-Type: application/json' \
  -d '{"message":"hello"}'
```

ผลลัพธ์ที่คาดหวัง:

- `GET /` คืน `{"status":"ok"}`
- `GET /health` คืนสถานะพร้อม timestamp
- `POST /echo` คืน body เดิม และกำหนดให้ `message` เป็น string

ตรวจ production bundle โดยไม่ deploy:

```sh
moon run api:build
```

### Desktop

เตรียม dependency ของระบบก่อนตาม platform:

- macOS: ติดตั้ง Xcode Command Line Tools ด้วย `xcode-select --install`
- Windows: ไม่ต้องติดตั้ง dependency เพิ่มสำหรับ Win32/DirectWrite แต่ต้องมี toolchain ที่ `proto use` ติดตั้งไว้
- Debian/Ubuntu: ต้องมี C toolchain, CMake, Vulkan driver และแพ็กเกจ `libvulkan1`, `libwayland-dev`, `libx11-xcb-dev`, `libxkbcommon-x11-dev`, `libfontconfig-dev`
- WSL2/WSLg: แอปจะเลือก XWayland อัตโนมัติเมื่อพบทั้ง X11 และ Wayland เนื่องจาก GPUI `0.2.2` ไม่รองรับ Wayland protocol version ที่ WSLg ประกาศ

รันแอป:

```sh
moon run desktop:dev
```

ครั้งแรกอาจใช้เวลานานเพราะ Cargo ต้องดาวน์โหลดและ compile GPUI จาก source

คำสั่งตรวจและ build:

```sh
moon run desktop:check
moon run desktop:build
```

ไฟล์ release จะอยู่ที่ `target/release/opencut-desktop` (บน Windows เป็น `opencut-desktop.exe`)

## 4. รันหลายแอปพร้อมกัน

เปิดคนละ terminal จาก root ของ repository:

Terminal 1:

```sh
moon run web:dev
```

Terminal 2:

```sh
moon run api:dev
```

Terminal 3 (เมื่อต้องการ Desktop):

```sh
moon run desktop:dev
```

แต่ละ process เป็น dev server/native app ที่ทำงานค้างอยู่ กด `Ctrl+C` ใน terminal นั้นเพื่อหยุด

## 5. ตัวแปรแวดล้อม

การรัน Web, API และ Desktop ใน local ปัจจุบันไม่ต้องใช้ environment variable

ไฟล์ `.env.example` มีเฉพาะ:

```dotenv
R2_BUCKET=opencut-assets
```

ค่านี้ใช้กับ task อัปโหลดไฟล์ SVG ใน `brand/marks` ขึ้น Cloudflare R2 เท่านั้น หากต้องใช้งาน:

```sh
cp .env.example .env.local
moon run :upload-logos
```

ก่อนรันต้อง login Wrangler และมีสิทธิ์เขียน bucket ที่กำหนด คำสั่งนี้อัปโหลดไปยัง remote R2 จริง จึงไม่ใช่ขั้นตอนที่จำเป็นสำหรับ local development

## 6. Build, test และ CI

รัน task ที่เหมาะกับแต่ละแอป:

```sh
moon run web:build
moon run web:test
moon run api:build
moon run desktop:check
moon run desktop:build
```

หากต้องการใช้โหมด CI ให้ระบุเฉพาะชนิด task ที่ปลอดภัยอย่างชัดเจน:

```sh
moon ci :build :test :check
```

Moon จะเลือกเฉพาะ project ที่ได้รับผลกระทบจากไฟล์ที่เปลี่ยน ดูรายละเอียดได้จาก [Moon CI documentation](https://moonrepo.dev/docs/commands/ci)

> ข้อควรระวัง: workflow ปัจจุบันใน `.github/workflows/bun-ci.yml` ใช้ `moon ci` โดยไม่ระบุ target ขณะที่ task `deploy` ยังไม่ได้ตั้ง `runInCI: false` ตามค่า config ที่อยู่ใน repository จึงควรตรวจ action graph และ Cloudflare credentials ก่อนใช้ workflow นี้บน branch ที่มีการเปลี่ยน Web/API การรันในเครื่องควรใช้คำสั่งแบบระบุ target ด้านบน

ปัจจุบันมี task test เฉพาะ Web และยังไม่พบไฟล์ test ใน source tree การรัน `web:test` จึงอาจรายงานว่าไม่พบ test จนกว่าจะมีการเพิ่ม test หรือปรับค่า Vitest

## 7. Deploy (เมื่อมีสิทธิ์ Cloudflare)

การ deploy ไม่จำเป็นสำหรับการพัฒนาในเครื่อง และจะเปลี่ยนแปลง Cloudflare account ที่ Wrangler login อยู่

```sh
moon run web:deploy
moon run api:deploy
```

- Web ใช้ Worker ชื่อ `opencut-web` และกำหนด custom domain `new.opencut.app`
- API ใช้ Worker ชื่อ `opencut-api`

ตรวจ account/target ให้ถูกต้องก่อนสั่ง deploy ทุกครั้ง

## 8. โครงสร้างโปรเจกต์ที่เกี่ยวกับการรัน

```text
.
├── .prototools               # pin เวอร์ชัน Moon, Bun และ Rust
├── .moon/                    # JavaScript/Rust toolchain และ workspace config
├── moon.yml                  # task ระดับ root เช่น upload-logos
├── Cargo.toml                # Rust workspace
├── apps/
│   ├── web/
│   │   ├── package.json      # dev/build/test/deploy scripts
│   │   ├── moon.yml          # Moon tasks ของ Web
│   │   ├── vite.config.ts    # Vite + React + TanStack + Cloudflare
│   │   ├── wrangler.jsonc    # Cloudflare config ของ Web
│   │   └── src/              # routes, UI components และ styles
│   ├── api/
│   │   ├── package.json      # dev/build/deploy scripts
│   │   ├── moon.yml          # Moon tasks ของ API
│   │   ├── wrangler.jsonc    # Cloudflare Worker config
│   │   └── src/index.ts      # Elysia app และ endpoints
│   └── desktop/
│       ├── Cargo.toml        # Rust package ของ Desktop
│       ├── moon.yml          # dev/check/build tasks
│       └── src/              # GPUI window, shell, panels และ components
├── brand/marks/              # SVG assets สำหรับอัปโหลด R2
└── .github/workflows/        # CI ที่รัน moon ci บน Linux/Windows/macOS
```

## 9. แก้ปัญหาเบื้องต้น

### หา `proto`, `moon` หรือ `bun` ไม่เจอ

เปิด terminal ใหม่หลังติดตั้ง proto หรือเพิ่มตำแหน่ง shim ที่ installer แจ้งลงใน `PATH` จากนั้นรัน `proto use` ที่ root อีกครั้ง

### Web/API แจ้งว่าหา package ไม่เจอ

ติดตั้ง dependency ของแอปนั้นใหม่:

```sh
bun install --cwd apps/web
bun install --cwd apps/api
```

### Port ถูกใช้งานอยู่

Web fix port ไว้ที่ `5173`; ให้หยุด process ที่ใช้ port นี้ก่อน ส่วน API ให้ตรวจ URL/port ที่ Wrangler แสดงใน terminal

### Desktop compile ไม่ผ่านบน Linux

ตรวจว่า system packages, Vulkan driver, C compiler และ CMake ติดตั้งครบ แล้วลอง `moon run desktop:check` เพื่อดู error โดยไม่ต้องเปิดหน้าต่าง

### ต้องการข้าม Moon เพื่อ debug

สามารถรันคำสั่งต้นทางจากโฟลเดอร์ของแต่ละแอปได้:

```sh
bun run --cwd apps/web dev
bun run --cwd apps/api dev
cargo run -p opencut-desktop
```

อย่างไรก็ตาม แนะนำให้ใช้ Moon เป็นหลักเพื่อให้พฤติกรรมตรงกับ task config และ CI ของ repository
