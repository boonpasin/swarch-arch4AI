# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** Pasin Thanyakasikol พศิน ธัญญกสิกล

**Student ID / รหัสนิสิต:** 6732025921

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:** B (last digit 1 is odd)

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Availability | the kiosk accepts 100% of reviews even when offline for minutes. no review is lost |
| 2 | Accuracy | at least 90% of labels correct (QR3) |
| 3 | Timeliness / cost | every review labelled by end of day. send as few as possible to the large model |

## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

T4 asynchronous / batch. the kiosk saves the review on the tablet first then syncs when the wifi comes back. the review service stores it as pending - same as the fallback in Task 1. a batch job scores the pending ones every few minutes with cascade (threshold 0.7) and cache. owners only need labels by end of day so nobody waits for the model

```mermaid
C4Container
    title Canteen kiosk (scenario B): container diagram
    Person(student, "Student", "Writes a review on the canteen tablet")
    Person(owner, "Restaurant owner", "Reads labels at the end of the day")
    System_Boundary(campuseats, "CampusEats") {
        Container(kiosk, "Kiosk app", "Tablet app, local storage", "Saves every review on the tablet first and syncs when Wi-Fi is back")
        Container(gateway, "API gateway", "Python, FastAPI", "Single entry point")
        Container(review, "Review service", "Python, FastAPI", "Stores reviews as pending")
        Container(batch, "Batch scorer", "Python, scheduled job", "Scores pending reviews with cascade + cache")
        Container(inference, "Inference service", "Python, FastAPI, scikit-learn", "Small model and large model")
    }
    Rel(student, kiosk, "Writes review")
    Rel(kiosk, gateway, "Syncs saved reviews", "HTTP/JSON, retry")
    Rel(gateway, review, "Forwards requests", "HTTP/JSON")
    Rel(batch, review, "Reads pending, writes labels", "HTTP/JSON")
    Rel(batch, inference, "POST /predict", "REST")
    Rel(owner, gateway, "Reads review counts by label", "HTTP/JSON")
    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| t1-after | with fallback 0 errors and 23 pending instead of 20 errors in t1-before. saving pending and scoring later loses nothing |
| t2-0.7 | cascade keeps 93.0% accuracy with only 53% large calls vs 100% in t0-large |
| t3-cache | cache on top brings large calls down to 38% at the same 93.0% |
| t0-small | small model alone is only 72.5% so running only the small model on the tablet fails QR3 |

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** Store reviews on the kiosk and score them in batch with cascade + cache

**Status:** Proposed

**Context:** canteen wifi drops several times a day for minutes. every review must be accepted even offline. owners only need labels by end of day. large model is accurate but slow and costs money per call (20 calls/s max)

**Decision:** T4 batch. the kiosk keeps reviews in local storage and syncs with retry when online. the review service saves them as pending. a scheduled batch job labels pending reviews using cascade (0.7) plus cache

**Consequences:**
- Positive: no review is lost when offline. no user waits for the model. the batch can run slowly so the 20 calls/s limit is never a problem. cascade + cache cut large calls to about 38% (t3-cache) with 93% accuracy
- Negative: labels are late - minutes to hours. the kiosk needs local storage and sync logic and we must handle duplicates when it retries. one more thing to run and monitor (the batch job)

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> T1 synchronous - the tablet can't reach the service when offline so reviews fail. that breaks the main requirement. T2 small model on the tablet - works offline but only 72.5% (t0-small). T3 hybrid would still need the network for hard reviews and real-time labels aren't needed here anyway so batch is simpler
