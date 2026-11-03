# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** Pasin Thanyakasikol พศิน ธัญญกสิกล

**Student ID / รหัสนิสิต:** 6732025921

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 512 | 608 | 93.5% | 100% |
| t0-small | 29 | 68 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

> the large model buys accuracy. 93.5% vs 72.5% - small model gets about 1 in 4 wrong and fails QR3. the cost is time and money. p50 512 ms vs 29 ms and every review is a paid large call

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 28.6 | 235 | 352 | 20 | 0 | 92.8% |
| t1-after | 10.0 | 429 | 1018 | 0 | 23 | 92.7% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> p50 only counts requests that got an answer. the 20 hung ones timed out and just disappear from the latency numbers. also while they hang there is less queue for the others so the rest look fast. the real story is errors 20 and run time 28.6 s vs 10 s

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> a hung request doesn't just wait. it holds a slot the whole time - a connection in the gateway and a worker in the review service. with only 10 in flight a few hung ones block everyone behind them. so the slowness spreads to reviews that never touched a hung call

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> they should get labelled later by a batch job. store them as pending and every few minutes or at night score them all again with the large model. this is the asynchronous / batch topology (T4)

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> 0.1 s is shorter than one normal large call (~200 ms) so almost every review becomes pending even when nothing is wrong. 10 s is the same as the client timeout so a hang still holds the slot for 10 s and the user already gave up. the fallback is useless at that point

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 12 | 603 | 91.5% | 38% |
| t2-0.7 | 0.7 | 410 | 610 | 93.0% | 53% |
| t2-0.8 | 0.8 | 467 | 662 | 93.5% | 64% |
| t2-0.9 | 0.9 | 469 | 592 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> 0.7. it gets 93.0% so QR3 has margin and only 53% of reviews go to large. 0.6 is cheaper (38%) but 91.5% is too close to 90% on 200 reviews. I give up some cost (QR4) for safety on accuracy

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> cascade only makes the easy reviews fast. those were already under p50 so p50 drops. but the hard ones still go small then large then queue for the 4 slots. p95 is those hard ones so it stays around 600 ms

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> sarcasm like 'great another cold meal' - lots of positive words so the small model is very sure it's positive. it never goes to large. the owner sees a happy review and misses an unhappy customer

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 11 | 576 | 93.0% | 38% | 35% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> short common reviews that repeat exactly like 'delicious' or 'too salty'. 35% hits in t3-cache and large calls went 53% -> 38%. long unique reviews never hit because nobody writes the exact same text twice

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> the old answers are from the old model so the cache must be cleared or it keeps serving old labels. I would put the model version in the cache key - like 'large-v2:delicious'. new model new version so old entries just never match and get evicted by LRU

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> p50 can lie. in t1-before p50 looked better than baseline while 10% of requests failed. you need errors and run time next to the percentiles or a broken system looks faster
