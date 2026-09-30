# เชื่อม GitHub PR กับ Linear ทีม BOO

Flow หลักคือ `feature → dev → main`: review และ QA ที่ `dev` แล้วแจ้ง Discord เมื่อ merge เข้า `main`
workflow Discord ไม่เปลี่ยนสถานะ Linear และไม่ยืนยันว่า deploy สำเร็จ

Workflow `Linear status sync` ใช้กับ repository `HectorRussia/beastscribe`, workspace
[`foro`](https://linear.app/foro/team/BOO/active) และทีม `BOO` โดยอ่านรหัสงานจากชื่อ
branch ที่คัดลอกด้วย **Copy git branch name** ใน Linear

## พฤติกรรม

| เหตุการณ์ | สถานะก่อนหน้า | ผลลัพธ์ |
| --- | --- | --- |
| สร้าง branch หรือ push ก่อนมี PR | ทุกสถานะ | ไม่เปลี่ยน |
| เปิด Draft PR | ทุกสถานะ | ไม่เปลี่ยน |
| เปิด/เปิดใหม่ PR ปกติเข้า `dev` หรือกด Ready for review | `In Progress` | `In Review` |
| Push เพิ่มหรือแก้ไข PR ที่พร้อมตรวจและเข้า `dev` | `In Progress` | `In Review` |
| Merge PR เข้า `dev` | `In Progress` หรือ `In Review` | `QA` |
| ปิด PR โดยไม่ merge หรือ PR เข้า branch อื่น | ทุกสถานะ | ไม่เปลี่ยน |
| เหตุการณ์ใด ๆ | `your task`, `reject`, `QA`, `Done`, `Canceled`, `Todo`, `Backlog` | ไม่เปลี่ยน |

- ผู้ใช้ลากงานเข้า `your task`, `In Progress`, `reject` เอง และลากออกจาก `QA` เมื่อทดสอบเสร็จ
- หลังแก้งานที่ถูก reject ให้ลากกลับ `In Progress` แล้ว push เพิ่มบน PR ที่พร้อมตรวจ
  หรือเปิด/เปิดใหม่ PR; workflow จะย้ายเข้า `In Review` เมื่อเกิด event ถัดไป
- การลากสถานะใน Linear อย่างเดียวไม่เรียก workflow และไม่มีการตั้งค่า webhook ฝั่ง Linear
- เปลี่ยน PR กลับเป็น Draft หรือปิด PR ไม่ย้ายงานย้อนหลัง หากต้องการเปลี่ยนสถานะให้ลากเอง
- ใช้หนึ่งงานต่อหนึ่ง branch/PR หากมีหลาย PR ของงานเดียวกัน PR แรกที่ merge สามารถย้ายงานเข้า `QA`
  ได้ทันที รุ่นนี้ไม่รอให้ทุก PR merge ครบ

## ตั้งค่าครั้งแรก

1. ใน Linear ตรวจว่าทีม `BOO` ใน workspace `foro` มีสถานะชื่อ **`In Progress`**, **`In Review`**
   และ **`QA`** ตรงตามตัวพิมพ์และช่องว่าง สคริปต์ค้นหา UUID ให้เอง ไม่สร้างสถานะใหม่
2. ไปที่ Linear **Settings → Security & access → Personal API keys** สร้าง key สำหรับ
   workspace `foro` ที่มีสิทธิ์อ่านงาน/สถานะและแก้ไข issue ของทีม BOO จำกัดการเข้าถึงทีม BOO
   ได้ถ้ามีตัวเลือกนี้ ไม่ต้องให้สิทธิ์สร้างหรือลบงาน
3. เปิด [GitHub Actions secrets ของ repo](https://github.com/HectorRussia/beastscribe/settings/secrets/actions)
   → **New repository secret** ตั้งชื่อ `LINEAR_HORN_PROD_BOOK` แล้ววางค่า key ในช่อง Secret
   โดยไม่เติม `Bearer` ไม่บันทึก key ลงไฟล์ ไม่ใส่ใน PR และไม่ส่งลงแชต
   `GITHUB_TOKEN` มีให้จาก GitHub Actions อัตโนมัติ ไม่ต้องสร้างเอง
4. ถ้าใช้ Linear GitHub integration อยู่ ให้ไปที่ **Settings → Team BOO → Workflows & automations
   → Pull request and commit automations** แล้วตั้งกฎเปลี่ยนสถานะสำหรับ repo/flow นี้เป็น
   **No action** รวมทั้งกฎค่าเริ่มต้น กฎเฉพาะ branch และกฎ merged/ready for merge
   เพื่อไม่ให้ชนกับ workflow หรือข้าม `QA` ไป `Done` การเปลี่ยนกฎระดับทีมอาจกระทบ repo อื่นที่ทีมใช้
5. ใน **Settings → Code & reviews** ปิดการย้าย issue ไปสถานะ Started เมื่อคัดลอกชื่อ branch
   เพื่อให้การย้ายเข้า `In Progress` เป็นการลากด้วยมือ สมาชิกแต่ละคนตรวจการตั้งค่าของตนเอง
6. นำ workflow, สคริปต์ และ tests เข้า `main` แล้วตรวจว่า repo เปิดใช้ GitHub Actions
   การเปลี่ยนใน PR ติดตั้งนี้ยังไม่ถูกใช้จนกว่าจะเข้า `main`; สร้าง `dev` จาก `main` ที่ติดตั้งแล้ว และเริ่มตรวจผลด้วย PR ถัดไป

ไม่จำเป็นต้องติดตั้ง Linear GitHub integration เพื่อให้สคริปต์เปลี่ยนสถานะผ่าน API ได้
หากต้องการให้ PR แสดงเป็น attachment ใน Linear สามารถใช้ integration สำหรับการเชื่อม PR
โดยปิดกฎเปลี่ยนสถานะที่ทับซ้อนกัน สคริปต์นี้ไม่สร้าง attachment หรือโพสต์ comment

## ชื่อ branch

ตัวอย่างจาก Linear:

```text
username/boo-123-add-feature
BOO-123-add-feature
codex/boo-123-add-feature
```

ชื่อ branch ต้องมีรหัส `BOO-เลขงาน` แบบสมบูรณ์หนึ่งรหัส ไม่จำเป็นต้องเปลี่ยนชื่อที่คัดลอกจาก Linear
ระบบรองรับตัวพิมพ์เล็ก–ใหญ่ของรหัส แต่ชื่อสถานะต้องตรงกับที่กำหนด
ไม่ใช้รหัสใน PR title, description หรือ commit message

| ตัวอย่าง | ผล |
| --- | --- |
| `username/boo-123-add-feature` | เชื่อมกับ `BOO-123` |
| `feature/no-issue` | ข้าม แม้ PR title มี `BOO-123` |
| `feature/boo-123-boo-456` | ข้าม เพราะมีหลายรหัส |
| `feature/XBOO-123` หรือ `boo-123abc` | ข้าม เพราะไม่ใช่รหัสสมบูรณ์ |

## ตรวจผลและแก้ปัญหา

เปิด [GitHub Actions](https://github.com/HectorRussia/beastscribe/actions) เลือก **Linear status sync**
แล้วดู summary หรือ step **Update Linear issue status**:

- `updated`: เปลี่ยนสถานะสำเร็จ เช่น `BOO-123: In Progress -> In Review.`
- `skipped`: ไม่เข้าเงื่อนไข พร้อมเหตุผล เช่น Draft, ไม่มีรหัสงาน หรือสถานะที่ต้องคงไว้
- `failed`: ตรวจ error annotation; ต้องแก้การตั้งค่าหรือปัญหา API แล้วจึง rerun

PR จาก fork จะถูกข้ามทั้ง job โดยไม่ส่ง Linear key เข้า job
PR เข้า branch อื่นจะไม่มี run จนกว่าแก้ base มาเป็น `dev`

| ปัญหา | วิธีตรวจ |
| --- | --- |
| `LINEAR_API_KEY is missing` | เพิ่ม repository secret `LINEAR_HORN_PROD_BOOK`; workflow ส่งค่านี้ให้สคริปต์ผ่าน environment variable `LINEAR_API_KEY` |
| HTTP 401/403 | ตรวจว่า key ยังใช้ได้และมีสิทธิ์อ่าน/แก้ไขงาน BOO |
| key ไม่ใช่ workspace `foro` | สร้าง key ใน workspace ให้ถูกต้อง |
| ไม่พบงานหรือ GraphQL errors | ตรวจรหัสงาน สิทธิ์เข้าถึง และ schema ตามเอกสาร Linear |
| ไม่พบ `In Review` หรือ `QA` ที่ตรงเพียงหนึ่งสถานะ | ตรวจชื่อสถานะและทีม BOO |
| HTTP 429/5xx หรือ timeout | รอให้บริการพร้อมแล้วใช้ Re-run failed jobs |
| mutation ไม่ยืนยันผลสำเร็จ | ตรวจสถานะใน Linear ก่อน rerun เพราะ request อาจถูกประมวลผลแล้ว |
| งานไป `Done` เองหรือสถานะเปลี่ยนเมื่อ copy branch | ตรวจ automation ใน Linear ที่ยังเปิดอยู่ |

แต่ละ run อ่าน PR ล่าสุดจาก GitHub และอ่านสถานะงานจาก Linear ก่อนแก้ไข การ rerun event เก่าหลัง merge
จะอิง PR ที่ merge แล้ว และไม่ย้ายงานจาก `QA`, `Done` หรือสถานะที่ผู้ใช้ต้องควบคุมกลับไป `In Review`
งานแต่ละ PR รันแบบเรียงคิวและไม่ยกเลิก run ที่กำลังทำงาน

Linear API ไม่มี conditional update แบบ atomic สำหรับ flow นี้ จึงยังมีช่วงสั้น ๆ ระหว่างอ่านกับเขียน
ที่การลากสถานะพร้อมกับ workflow อาจชนกัน ควรรอ job จบก่อนลากสถานะ และปิด automation ที่ทำงานซ้ำ

## ทดสอบ

ใช้ Node.js 24 จาก root ของ repo โดยไม่ต้องติดตั้ง npm dependencies หรือมี API key:

```sh
node --check .github/scripts/linear-status-sync.mjs
node --test .github/scripts/linear-status-sync.test.mjs .github/scripts/discord-main-merge.test.mjs
```

Tests ใช้ mock API ไม่มีการเรียก Linear จริง ครอบคลุมชื่อ branch, PR lifecycle, สถานะที่ควบคุมเอง,
การรันซ้ำ, event เก่า, การตรวจ workspace/team, HTTP/GraphQL errors และการไม่เปิดเผยข้อมูลจาก API ใน log

หลังตั้ง secret และนำไฟล์เข้า `main` แล้ว ทดสอบจริงด้วยงาน BOO สำหรับทดสอบหนึ่งรายการ:

1. ลากงานไป `In Progress` คัดลอกชื่อ branch แล้วสร้าง branch ภายใน repo นี้
2. เพิ่มการเปลี่ยนแปลงเล็กน้อยสำหรับทดสอบและเปิด Draft PR เข้า `dev`: งานต้องยังอยู่ `In Progress`
3. กด **Ready for review**: รอ workflow สำเร็จและตรวจว่างานเข้า `In Review`
4. ลากไป `reject` แล้ว push เพิ่ม: งานต้องยังอยู่ `reject`
5. ลากกลับ `In Progress` แล้ว push เพิ่ม: งานต้องเข้า `In Review`
6. Merge PR เข้า `dev`: งานต้องเข้า `QA`
7. Rerun run เดิมที่เปิด PR: งานต้องยังอยู่ `QA`
8. เปิด PR `dev → main` และใช้ **Create a merge commit**: Discord ต้องแจ้งงาน BOO และ Linear ยังอยู่ `QA`

การทดสอบจริงต้องมี key และ issue ที่เข้าถึงได้ ผล tests แบบ mock ไม่ถือเป็นการยืนยันว่าเชื่อมต่อจริงแล้ว

## ข้อกำหนดของ workflow

- รับ `pull_request_target` เฉพาะ `opened`, `reopened`, `ready_for_review`, `synchronize`, `edited`, `closed`
  และ base branch `dev`
- รันบน Ubuntu และ Node.js 24 ใช้ `fetch` ในตัว ไม่ต้องมี SDK หรือ npm dependencies
- ให้ GitHub token เฉพาะ `contents: read` และ `pull-requests: read`
- Checkout เฉพาะ commit ที่เชื่อถือได้จาก default branch จาก `refs/heads/main` และไม่เก็บ Git credentials
- ห้ามเปลี่ยน checkout ไปเป็น PR head/merge ref หรือเพิ่มขั้นตอนที่รันโค้ดจาก PR ใน job ที่มี Linear key
- กำหนด timeout ต่อ request 15 วินาทีและทั้ง job 5 นาที ไม่ follow HTTP redirects
- ชื่อ workspace/team/branch อยู่ในสคริปต์; หากเปลี่ยนชื่อเหล่านี้ต้องปรับสคริปต์และ workflow ให้ตรงกัน

อ้างอิง: [Linear GraphQL API](https://linear.app/developers/graphql),
[Linear GitHub integration และ automation](https://linear.app/docs/github),
[GitHub pull_request_target](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#pull_request_target)

## แจ้ง Discord เมื่อ merge เข้า main

Workflow **Discord main merge** ทำงานแยกจาก Linear sync เมื่อ PR ภายใน repo ถูก merge เข้า `main`
สำเร็จเท่านั้น การปิดโดยไม่ merge, merge เข้า `dev`, direct push และ PR จาก fork ไม่ส่งข้อความ
งานใน Linear คงสถานะเดิมทั้งหมด รวมถึง `QA` ไม่ถูกย้ายไป `Done`

### ตั้งค่า Discord

1. ในช่อง Discord แบบข้อความที่ต้องการรับแจ้ง ไปที่ **Edit Channel → Integrations → Webhooks**
   สร้าง webhook และคัดลอก Webhook URL ต้องมีสิทธิ์จัดการ webhook ในช่องนั้น
2. เพิ่ม repository Actions secret ชื่อ **`DISCORD_WEBHOOK_URL`** ที่
   [หน้า secrets](https://github.com/HectorRussia/beastscribe/settings/secrets/actions)
   วาง URL เต็ม เช่นรูปแบบ `https://discord.com/api/webhooks/<id>/<token>`
   ไม่ส่ง URL จริงลงแชตหรือ commit ลง repo
3. ใช้ repository secret **`LINEAR_HORN_PROD_BOOK`** เดิมเพื่ออ่านชื่อและผู้รับผิดชอบงานสำหรับข้อความ
   ไม่ต้องสร้าง Discord bot หรือ Discord bot token
4. นำ workflow และสคริปต์ทั้งหมดเข้า `main` และสร้าง branch `dev` จาก `main` นั้นก่อนเริ่ม flow
   คู่มือนี้ไม่ถือว่ามีการ push ไฟล์ สร้าง remote branch หรือตั้ง secrets ให้แล้ว

สำหรับส่งเข้า thread สามารถใช้ webhook URL ที่มี `?thread_id=<เลข thread>` ได้
รุ่นแรกไม่สร้าง thread ใหม่ใน forum channel อัตโนมัติ

### วิธีรวมงานแต่ละรอบ

- เปิด PR **`dev → main`** และเลือก **Create a merge commit** ทุกครั้ง เพื่อคงประวัติที่ใช้แยกงานแต่ละรอบ
  PR งานย่อยเข้า `dev` สามารถใช้ squash หรือ merge commit ได้ ไม่ต้องปิด squash ของทั้ง repo
- สคริปต์อ่านสอง parent ของ merge commit: main ก่อน merge และ dev ของรอบนั้น แล้วใช้ GitHub compare API
  อ่าน commit ใหม่ทุกหน้า โดยไม่ใช้ปลาย branch ปัจจุบันซึ่งอาจมีงานรอบใหม่เข้าไปแล้ว
- อ่าน PR ที่ merge เข้า `dev` ทุกหน้า แล้วจับคู่ `merge_commit_sha` กับ commit ใหม่ ดึงรหัส BOO จากชื่อ branch
  และตัดรหัสซ้ำ งานรอบก่อนที่อยู่ในประวัติ main แล้วจะไม่ถูกนำมาแจ้งอีก
- หาก squash/rebase PR `dev → main` สคริปต์จะ fail พร้อมคำอธิบาย ไม่ส่งรายการที่อาจปนงานรอบเก่า
  การ rerun ไม่สามารถซ่อมประวัติ merge ผิดรูปแบบได้ ต้องตรวจประวัติ branch ก่อนรอบถัดไป
- สำหรับ branch งานที่ merge เข้า main โดยตรง ใช้รหัส BOO จากชื่อ branch นั้น
  ไม่อ่านรหัสจาก PR title, description หรือ commit message
- หากไม่พบงาน BOO ให้แจ้งสรุป PR พร้อมข้อความ “ไม่พบงาน BOO ที่เชื่อมกับ PR นี้”
  ถ้า branch ที่รวมอยู่มีหลายรหัส BOO หรืออ่านข้อมูล Linear ไม่สำเร็จ ให้ fail ก่อนส่ง Discord

### รูปแบบข้อความและข้อจำกัด

ข้อความใช้ชื่อ **Deployment Bot**, embed สีเขียว และหัวข้อ **🚀 Merged to main** พร้อมจำนวนงาน
รหัส/ชื่องาน ผู้รับผิดชอบ ลิงก์ Linear ลิงก์ PR ผู้ merge และเวลาไทย `Asia/Bangkok`
งานไม่มีผู้รับผิดชอบแสดง “ยังไม่ระบุ” ใช้ข้อมูลชื่อ/ผู้รับผิดชอบล่าสุดใน Linear ณ ตอนรัน

ข้อความไม่แสดง “Deployment complete” หรือ “QA → Done” เพราะยังไม่มี deployment pipeline
และ workflow นี้ไม่เปลี่ยนสถานะงาน รายการยาวจะแบ่งเป็นหลายข้อความพร้อมเลขหน้า
ไม่เรียก mentions และไม่ดึงภาพ avatar จากข้อมูลภายนอก

ดูผลที่ GitHub Actions → **Discord main merge** → **Notify Discord** หรือ summary:

- `sent`: Discord ยืนยันว่าบันทึกข้อความแล้วผ่าน `wait=true`
- `skipped`: ไม่เข้าเงื่อนไขการแจ้ง
- `failed`: ตรวจ error เช่น secret ขาด, API ปฏิเสธสิทธิ์, rate limit หรือ timeout แล้วแก้ก่อน rerun

ไม่บันทึกประวัติการส่งถาวร ดังนั้น **rerun อาจส่งซ้ำ** แม้ run ก่อนสำเร็จแล้ว
ถ้าส่งหลายข้อความแล้วล้มเหลวกลางทาง log จะแจ้งจำนวนข้อความที่ยืนยันแล้ว
timeout อาจเกิดหลัง Discord รับข้อความไปแล้ว ควรตรวจช่องก่อน rerun
สคริปต์ไม่ retry POST อัตโนมัติเพื่อลดการส่งซ้ำ และไม่แสดง webhook URL/token ใน log

อ้างอิง: [Discord Webhook](https://docs.discord.com/developers/resources/webhook#execute-webhook),
[GitHub Compare API](https://docs.github.com/en/rest/commits/commits#compare-two-commits)
