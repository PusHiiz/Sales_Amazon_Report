# Sales_Amazon_Report (Kaggle)
**Data source:**
- Amazon Sales Report.csv (transaction_level)
- Sale Report.csv (information product e.g. sku_code, desigh, size, color, stock)

**Tools Used:** Python (Pandas, NumPy)

## Load data

## Data Profiling Table Amazon Sales Report
- ทุก columns จาก Amazon Sales Report เพื่อทำความเข้าใจข้อมูล และดูความผิดปกติเบื้องต้นเพื่อทำ Data Cleaning ในขั้นตอนต่อไป
- ในขั้นตอนนี้มีการ Data Cleaning rename columns ไปบ้าง เพราะเพื่อความสะดวกในการเรียก column นั้น ๆ
- Add column unit_price เพราะดูแล้วจำเป็น แต่ดันไม่มี
## Data Cleaning Table Amazon Sales Report
- change type ship_postal float64 to object เพราะรหัสไปรษณีย์ไม่สามารถนำมาคำนวนได้ ถึงแม้จะเป็นตัวเลขก็ตาม
- จำนวนแถวที่ไม่มีข้อมูลสถานที่จัดส่ง (เช่น `ship_city`, `ship_state`, `ship_postal_code`, `ship_country`) มีจำนวน 33 records จากทั้งหมด 128,975 records คิดเป็นเพียง 0.0256% ของข้อมูลทั้งหมด เนื่องจากข้อมูลดังกล่าวจำเป็นต่อการวิเคราะห์ตามพื้นที่/ภูมิภาค เช่น การทำ Heatmap, Geo Analysis, หรือการแยกยอดขายตามจังหวัด จึงได้ **ลบข้อมูลเหล่านี้ออกก่อนการวิเคราะห์** เพื่อรักษาความถูกต้องของ Insight ที่ได้จากข้อมูล


## Data Profiling Table Sale Report
- ตรวจเจอว่า sku มีค่า #REF เราต้อง Clean ให้เป็น NaN
- ตรวจพบว่ามีค่าที่เป็น NaN ทุก columns (ยกเว้น index) ที่เริ่มต้นจาก index ที่ 9235 ถึง 9271 รวม 36 rows
## Data Cleaning Table Sale Report
- rename columns sku_code เป็น sku เพื่อจะเอาไป match
- ลบ 36 rows ที่ index เกินไป นี้ออกเพื่อความถูกต้องในการคำนวนต่อไป
- ตอนที่เราเริ่มตรวจสอบข้อมูล เราได้พบว่า column sku มีค่า #REF! อยู่ ซึ่งทำให้ข้อมูลไม่สามารถแสดงได้ เราจึงจำเป็นต้องจัดการกับค่า #REF! ซึ่งเราไม่ลบ แต่เราเปลี่ยนเป็น NaN เพราะว่ายังมี columns อื่นที่ใช้ประโยชน์ได้
- Change type stock float to int ต้องทำตอนที่ไม่มีค่า NaN เท่านั้น

## Merge table ครั้งที่ 1
- merge โดย sku table Amazon Sales Report กับ sku table Sale Report พบว่า mismatch ถึง 7705 ซึ่งรู้สึกว่าเยอะมาก
- หาสาเหตุว่าทำไมถึงเยอะ ลองไปลบช่องว่างของ key ที่จะ merge แล้วก็ได้เท่าเดิม พอไปตรวจสอบจริง ๆ ก็พบว่า mismatch จริง ๆ
- ระหว่างเราเช็คว่า 7705 ที่ match ไม่ได้มาจากไหน เราไปพบว่า style กับ design_no. และ size (t1) กับ size (t2) มีค่าแบบเดียวกัน (sub-key) เราจึงเลือกอันนี้มาใช้ merge แทน
- แต่เราจะเลือกเฉพาะอันใดอันนึงไม่ได้ เพราะอาจจะมีค่าที่ผิดเพี้ยนไป เช่น style กับ design_no. match กันได้ แต่ size กับ size match กันไม่ได้ จึงจำเป็นต้องรวมทั้ง 2 อันเป็น column ใหม่เพื่อใช้ merge

## Prepare before Merge again
### Summary
- match size (t1) กับ size (t2) ได้ 100%
- match style กับ design_no. ไม่ได้ 100% ซึ่ง match ไม่ได้ 96
- หลังจาก combined style_size กับ design_size match ไม่ได้เพิ่ม 17
- รวมที่ match ไม่ได้เป็น 113
- เราไม่ได้ clean ค่าพวกนี้ออก ส่วนมากเราไม่ได้ใช้เพราะใน table1 ไม่มีค่าพวกนั้น
- 17 ค่าที่เพิ่มมามาจาก style match design กันได้ แต่ไม่มี size ที่ match กันได้ เช่น JNE3360 ใน style กับ design มีเหมือนกัน แต่ใน size (t1) เป็น S แต่ size (t2) มีแค่ XL JNE3360_S not match JNE3360_XL
- ถึงจะ match ไม่ได้ แต่เรายังเอาค่าจาก table1 ไปใส่ table2 ได้ โดนเอาข้อมูลใน sku ใน table1

## Merge table ครั้งที่ 2
- หลังจากดูความเป็นไปแล้ว match ได้เยอะกว่าจริง ๆ mismatch แค่ 113 เอง
- หลังจาก Merge แล้ว ผลกลับไม่เป็นไปแบบที่คิด ดันมีค่าเพิ่มมากขึ้น จากปกติ 128,975 แถว กลายเป็น 129,090 แถว เพิ่มมากขึ้น (ใช้ Left Join ไม่ควรเป็นแบบนั้น)
- ตรวจพบว่ามี Table Sale Report มีค่าซ้ำถึง 15 ค่า ทำให้เกิด duplicated ขึ้น
- design_size ค่าเดียว แต่มีหลาย records ซึ่ง records เหล่านั้น sku เป็น NaN
- เราจัดการโดยใช้ groupby เพราะมี records นึง ในแถว sku ที่เป็น NaN มี stock เป็น 1 เราจึงต้องรวมเข้ามาด้วย

## Merge table ครั้งที่ 3
- ครั้งนี้ค่าเป็นไปตามที่ต้องการแล้ว แต่ตอนก่อนเราจะ merge แล้วรู้แล้วว่า มันจะเติม stock ไปใน sku ที่ไม่ได้ match กันด้วย (7705 แถว) ซึ่งตรงนี้เราต้องเติม 0

## Data Cleaning after Merge
- เติม stock 0 ใน sku (table1) != sku (table2) ที่เติม 0 ไม่ใช่ NaN เพราะว่าค่าที่ไม่ได้ match แสดงว่าของจริง ๆ ไม่มีแล้ว อาจจะไม่ได้ผลิตแล้ว stock จึงควรเป็น 0 (ถ้าเป็นสถานการณ์จริง ต้องตรวจสอบจากสถานที่จริง ๆ ด้วย)
- ตอนที่เราจะ Merge เราก็ไปเห็นว่า sku (table1) มีสีบอกอยู่ด้วย เราจึงจะเอาสีมาเติมในค่าที่เป็น NaN
- สรุปเติมได้อยู่ 2 สี ก็คือ Black กับ Mustard
- ตรวจสอบ missing value ที่เหลือ
- เริ่มจัดการกับ columns ที่สำคัญก่อน ก็คือ amount, currency, unit_price

*Project ได้ดำเนินการถึงตอนนี้ ขั้นตอนก่อน EDA ซึ่งจะมีการอัพเดตต่อไปเรื่อย ๆ จนจบกระบวนการเพื่อนำไปทำ Dashboard และ Presentation ต่อไปครับ*

