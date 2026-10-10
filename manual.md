# คู่มือ Aviation Monitor: สร้างโปรแกรม ออกเวอร์ชัน และอัปเดต

คู่มือนี้อธิบายตั้งแต่เตรียมเครื่อง สร้างโปรแกรม ติดตั้ง ไปจนถึงออกอัปเดตผ่าน GitHub
ค่าที่ใช้ในคู่มือนี้

| รายการ | ค่า |
|---|---|
| GitHub repo | `kanutsanun-b/aviation-monitor` (ต้องเป็น **Public**) |
| ลิงก์ `version.json` (ฝังในโปรแกรมแล้ว) | `https://raw.githubusercontent.com/kanutsanun-b/aviation-monitor/main/release/version.json` |
| โฟลเดอร์ทำงานในเครื่อง | `C:\AviationMonitor` (เปลี่ยนได้) |

---

## 0. ภาพรวม: ไฟล์อะไรทำอะไร

| ไฟล์ | หน้าที่ |
|---|---|
| `mornitor_data.py` | ตัวโปรแกรมหลัก (หน้าต่าง GUI, ปุ่มรันสคริปต์, ปุ่ม Update) |
| `01_taf_decode.py`, `02_flight_route.py`, `03_tc_warning_graphic.py`, `05_daily_forcast.py`, `06_satmet_radar.py`, `07_sigmet.py` | สคริปต์ย่อยแต่ละงาน |
| `app_version.py` | เก็บเลขเวอร์ชัน: `APP_VERSION` (ตัวโปรแกรม) และ `SCRIPTS_VERSION` (ชุดสคริปต์) |
| `build.bat` | สร้าง `dist\AviationMonitor.exe` ด้วย PyInstaller |
| `installer.iss` | สคริปต์ของ Inno Setup สำหรับสร้างตัวติดตั้ง |
| `make_release.py` | สร้างตัวติดตั้ง / zip สคริปต์ และ `version.json` ลงโฟลเดอร์ `release` |
| `config.json` | ตั้งค่า (API key ของ CheckWX ฯลฯ) วางไว้ข้างตัว .exe |

**ระบบอัปเดตมี 2 ระดับ**

1. **อัปเดตเฉพาะสคริปต์**: ไฟล์ zip เล็ก (ราว 20 KB) ใช้ได้ทันทีโดยไม่ต้องรีสตาร์ท ใช้บ่อยที่สุด
2. **อัปเดตทั้งโปรแกรม**: ตัวติดตั้งเต็ม (ไฟล์ใหญ่) ใช้เมื่อเปลี่ยนตัว launcher หรือเพิ่มไลบรารีใหม่

---

## 1. เตรียมเครื่อง (ทำครั้งเดียว)

1. ติดตั้ง **Python 3.11 หรือ 3.12** จาก python.org
   ตอนติดตั้งต้องติ๊ก **"Add python.exe to PATH"**
2. ติดตั้ง **Inno Setup 6** (ฟรี) จาก jrsoftware.org
3. มีบัญชี **GitHub** และเปิด 2FA (ขอแนะนำอย่างยิ่ง ดูหัวข้อ 9)
4. สร้างโฟลเดอร์ทำงาน เช่น `C:\AviationMonitor` แล้วรวมไฟล์ทั้งหมดในตารางหัวข้อ 0 ไว้ในโฟลเดอร์เดียวกัน

> ถ้ามีไฟล์เวอร์ชันเก่าปนอยู่ ให้ทับด้วยไฟล์ล่าสุดเสมอ

---

## 2. ตั้งค่า GitHub (ทำครั้งเดียว)

1. สร้าง repository ชื่อ `aviation-monitor` ตั้งเป็น **Public**
   (ถ้าเป็น Private โปรแกรมจะเข้าถึง `version.json` ไม่ได้)
2. ยังไม่ต้องอัปโหลดอะไรตอนนี้

---

## 3. ตรวจค่าในโปรแกรมก่อน build (ทำครั้งเดียว)

1. เปิด `mornitor_data.py` ตรวจว่าบรรทัด `UPDATE_URL` เป็นลิงก์ของคุณ

   ```python
   UPDATE_URL = "https://raw.githubusercontent.com/kanutsanun-b/aviation-monitor/main/release/version.json"
   ```

2. เปิด `app_version.py` ตรวจเลขเวอร์ชัน (เริ่มต้น)

   ```python
   APP_VERSION = "1.1.0"
   SCRIPTS_VERSION = 1
   ```

3. เปิด `config.json` ใส่ `checkwx_api_key` ของคุณ
   (ถ้าจะแจกให้คนอื่น ให้ปล่อยเป็นข้อความตัวอย่าง แล้วให้ผู้ใช้ใส่เองตอนติดตั้ง)

---

## 4. ออกเวอร์ชันแรก (1.1.0)

### 4.1 สร้างตัวโปรแกรม

1. ดับเบิลคลิก `build.bat` รอจนขึ้น `SUCCESS` จะได้ `dist\AviationMonitor.exe`
   ถ้าไม่สำเร็จ ให้ดูท้ายไฟล์ `build_log.txt`
2. วาง `config.json` ไว้ข้าง `dist\AviationMonitor.exe` แล้วเปิดทดลองกดทีละปุ่ม

### 4.2 สร้างตัวติดตั้งและ version.json

เปิด Command Prompt ในโฟลเดอร์ทำงาน แล้วรัน

```
python make_release.py app https://github.com/kanutsanun-b/aviation-monitor/releases/download/v1.1.0 "ปล่อยใช้งานครั้งแรก"
```

ผลลัพธ์ในโฟลเดอร์ `release`

- `AviationMonitor_Setup_1.1.0.exe` (ตัวติดตั้ง)
- `version.json`

ตรวจด้วยคำสั่ง `dir release`

> ข้อความในเครื่องหมายคำพูดเป็นหมายเหตุที่ผู้ใช้เห็นตอนอัปเดต ใส่อะไรก็ได้
> แต่ห้ามมีเครื่องหมายคำพูดคู่ `"` อยู่ในข้อความ
> แท็ก `v1.1.0` ในลิงก์ต้องตรงกับแท็กที่จะสร้างในขั้นตอน 4.3

### 4.3 อัปโหลดตัวติดตั้งขึ้น GitHub Releases

1. เปิด `https://github.com/kanutsanun-b/aviation-monitor/releases/new`
2. ช่อง **Tag** พิมพ์ `v1.1.0` แล้วเลือก Create new tag
3. ช่อง **Release title** ใส่ `v1.1.0`
4. เลื่อนลงไป **ใต้ช่องคำอธิบาย** ที่กรอบเส้นประ **"Attach binaries by dropping them here or selecting them"**
   ลากไฟล์ `AviationMonitor_Setup_1.1.0.exe` ไปวางในกรอบนี้
   *(อย่าวางในช่อง "Describe this release" เพราะจะขึ้นว่า "We don't support that file type")*
5. รอให้ชื่อไฟล์ขึ้นในรายการและแถบอัปโหลดเต็ม แล้วกด **Publish release**

### 4.4 อัปโหลด version.json เข้า repo

1. ที่หน้า repo กด **Add file → Create new file**
2. ช่องชื่อไฟล์พิมพ์ `release/version.json` (พิมพ์ `release/` นำหน้าเพื่อสร้างโฟลเดอร์)
3. เปิดไฟล์ `release\version.json` ในเครื่อง คัดลอกเนื้อหาทั้งหมดมาวาง
4. กด **Commit changes**

### 4.5 ตรวจลิงก์

เปิดลิงก์นี้ในเบราว์เซอร์ ต้องเห็นข้อความ JSON ล้วนๆ

```
https://raw.githubusercontent.com/kanutsanun-b/aviation-monitor/main/release/version.json
```

ถ้าเห็น 404 แปลว่าไฟล์ยังไม่อยู่ที่ `release/version.json` ถ้าเนื้อหาเก่า ให้รอสักครู่ (GitHub แคชไว้ไม่กี่นาที)

---

## 5. ติดตั้งและทดสอบระบบอัปเดต

1. รัน `AviationMonitor_Setup_1.1.0.exe` (ไม่ต้องใช้สิทธิ์ Admin)
2. ตอนท้ายการติดตั้งจะเปิด Notepad ให้แก้ `config.json` ใส่ API key ของ CheckWX
3. เปิดโปรแกรมจากไอคอนบน Desktop แล้วกดปุ่ม **Update** (แถวล่าง ข้างปุ่ม Clear Log)
   ควรขึ้นว่า **"เป็นเวอร์ชันล่าสุดแล้ว (โปรแกรม v1.1.0, สคริปต์ v1)"**
   นั่นแปลว่าอ่าน `version.json` ได้ถูกต้อง
4. แจกไฟล์ติดตั้งให้ผู้ใช้ (ส่งไฟล์ตรงๆ หรือให้โหลดจากหน้า Release)

> หาโฟลเดอร์ติดตั้ง: คลิกขวาที่ไอคอนบน Desktop → **Open file location**
> `config.json` และโฟลเดอร์ `output` อยู่ที่นั่น

---

## 6. ภายหลัง: อัปเดตเฉพาะสคริปต์ (ใช้บ่อย ไฟล์เล็ก)

ใช้เมื่อแก้โค้ดในสคริปต์ 01–07 และไม่ได้เพิ่มไลบรารีใหม่

1. แก้สคริปต์ที่ต้องการ
2. เปิด `app_version.py` เพิ่ม `SCRIPTS_VERSION` ขึ้น 1 (เช่น `2`)
3. รัน

   ```
   python make_release.py scripts https://raw.githubusercontent.com/kanutsanun-b/aviation-monitor/main/release "แก้การดาวน์โหลดภาพดาวเทียม"
   ```

   จะได้ `release\scripts_2.zip` และ `release\version.json` (ฉบับรวมทั้งส่วน `app` และ `scripts`)
4. ที่ GitHub เข้าโฟลเดอร์ `release` ของ repo แล้วกด **Add file → Upload files**
   อัปโหลด `scripts_2.zip` และ `version.json` (ชื่อซ้ำจะทับไฟล์เดิม) แล้ว **Commit changes**
5. ผู้ใช้กดปุ่ม **Update** หรือรอตอนเปิดโปรแกรมครั้งถัดไป จะมีหน้าต่างถามยืนยัน
   ใช้ได้ทันที ไม่ต้องรีสตาร์ท

---

## 7. ภายหลัง: อัปเดตทั้งโปรแกรม (ใช้เมื่อเปลี่ยน launcher หรือเพิ่มไลบรารี)

1. เปิด `app_version.py` เพิ่ม `APP_VERSION` เช่น `"1.2.0"`
   (ถ้าสคริปต์เปลี่ยนด้วย ให้เพิ่ม `SCRIPTS_VERSION` ด้วย)
2. รัน `build.bat`
3. รัน

   ```
   python make_release.py app https://github.com/kanutsanun-b/aviation-monitor/releases/download/v1.2.0 "สรุปสิ่งที่แก้"
   ```

4. สร้าง Release ใหม่ แท็ก `v1.2.0` แนบ `release\AviationMonitor_Setup_1.2.0.exe` แล้ว **Publish release**
5. **หลังจาก** Publish เสร็จ ค่อยอัปโหลด `version.json` ใหม่เข้า `release/` ใน repo
   (ลำดับนี้สำคัญ ไม่งั้นผู้ใช้จะเจอลิงก์ที่ยังไม่มีไฟล์)
6. ผู้ใช้กดปุ่ม **Update** → ยืนยัน → โปรแกรมดาวน์โหลดตัวติดตั้ง ตรวจ SHA-256 ติดตั้งทับ
   แล้วเปิดโปรแกรมใหม่ให้เอง (`config.json` และ `output` ไม่ถูกแตะ)

---

## 8. แก้ปัญหาที่พบบ่อย

| อาการ | สาเหตุ / วิธีแก้ |
|---|---|
| `build.bat` ไม่ได้ `dist\AviationMonitor.exe` | ดู `build_log.txt` ท้ายไฟล์ ส่วนมากคือไม่มี Python ใน PATH, ไฟล์ไม่ครบโฟลเดอร์, หรือ Python เวอร์ชันใหม่เกินไป (ใช้ 3.11/3.12) |
| กดสคริปต์แล้วขึ้น `ModuleNotFoundError` | PyInstaller หาไลบรารีไม่เจอ ให้เพิ่มชื่อโมดูลใน `build.bat` (`--hidden-import ...`) และในบล็อก "Bundle hints" ของ `mornitor_data.py` แล้ว build ใหม่ |
| TAF Decode ขึ้นว่าไม่พบ Configuration file | ยังไม่ได้วาง `config.json` ข้างตัว .exe |
| ปุ่ม Sigmet ผิดพลาด | เครื่องปลายทางต้องติดตั้ง Google Chrome |
| ปุ่ม Update ขึ้น "ยังไม่ได้ตั้งค่า update_url" | `UPDATE_URL` ในโค้ดว่าง และ `config.json` ไม่มี `update_url` ให้ตั้งค่าในโค้ดแล้ว build ใหม่ หรือเพิ่ม `"update_url": "<ลิงก์ version.json>"` ใน `config.json` |
| ปุ่ม Update ขึ้น "ตรวจสอบอัปเดตไม่สำเร็จ" + 404 | ยังไม่ได้อัปโหลด `release/version.json` หรือ repo ไม่ใช่ Public หรือพิมพ์ชื่อ/ตัวพิมพ์เล็ก-ใหญ่ในลิงก์ผิด |
| อัปโหลด .exe ใน Release แล้วขึ้น "We don't support that file type" | ไฟล์ถูกวางในช่องคำอธิบาย ให้วางในกรอบ "Attach binaries..." ที่อยู่ใต้ช่องนั้น |
| อัปเดตแล้วขึ้น "SHA-256 ไม่ตรง" | ไฟล์ที่อัปโหลดไม่ใช่ไฟล์เดียวกับที่ `make_release.py` คำนวณไว้ ให้รัน `make_release.py` ใหม่ อัปโหลดไฟล์ใหม่และ `version.json` ใหม่ให้ตรงกัน |
| อัปเดตสคริปต์ขึ้นว่าต้องใช้โปรแกรมเวอร์ชันใหม่กว่า | สคริปต์ชุดนั้นระบุ `min_app_version` สูงกว่าที่ติดตั้ง ต้องออกอัปเดตทั้งโปรแกรม (หัวข้อ 7) ก่อน |
| สคริปต์ที่อัปเดตแล้วมีปัญหา | ลบโฟลเดอร์ `scripts` ข้างตัว .exe ในโฟลเดอร์ติดตั้ง จะกลับไปใช้ชุดที่ฝังมากับโปรแกรม |
| Windows ขึ้นหน้าเตือน SmartScreen | ไฟล์ไม่ได้ลงลายเซ็นดิจิทัล กด More info → Run anyway |

---

## 9. ข้อควรระวังและความปลอดภัย

- **เก็บโฟลเดอร์ `release` ในเครื่องไว้เสมอ** `make_release.py` รวมข้อมูลเดิมใน `version.json` ถ้าลบโฟลเดอร์ ส่วนที่ไม่ได้ออกใหม่จะหายจากไฟล์
- **ห้าม commit ตัวติดตั้ง .exe เข้า repo** (GitHub ปฏิเสธไฟล์เกิน 100 MB) แนบเป็น asset ใน Release เท่านั้น ส่วน `version.json` และ `scripts_N.zip` commit เข้า repo ได้
- โปรแกรมโหลดไฟล์อัปเดตจากลิงก์ `https://` เท่านั้น และตรวจ SHA-256 ก่อนใช้ทุกครั้ง
  สคริปต์ที่อัปเดตจะถูกตรวจชื่อไฟล์และ syntax ก่อนนำมาแทนของเดิม
- **ความเสี่ยงที่ต้องรู้:** สคริปต์ที่อัปเดตคือโค้ด Python ที่รันบนเครื่องผู้ใช้
  ใครที่เขียนไฟล์ใน repo ได้ ก็ส่งโค้ดเข้าเครื่องผู้ใช้ทุกคนได้ SHA-256 ป้องกันไฟล์เสียหายหรือถูกสลับระหว่างทาง แต่ไม่ใช่ลายเซ็นดิจิทัล
  จึงควรเปิด 2FA ให้บัญชี GitHub และไม่ให้ผู้อื่นมีสิทธิ์เขียน repo โดยไม่จำเป็น
- เครื่องผู้ใช้ต้องต่ออินเทอร์เน็ต และเข้าเว็บต้นทาง (TMD, CheckWX, JTWC, aviationweather.gov, GitHub) ได้
- ไฟล์ที่ push ขึ้น `raw.githubusercontent.com` อาจใช้เวลาไม่กี่นาทีกว่าจะเห็นผล

---

## 10. สรุปคำสั่งที่ใช้บ่อย

```
:: สร้างโปรแกรม
build.bat

:: ออกอัปเดตเฉพาะสคริปต์ (หลังเพิ่ม SCRIPTS_VERSION)
python make_release.py scripts https://raw.githubusercontent.com/kanutsanun-b/aviation-monitor/main/release "สรุปที่แก้"

:: ออกอัปเดตทั้งโปรแกรม (หลังเพิ่ม APP_VERSION และรัน build.bat)
python make_release.py app https://github.com/kanutsanun-b/aviation-monitor/releases/download/v<เวอร์ชัน> "สรุปที่แก้"

:: ถ้าคอมไพล์ตัวติดตั้งด้วย Inno Setup เองแล้ว
python make_release.py app <base-url> "สรุป" --skip-compile
```
