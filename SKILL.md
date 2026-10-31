name: find-skills
slug: find-skills
displayName: ค้นหาทักษะ · Find Skills
description: ไม่รู้จะติดตั้งสกิลอะไรดี? อยากให้ AI ค้นหาสกิลที่เหมาะสมที่สุดโดยอัตโนมัติหรือไม่? "ค้นหาทักษะ · Find Skills" เป็นเครื่องมือค้นหาสกิลสองระบบที่ขับเคลื่อนด้วยบริบทและคีย์เวิร์ด (skill discovery / skill finder / install skills) ไม่ว่าจะอธิบายสถานการณ์หรือความต้องการด้วยภาษาธรรมชาติ (เช่น "อยากทำโปสเตอร์" "ช่วยวิเคราะห์หุ้นให้หน่อย" "หาสกิลล่าโปรโมชั่น/ส่วนลด") หรือสั่งตรงๆ เช่น "ติดตั้งสกิล / find skills / หาสกิล / มีสกิลทำ XX ไหม" ระบบจะทำการค้นหาแบบรวมศูนย์ 6 ชั้นจาก Built-in ทางการ, สกิลที่ติดตั้งแล้วในเครื่อง, SkillHub, Xiaping (虾评), GitHub และ ClawHub พร้อมแนะนำสกิลที่ตรงจุดที่สุดและติดตั้งได้ในคลิกเดียว เข้ามาแทนที่ปลั๊กอิน find-skills เดิมของทางการอย่างสมบูรณ์ คำกระตุ้น (Trigger): 找技能, 安装技能, 技能推荐, 技能发现, find skills, skill discovery, install skill
version: "1.7.0"

find-skills (ระบบจับคู่สกิลตามบริบทการใช้งาน)

ภาพรวม (Overview)

สกิลนี้ออกแบบมาเพื่อเป็น เครื่องมือค้นหาสกิลที่ขับเคลื่อนด้วยบริบทการใช้งาน (Scenario-Driven) โดยผู้ใช้เพียงอธิบายความต้องการด้วยภาษาพูดทั่วไป ระบบจะวิเคราะห์เจตนา ค้นหาข้ามแหล่งข้อมูลแบบรวมศูนย์ และแนะนำสกิลที่เหมาะสมที่สุดให้อัตโนมัติ

ความสัมพันธ์กับปลั๊กอิน find-skills ตัวเดิมของทางการ:

ปลั๊กอิน find-skills ดั้งเดิมถูกตั้งค่าสถานะเป็น DEPRECATED (เลิกใช้งานแล้ว) และจะไม่ถูกทริกเกอร์โดยอัตโนมัติ

สกิลนี้ครอบคลุมความสามารถเดิมทั้งหมดอย่างสมบูรณ์ พร้อมเพิ่มฟังก์ชันการสแกน Built-in ในระบบ, สแกนสกิลที่ติดตั้งในเครื่อง, วิเคราะห์บริบทความหมาย (Semantic Understanding) และแจกแจงเหตุผลประกอบการแนะนำ

กระบวนการทำงานหลัก (Core Workflow)

ขั้นตอนที่ 1: ทำความเข้าใจบริบทของผู้ใช้ (Understand User Scenario)

สกัดข้อมูลต่อไปนี้จากข้อความภาษาธรรมชาติของผู้ใช้:

เจตนาของงาน (Task Intent): ผู้ใช้ต้องการทำอะไร?

หมวดหมู่/ขอบเขต (Domain Tag): จัดอยู่ในสายงานหรือกลุ่มใด?

คำค้นหา (Search Keywords): ทั้งภาษาจีนและภาษาอังกฤษ (สำหรับค้นหาบนแหล่งข้อมูลภายนอก)

ตัวอย่าง:

"อยากทำโปสเตอร์สักใบ" $\rightarrow$ เจตนา: ออกแบบ/ทำกราฟิก; หมวดหมู่: การสร้างคอนเทนต์ (Content Creation); คีย์เวิร์ด: poster, design, 海报, 设计

"ช่วยวิเคราะห์ตลาดหุ้นวันนี้ให้หน่อย" $\rightarrow$ เจตนา: วิเคราะห์หุ้น; หมวดหมู่: การเงิน (Finance); คีย์เวิร์ด: stock, A股, 大盘, 分析

ขั้นตอนที่ 2: การค้นหาแบบรวมศูนย์ 3 ระดับ (Three-Tier Federated Search)

2.1 ระดับที่ 1: สกิล Built-in ทางการ (มาพร้อม WorkBuddy ไม่ต้องติดตั้งเพิ่ม)

สแกนไดเรกทอรีสกิลมาตรฐานที่ฝังมากับตัวแอป แล้วอ่าน name และ description ของแต่ละสกิล:

ls /Applications/WorkBuddy.app/Contents/Resources/app.asar.unpacked/resources/builtin-skills/


ใช้เครื่องมือ Read เพื่ออ่าน YAML frontmatter ในไฟล์ SKILL.md ของแต่ละสกิล แล้วทำ Semantic Matching เทียบกับบริบทของผู้ใช้

เกณฑ์การให้คะแนนความตรงกัน (ตามลำดับความสำคัญ):

คำสำคัญในบริบทของผู้ใช้ปรากฏตรงๆ ใน description ของสกิล $\rightarrow$ คะแนนสูง

ชื่อสกิล (name) สอดคล้องกับเจตนาของผู้ใช้อย่างชัดเจน $\rightarrow$ คะแนนสูง

คำอธิบาย (description) สอดคล้องกับหมวดหมู่งานของผู้ใช้ $\rightarrow$ คะแนนปานกลาง

2.2 ระดับที่ 2: สกิลที่ติดตั้งไว้แล้วในเครื่อง (Local Installed Skills)

สแกนสกิลที่ติดตั้งไว้แล้วจาก 2 ตำแหน่งต่อไปนี้:

# สกิลระดับผู้ใช้ (User-level)
ls ~/.workbuddy/skills/

# สกิลระดับโปรเจกต์ (Workspace ปัจจุบัน)
ls .workbuddy/skills/ 2>/dev/null || echo "ไม่มีสกิลระดับโปรเจกต์"


ใช้เครื่องมือ Read ดึง YAML frontmatter ใน SKILL.md เพื่อนำ name และ description มาจับคู่ความหมายกับบริบทของผู้ใช้

ข้อควรระวัง: หากสกิลที่ติดตั้งในเครื่องตรงกับสกิล Built-in ทางการในขั้นตอน 2.1 ให้ตัดรายการที่ซ้ำออก โดยเก็บเฉพาะรายการที่มีลำดับความสำคัญสูงสุดไว้

2.3 ระดับที่ 3: แหล่งตลาดสกิลภายนอก (Remote Marketplaces)

ตรวจสอบแคช Local Marketplace ก่อน (อิงตรรกะ Step 5 ของ find-skills เดิม):

ls ~/.workbuddy/skills-marketplace/skills 2>/dev/null


หากพบสกิลที่ตรงความต้องการอยู่ในแคชท้องถิ่น ให้คัดลอกมาติดตั้งได้ทันที ไม่จำเป็นต้องดาวน์โหลดซ้ำ:

cp -r ~/.workbuddy/skills-marketplace/skills/<skill-folder-name> ~/.workbuddy/skills/<skill-folder-name>


การค้นหาจากระยะไกล (Remote Search) (อิงตรรกะ Step 2/2b ของ find-skills เดิม):

หากไม่พบในแคชท้องถิ่น ให้ดำเนินการค้นหาตามลำดับแหล่งข้อมูลดังนี้:

① SkillHub Official Marketplace (แหล่งหลัก ค้นหาก่อนเป็นอันดับแรก):

curl -s "https://lightmake.site/api/v1/search?q=<URL-encoded คีย์เวิร์ดภาษาจีน>&limit=10"
curl -s "https://lightmake.site/api/v1/search?q=<URL-encoded คีย์เวิร์ดภาษาอังกฤษ>&limit=10"


กรองผลลัพธ์ที่มีความตรงกันต่ำออก (score < 0.05)

② ตลาดสกิลเซี่ยผิง - Xiaping (虾评) (แหล่งรวมสกิลภาษาจีนเป็นหลัก):

Base URL ของ Xiaping API: https://xiaping.coze.com (หมายเหตุ: ไม่ใช่ coze.site)

ค้นหาสกิล:

# ค้นหาสกิลตามคีย์เวิร์ด
curl -s "https://xiaping.coze.com/api/skills/search?q=<URL-encoded คีย์เวิร์ด>&limit=10"

# ดูรายละเอียดสกิล
curl -s "https://xiaping.coze.com/api/skills/<skill-id>"

# ดึงลิงก์ดาวน์โหลดสกิล
curl -s "https://xiaping.coze.com/api/skills/<skill-id>/download"


ต้องโหลดสกิล คู่มือเซี่ยผิง (虾评指南) ก่อน (ที่อยู่ไฟล์: ~/.workbuddy/skills/虾评指南/SKILL.md) แล้วดำเนินการค้นหาและดาวน์โหลดตามข้อกำหนดในหัวข้อ §1.1 + §1.2

ข้อสำคัญ: Xiaping เป็นแหล่งสกิลภาษาจีนที่ครอบคลุมมาก หากเป็นงานที่ใช้ภาษาจีน ผลลัพธ์จาก Xiaping มักจะตรงจุดและใช้งานได้ดีกว่า SkillHub

③ คลังสกิลบน GitHub (GitHub Repositories) (แหล่งสกิลแบบโอเพนซอร์ส):

ค้นหาไฟล์สกิลของ WorkBuddy / Claw บน GitHub

หมายเหตุเรื่องสิทธิ์การเข้าถึง: GitHub Search API จำเป็นต้องมีการยืนยันตัวตน หากมีการตั้งค่าตัวแปรสภาพแวดล้อม GITHUB_TOKEN ระบบจะแนบสิทธิ์ให้อัตโนมัติ (โควตา 5,000 ครั้ง/ชั่วโมง) หากไม่มี จะใช้งานผ่านโควตา Anonymous (60 ครั้ง/ชั่วโมง ซึ่งอาจไม่เพียงพอต่อการใช้งาน)

# หากมี GitHub Token ให้ใส่ Header สำหรับ Authentication
if [ -n "$GITHUB_TOKEN" ]; then
  AUTH_HEADER="-H \"Authorization: Bearer $GITHUB_TOKEN\""
else
  AUTH_HEADER=""
fi

# ค้นหาโค้ดที่มีไฟล์ SKILL.md
curl -s $AUTH_HEADER "https://api.github.com/search/code?q=filename:SKILL.md+<คีย์เวิร์ด>&per_page=10"

# ค้นหารีโพสิทอรีที่เกี่ยวข้องกับสกิล
curl -s $AUTH_HEADER "https://api.github.com/search/repositories?q=<URL-encoded คีย์เวิร์ด>+skill+in:name,description&per_page=10"


ข้อควรระวัง: GitHub Code Search API (filename:SKILL.md) ต้องยืนยันตัวตนเท่านั้นจึงจะใช้งานได้ หากไม่มี GITHUB_TOKEN ระบบจะลดระดับมาค้นหาเฉพาะ Repositories อย่างเดียว (ค้นหาแบบ Anonymous ได้)

วิธีการติดตั้ง:

# Clone คลังสกิลมาจาก GitHub
git clone "https://github.com/<user>/<repo>.git" <target-skills-dir>/<skill-name>/

# ตรวจสอบความถูกต้องของการติดตั้ง
ls <target-skills-dir>/<skill-name>/SKILL.md


ข้อควรระวัง: ผลลัพธ์จาก GitHub จำเป็นต้องมีการตรวจสอบด้วยตนเองอีกครั้งว่ามีไฟล์ SKILL.md ที่สมบูรณ์หรือไม่ เนื่องจากการค้นหาโค้ดอาจติดไฟล์ทั่วไปที่ไม่ใช่โครงสร้างสกิลที่ถูกต้องกลับมาด้วย
