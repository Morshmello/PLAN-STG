# Git Branch Strategy

เอกสารสรุปแนวทางการใช้งาน branch สำหรับทีม

---

## Branches

| Branch | หน้าที่ | Reset |
|--------|--------|-------|
| `develop` | Integration ของทีม dev | ทุก 1-2 สัปดาห์ |
| `preview` (**optional**) | ให้ลูกค้าทดสอบเท่านั้น code ใกล้เคียง main | ทุก 1-2 สัปดาห์ |
| `main` | UAT เทียบเท่า production | - |
| `tag` | Production (v.x.x.x) | - |

---

## Naming Convention

- `feat/xxx` — feature ใหม่
- `fix/xxx` — แก้ bug ปกติ
- `hotfix/xxx` — แก้ด่วนบน production

---

## Workflows

### Flow: Feature ใหม่ (พร้อมให้ลูกค้าดู)

```
main → feat/xxx → PR เข้า develop (ทีมเทส)
                → (QA pass) PR เข้า preview (ลูกค้าดู)
                → (ลูกค้า CF) PR เข้า main → tag
```

### Flow: Feature dev (ยังไม่พร้อมให้ลูกค้าดู)

```
main → feat/xxx → PR เข้า develop เท่านั้น
```

### Flow: Fix bug ปกติ

```
main → fix/xxx → PR เข้า develop (เทส)
               → PR เข้า main → tag
```

### Flow: Hotfix

```
main → hotfix/xxx → PR เข้า main → tag
                  → back-merge เข้า develop และ preview
```

### Flow: Cancel (ลูกค้าไม่เอา / ยกเลิก feature)

- ปิด PR + ลบ feature branch
- commit ที่ค้างใน develop/preview จะหายตอน reset รอบถัดไป
- ถ้าเร่งด่วน (กระทบระบบ) → revert commit ทันที ไม่ต้องรอ reset

---

## กฎเหล็กของ Feature Branch

1. **Update `feat/xxx` จาก `main` เท่านั้น** — ห้าม merge `develop` เข้า `feat/xxx` เด็ดขาด
2. **PR เข้า `main` = ใช้ feature branch ตัวเดิม** — ไม่ใช่ merge `develop` เข้า `main`
3. **เปิด Squash Merge** เป็น default ใน GitHub Settings → Pull Requests

---

## การ Resolve Conflict

> ⚠️ **ห้ามใช้ GitHub Web UI conflict resolver**
> เพราะมันจะ merge `develop` เข้า `feat/xxx` ทำให้ feature branch สกปรก

### วิธีที่ถูกต้อง — resolve ในเครื่องบนฝั่งปลายทาง

```bash
git checkout develop
git pull
git merge --no-ff feat/xxx
# แก้ conflict
git add .
git commit
git push
```

PR จะ auto-close, `feat/xxx` ยังสะอาดอยู่ที่เดิม

### วิธีกู้ `feat/xxx` ที่สกปรกไปแล้ว

```bash
git checkout feat/xxx
git reflog                    # หา commit hash ก่อนโดน merge develop
git reset --hard <commit>
git push --force-with-lease
```

---

## Reset Policy (`develop` และ `preview`)

**ความถี่:** ทุก 1-2 สัปดาห์ (ตาม sprint cycle)

### ขั้นตอน

```bash
git checkout develop
git reset --hard main
git push --force-with-lease origin develop
```

(`preview` ทำเหมือนกัน)

### ก่อน reset

- TL/PM แจ้งทีมล่วงหน้าอย่างน้อย 1 วัน
- รวบรวมรายชื่อ feature branch ที่ยัง active

### หลัง reset

- Re-merge feature branch ที่ยัง active กลับเข้า `develop` / `preview` ทีละตัว
- Feature ที่ตายแล้ว / ลูกค้าไม่ตอบ → ปล่อยให้หายไป

---

## Gate ก่อน Merge

| ปลายทาง | เงื่อนไข |
|---------|---------|
| `develop` | PR review ผ่าน |
| `preview` | QA pass บน develop แล้ว |
| `main`    | ลูกค้า CF จาก preview แล้ว |
| `tag`     | ทีมยืนยันพร้อมขึ้น production |

---

## Checklist เริ่มใช้งาน

- [ ] สร้าง branch `preview` จาก `main`
- [ ] ตั้ง branch protection rule: `develop`, `preview`, `main`
- [ ] เปิด Squash Merge เป็น default
- [ ] ตกลงรอบ reset (แนะนำ 1-2 สัปดาห์)
- [ ] กำหนดผู้รับผิดชอบ reset (TL/PM)
- [ ] อบรมทีมเรื่องวิธี resolve conflict ในเครื่อง (ห้ามใช้ UI)
- [ ] เขียน flow นี้ลง README หรือ Wiki ของ repo

---

## FAQ

**Q: ทำไมต้องมี `preview` แยกจาก `develop`?**
A: เพราะ `develop` ใช้ทีม dev integration งานและอาจไม่เสถียร ส่วน `preview` ต้องเสถียรพอให้ลูกค้าทดสอบได้ การแยก branch ช่วยไม่ให้สองหน้าที่นี้ขัดกัน

**Q: ทำไมต้อง reset develop/preview เป็นระยะ?**
A: เพราะ feature ที่ลูกค้าไม่ตอบหรือยกเลิกจะค้างสะสมเรื่อยๆ ทำให้ branch บวมและ diverge จาก main มาก การ reset เป็นระยะช่วยให้ branch สะอาดและลดโอกาสเกิด conflict

**Q: ถ้า reset แล้ว feature ที่ยัง active หายไปไหน?**
A: ตัว feature branch (`feat/xxx`) ยังอยู่ครบ แค่ commit ใน `develop` / `preview` ถูกล้าง ต้อง re-merge feature branch กลับเข้าไปใหม่

**Q: ทำไมห้ามใช้ GitHub UI resolve conflict?**
A: GitHub จะ auto-merge `develop` เข้า `feat/xxx` เพื่อ resolve ทำให้ feature branch มี commit ของ `develop` ติดมา พอ PR เข้า `main` ทีหลังจะลาก code อื่นที่ไม่เกี่ยวขึ้นไปด้วย
