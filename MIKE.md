เอกสาร FRS: ระบบ AI Outbound Debt Collection (SaaS Platform)
1. ภาพรวมของระบบ (System Overview)
ระบบ AI Voice Agent สำหรับการโทรออกอัตโนมัติ (Outbound Call) เพื่อเจรจาทวงถามยอดหนี้และทำนัดหมายชำระเงิน โดยใช้เทคโนโลยี Full-duplex Voice AI ที่สามารถโต้ตอบแบบเรียลไทม์ รองรับการพูดแทรก (Barge-in) และบันทึกผลการเจรจากลับลงฐานข้อมูลโดยอัตโนมัติ ออกแบบสถาปัตยกรรมแบบ Multi-tenant เพื่อให้บริการในรูปแบบ B2B SaaS สำหรับโบรคเกอร์หรือเอเจนซี่

2. โครงสร้างเทคโนโลยี (Technology Stack)
Voice Orchestrator: Dograh (Self-hosted Visual Builder)

AI Models: Deepgram (STT), OpenAI GPT-4o (LLM), ElevenLabs (TTS)

Telephony: SIP Trunk / FreeSWITCH / GSM Gateway

Backend & DB: Node.js หรือ Python / PostgreSQL / Redis (Message Queue)

3. สิทธิ์การใช้งาน (User Roles)
Super Admin (ผู้ให้บริการ SaaS): จัดการโควตาและกำหนดสิทธิ์ให้ลูกค้า (Tenants), ดูแลภาพรวมระบบเซิร์ฟเวอร์

Tenant Admin (ลูกค้า B2B / โบรคเกอร์): อัปโหลดรายชื่อลูกค้า, จัดการแคมเปญการโทร, ดูรายงานและผลการเจรจา

AI Agent (ระบบอัตโนมัติ): ดำเนินการโทรออก, เจรจาตาม System Prompt, ประเมินสถานการณ์, โอนสาย, และส่ง Webhook บันทึกผล

4. ข้อกำหนดทางฟังก์ชัน (Functional Requirements)
4.1 ระบบจัดการแคมเปญและข้อมูลลูกค้า (Campaign & Lead Management)
FR1.1 Upload Leads: ระบบต้องรองรับการอัปโหลดไฟล์ CSV/Excel (ชื่อ, เบอร์โทร, ยอดหนี้, วันครบกำหนด) โดย Tenant Admin

FR1.2 Data Validation: ระบบต้องตรวจสอบความถูกต้องของเบอร์โทรศัพท์ให้อยู่ในรูปแบบที่โทรออกได้ (เช่น +66) และลบเบอร์ซ้ำในแคมเปญเดียวกัน

FR1.3 Campaign Status: ระบบต้องมีสถานะของแคมเปญ เช่น Draft, Running, Paused, Completed

FR1.4 Lead Status: ระบบต้องติดตามสถานะรายชื่อแต่ละเบอร์ เช่น Pending, Calling, Completed, Failed, Voicemail

4.2 ระบบควบคุมการโทรออกอัตโนมัติ (Auto-Dialer Engine)
FR2.1 Queue Management: ระบบ Backend ต้องดึงเบอร์โทรที่มีสถานะ Pending มาใส่ใน Message Queue เพื่อเตรียมส่งให้ AI โทรออก

FR2.2 Concurrency Control: ระบบต้องจำกัดจำนวนคู่สายโทรออกพร้อมกัน (Concurrent Calls) ไม่ให้เกินโควตาของ SIP Trunk หรือแพ็กเกจที่ Tenant ซื้อไว้

FR2.3 Dynamic Payload Injection: ก่อนส่งคำสั่งโทร ระบบต้องดึงข้อมูลลูกค้า (ชื่อ, ยอดหนี้) เข้าไปแทนที่ใน System Prompt (Override Variables) และส่ง HTTP POST ไปยังเซิร์ฟเวอร์ Dograh

FR2.4 Retry Logic: หากเบอร์สายไม่ว่าง หรือไม่รับสาย ระบบต้องตั้งเวลาโทรซ้ำอัตโนมัติ (เช่น อีก 2 ชั่วโมง) ตามจำนวนครั้งที่กำหนดสูงสุด

4.3 ระบบปัญญาประดิษฐ์และการสนทนา (Voice AI & Conversation Flow)
FR3.1 Full-Duplex Audio: ระบบต้องรองรับการฟังและการพูดพร้อมกัน (Two-way communication)

FR3.2 Voice Activity Detection (VAD) & Barge-in: ระบบต้องสามารถหยุดพูดทันทีเมื่อลูกค้าพูดแทรก และประมวลผลคำพูดแทรกนั้นเพื่อตอบสนองต่อบริบทใหม่

FR3.3 First Message Trigger: ระบบต้องพูดประโยคเปิด (Greeting) ตามที่กำหนดทันทีเมื่อตรวจพบเสียงมนุษย์รับสาย

FR3.4 State Machine Navigation: ระบบต้องบังคับ AI ให้ดำเนินการตามลำดับขั้น: ยืนยันตัวตน -> แจ้งยอด -> เจรจาต่อรอง -> สรุปผล

FR3.5 Call Transfer (Escalation): ระบบต้องสามารถโอนสายเข้าเบอร์ของพนักงานมนุษย์ได้ทันที หากตรวจพบว่าลูกค้าโกรธ, ไม่ยอมรับหนี้, หรือร้องขอคุยกับมนุษย์

4.4 ระบบการจัดการหลังการโทร (Post-Call & Webhooks)
FR4.1 Function Calling (Tools): AI ต้องสามารถเรียกใช้ฟังก์ชันบันทึกข้อมูล (เช่น save_negotiation_result) เมื่อสรุปวันชำระเงินกับลูกค้าได้

FR4.2 Webhook Processing: ระบบ Backend ต้องมี Endpoint สำหรับรับ Payload JSON จาก Dograh เมื่อสายตัด หรือเมื่อมีการเรียกใช้ Tool

FR4.3 Database Update: ระบบต้องแยกวิเคราะห์ JSON และนำผลลัพธ์ (เช่น วันที่ตกลงชำระ, เหตุผลที่ขอเลื่อน, สถานะการโทร) ไปอัปเดตในตาราง Leads ทันที

FR4.4 Transcript Storage: ระบบต้องบันทึกประวัติข้อความการสนทนาทั้งหมด (Transcript) เพื่อให้ Tenant Admin ตรวจสอบย้อนหลังได้

4.5 ระบบแดชบอร์ดและรายงาน (Dashboard & Reporting)
FR5.1 Real-time Dashboard: แสดงผลรวมของสายที่โทรสำเร็จ, สายที่ไม่รับ, ยอดหนี้ที่ตกลงชำระได้แล้วแบบเรียลไทม์

FR5.2 Export Data: Tenant Admin ต้องสามารถดาวน์โหลดผลการเจรจากลับมาเป็นไฟล์ CSV/Excel ได้

FR5.3 Billing & Usage: ระบบต้องแสดงจำนวนนาทีที่ AI ใช้ไปในแต่ละรอบบิล เพื่อให้ Super Admin ใช้คิดเงินกับ Tenant

5. ข้อกำหนดที่ไม่ใช่ฟังก์ชัน (Non-Functional Requirements)
NFR1. Low Latency: ความหน่วงในการตอบสนองของ AI นับตั้งแต่ลูกค้าพูดจบจนถึง AI เริ่มเปล่งเสียง (Time-to-First-Byte) ต้องไม่เกิน 800 มิลลิวินาที

NFR2. Multi-tenant Isolation: ข้อมูลรายชื่อลูกค้าและฐานข้อมูลของ Tenant แต่ละรายต้องถูกแยกออกจากกันอย่างเด็ดขาดเพื่อความปลอดภัย

NFR3. PDPA & Compliance: ระบบต้องไม่แจ้งยอดหนี้หรือเปิดเผยข้อมูลความลับในข้อความแรก จนกว่าลูกค้าปลายทางจะยืนยันตัวตนสำเร็จตามกฎหมาย พ.ร.บ. ทวงถามหนี้

NFR4. Scalability: สถาปัตยกรรมของ Worker และ Queue ต้องรองรับการเพิ่มจำนวนโหนดเพื่อรองรับการขยายตัวของการโทรหลักหมื่นสายต่อวันได้



โครงสร้างฐานข้อมูล (PostgreSQL Schema)

-- เปิดการใช้งาน UUID extension (หากยังไม่ได้เปิด)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 1. ตาราง Tenants (ลูกค้า B2B / โบรคเกอร์ หรือบริษัททวงหนี้)
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    company_name VARCHAR(255) NOT NULL,
    contact_email VARCHAR(255),
    contact_phone VARCHAR(50),
    max_concurrent_calls INT DEFAULT 5, -- โควตาคู่สายที่โทรได้พร้อมกัน
    status VARCHAR(50) DEFAULT 'active', -- active, suspended
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 2. ตาราง Users (ผู้ใช้งานในระบบ แยกตาม Tenant)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(50) DEFAULT 'tenant_admin', -- super_admin, tenant_admin, viewer
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. ตาราง Campaigns (แคมเปญการโทรของแต่ละ Tenant)
CREATE TABLE campaigns (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    campaign_name VARCHAR(255) NOT NULL,
    base_system_prompt TEXT, -- Prompt เริ่มต้นสำหรับแคมเปญนี้
    status VARCHAR(50) DEFAULT 'draft', -- draft, running, paused, completed
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 4. ตาราง Leads (รายชื่อลูกค้าเป้าหมายที่ต้องโทรหา)
CREATE TABLE leads (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE, -- ใส่ไว้เพื่อความรวดเร็วในการ Query แยก Tenant
    
    -- ข้อมูลลูกค้า (นำไป Injection ใน Prompt)
    customer_name VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20) NOT NULL,
    debt_amount DECIMAL(12, 2) NOT NULL,
    due_date DATE,
    
    -- สถานะการโทรของระบบ (System State)
    call_status VARCHAR(50) DEFAULT 'pending', -- pending, calling, completed, failed, voicemail
    retry_count INT DEFAULT 0,
    next_retry_at TIMESTAMP WITH TIME ZONE,
    
    -- ผลลัพธ์จากการเจรจา (ได้จาก Webhook ของ AI)
    negotiation_result VARCHAR(100), -- agreed, requested_postpone, refused, transferred, wrong_person
    promise_to_pay_date DATE, -- วันที่ลูกค้ายืนยันจะจ่าย (ถ้ามี)
    ai_summary TEXT, -- AI สรุปสั้นๆ จากการคุย
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 5. ตาราง Call_Logs (ประวัติการโทรและบันทึกสนทนา)
CREATE TABLE call_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    lead_id UUID REFERENCES leads(id) ON DELETE CASCADE,
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    
    call_sid VARCHAR(100), -- ID อ้างอิงจากระบบ SIP/Dograh
    duration_seconds INT, -- ระยะเวลาที่คุย (เพื่อนำไปคิดเงิน)
    sip_status VARCHAR(50), -- answered, no_answer, busy, failed
    
    transcript JSONB, -- เก็บบทสนทนาทั้งหมดเป็น JSON Array (ใครพูดอะไรตอนไหน)
    recording_url VARCHAR(500), -- ลิงก์ไฟล์เสียงที่บันทึกไว้
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- ⚡ สร้าง Indexes เพื่อความเร็วในการค้นหาและจัดการ Queue
CREATE INDEX idx_leads_call_status ON leads(call_status);
CREATE INDEX idx_leads_tenant_id ON leads(tenant_id);
CREATE INDEX idx_leads_campaign_id ON leads(campaign_id);
CREATE INDEX idx_call_logs_tenant_id ON call_logs(tenant_id);



