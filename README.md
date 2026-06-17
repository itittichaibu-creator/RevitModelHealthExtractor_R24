<h1 align="center">📊 Revit Model Health Extractor (R24)</h1>

<p align="center">
  <strong>เครื่องมือสกัดข้อมูลสุขภาพของโมเดล (Model Health) แบบอัตโนมัติสำหรับ Autodesk Revit 2024 ออกรายงานเป็น Excel ได้ทันที</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Revit%20API-2024-blue?style=for-the-badge&logo=autodesk" alt="Revit 2024">
  <img src="https://img.shields.io/badge/Language-Python-3776AB?style=for-the-badge&logo=python" alt="Python">
  <img src="https://img.shields.io/badge/Dynamo-Node-red?style=for-the-badge" alt="Dynamo">
</p>

---

## 💡 ภาพรวม (Overview)
การตรวจสอบความเรียบร้อยหรือสุขภาพของโมเดล (Model Health) ไม่ใช่เรื่องยากอีกต่อไป! 
**Revit Model Health Extractor** คือเครื่องมือ (รันผ่าน Dynamo) ที่ช่วยให้คุณสกัดข้อมูลสำคัญต่างๆ ในโมเดล Revit 2024 ออกมาเป็นไฟล์ Excel (.xlsx) ได้อย่างรวดเร็ว พร้อมรองรับการกรอก Note เพื่อเปรียบเทียบในแต่ละ Phase (เช่น 50% DD) และยังคงประวัติการดึงข้อมูลไว้ในไฟล์เดิม

## ✨ ฟีเจอร์หลัก (Key Features)
- 🚀 **สกัดข้อมูลครอบคลุมทุกมิติ:**
  - จำนวนและรายการ **Warnings**
  - รายละเอียด **Model / Detail Groups** (Placement Counts, Member Counts)
  - รายการ **CAD Links** (เช็คสถานะ Pinned, Loaded)
  - รายการ **Views & Sheets**
  - ตรวจสอบ **Line Styles & Fill Patterns** ที่อาจเป็นขยะแฝง (เช่น จาก AutoCAD)
  - จำนวนการใช้งาน **Loadable Families**
  - ตรวจสอบสถานะ **Worksets & Revit Links**
- 🎯 **รู้ทันทีว่าใครเป็นคนทำ (Creator Tracking):** ติดตามชื่อผู้สร้าง (Worksharing User) ในรายการ Views, Sheets, CAD Links และ Line Styles
- 📊 **บันทึกทับไฟล์เดิมได้ (Append Data):** ระบบจัดกลุ่มข้อมูลอัตโนมัติใน Excel ทำให้สามารถเปิด-ปิดดูประวัติการดึงข้อมูลในแต่ละรอบ (Phase) ได้สะดวก
- 🖥️ **UI ใช้งานง่าย:** มีหน้าต่างป๊อปอัป (Save File Dialog) ให้เลือกที่เซฟไฟล์ และหน้าต่างกรอก "Extraction Note" ก่อนสกัดข้อมูล
- 🧩 **รองรับทุกสภาพแวดล้อม:** ตรวจจับขนาดไฟล์ได้ถูกต้อง ไม่ว่าจะเป็น Local, Revit Server หรือ Cloud Model

## 🛠️ การติดตั้ง (Installation)
1. เปิดโปรแกรม Revit 2024 และเข้าไปที่โปรเจกต์ของคุณ
2. เปิดโปรแกรม **Dynamo**
3. ลากสคริปต์ Python ในโปรเจกต์นี้ (`RevitModelHealthExtractor_R24.py`) ไปใช้งานในโหนดของ Dynamo หรือรันผ่าน Dynamo Player
4. ตรวจสอบให้แน่ใจว่าในเครื่องมีไลบรารี `openpyxl` สำหรับ Python (หากยังไม่มี ต้องติดตั้งเพิ่มเติมใน Environment ของ Dynamo)

## 🚀 วิธีใช้งาน (How to Use)
1. เปิดโมเดล Revit ที่ต้องการตรวจสอบ
2. รันสคริปต์ **Model Health Extractor** ผ่าน Dynamo
3. เมื่อมีหน้าต่างป๊อปอัปเด้งขึ้นมา ให้เลือกตำแหน่งที่ต้องการบันทึกไฟล์ Excel (`.xlsx`)
4. กรอกหมายเหตุหรือเฟสการทำงาน (Extraction Note) เช่น *Pre-Submission*, *50% DD*
5. รอสคริปต์ทำงานจนเสร็จ ข้อมูลจะถูกเขียนลง Excel พร้อมจัดกลุ่มแบบอัตโนมัติ (สามารถนำไปวิเคราะห์ต่อได้ทันที)

## 🧑💻 ประวัติการพัฒนา (Development Log)
โปรเจกต์นี้มีไฟล์บันทึกการอัปเดต (`CHANGELOG.md`) และคู่มือแบบละเอียด (`Manuals/ModelHealth_Extractor_Manual.html`) เพื่อใช้อ้างอิงและทำความเข้าใจการทำงานของโค้ดเบื้องหลัง

---
*พัฒนาด้วย ❤️ เพื่อยกระดับการทำงานสาย BIM*
