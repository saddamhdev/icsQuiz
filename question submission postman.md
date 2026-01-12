Below is **EXACT Postman-ready code** (requests + JSON bodies) for **your current system**.

You can **copy-paste directly into Postman** or use **curl**.

---

# 🔹 1️⃣ ACTIVITY API (USED DURING EXAM)

### Endpoint

```
POST http://localhost:8080/api/exam/activity
```

### Headers

```
Content-Type: application/json
Authorization: Bearer <JWT_TOKEN>   (optional)
```

---

## ✅ A. SINGLE_CHOICE (Radio)

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 101,
  "questionType": "SINGLE_CHOICE",
  "action": "SELECT",
  "value": 3
}
```

👉 Selecting option **3**
👉 Redis stores: `[3]`

---

## ✅ B. TRUE_FALSE

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 102,
  "questionType": "TRUE_FALSE",
  "action": "SELECT",
  "value": 1
}
```

👉 `1 = TRUE`, `0 = FALSE`

---

## ✅ C. MULTIPLE_CHOICE – SELECT option

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 103,
  "questionType": "MULTIPLE_CHOICE",
  "action": "SELECT",
  "value": 2
}
```

---

## ✅ D. MULTIPLE_CHOICE – UNSELECT option

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 103,
  "questionType": "MULTIPLE_CHOICE",
  "action": "UNSELECT",
  "value": 2
}
```

---

## ✅ E. TEXT

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 201,
  "questionType": "TEXT",
  "action": "TEXT_CHANGE",
  "value": "Java is platform independent"
}
```

---

## ✅ F. NUMERIC

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 301,
  "questionType": "NUMERIC",
  "action": "NUMERIC_CHANGE",
  "value": 42
}
```

---

## ✅ G. DESCRIPTIVE

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5,
  "questionId": 401,
  "questionType": "DESCRIPTIVE",
  "action": "TEXT_CHANGE",
  "value": "JVM provides portability by using bytecode..."
}
```

---

# 🔹 2️⃣ FINALIZE / SUBMIT EXAM (MCQ SNAPSHOT)

This is used when:

* User clicks **Submit**
* Time expires
* Crash recovery
* Admin force submit

---

### Endpoint

```
POST http://localhost:8080/api/exam/finalize/mcq
```

### Headers

```
Content-Type: application/json
```

### Body

```json
{
  "packageId": 1,
  "examId": 10,
  "userId": 5
}
```

👉 This will:

* Read Redis
* Delete old MCQSubmission
* Insert final MCQ rows

---

# 🔹 3️⃣ CURL VERSION (OPTIONAL)

### Activity (MCQ Select)

```bash
curl -X POST http://localhost:8080/api/exam/activity \
-H "Content-Type: application/json" \
-d '{
  "packageId":1,
  "examId":10,
  "userId":5,
  "questionId":103,
  "questionType":"MULTIPLE_CHOICE",
  "action":"SELECT",
  "value":2
}'
```

---

### Finalize MCQ

```bash
curl -X POST http://localhost:8080/api/exam/finalize/mcq \
-H "Content-Type: application/json" \
-d '{
  "packageId":1,
  "examId":10,
  "userId":5
}'
```

---

# 🔹 4️⃣ WHAT YOU SHOULD VERIFY AFTER REQUEST

### ✅ Database

```sql
SELECT * FROM exam_activity_log;
SELECT * FROM mcq_submission;
```

### ✅ Redis

```bash
redis-cli
HGETALL exam:live:1:10:5
```

---

# 🧠 FINAL CHECKLIST

✔ Postman activity works
✔ Redis updates live
✔ DB logs every action
✔ Finalize creates MCQSubmission

---

If you want next:
1️⃣ Postman for **TEXT / NUMERIC final tables**
2️⃣ Postman **restore answers from Redis**
3️⃣ Postman **evaluation API**
4️⃣ Export **Postman collection (.json)**

Just tell me 👇
