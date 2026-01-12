Perfect 👍
We’ll **safely add `packageId` everywhere** (frontend → backend → Redis → DB) without changing your logic.

Below is the **FULL UPDATED CODE**, only **necessary changes**, clean and consistent.

---

## 🧠 Updated Key Rule (VERY IMPORTANT)

**Redis Key now includes `packageId`**
So answers from different packages never mix.

```
exam:live:{packageId}:{examId}:{userId}
```

---

## 🟢 FRONTEND (React) — UPDATED

### Common sender (UPDATED)

```jsx
async function sendActivity(payload) {
  await fetch("/api/exam/activity", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(payload)
  });
}
```

---

### SINGLE_CHOICE / TRUE_FALSE

```jsx
function SingleChoice({ packageId, examId, userId, question }) {

  const onSelect = (optionId) => {
    sendActivity({
      packageId,
      examId,
      userId,
      questionId: question.id,
      questionType: question.type,
      action: "SELECT",
      value: optionId
    });
  };

  return (
    <>
      <h4>{question.text}</h4>
      {question.options.map(opt => (
        <label key={opt.id}>
          <input
            type="radio"
            name={`q-${question.id}`}
            onChange={() => onSelect(opt.id)}
          />
          {opt.text}
        </label>
      ))}
    </>
  );
}
```

---

### MULTIPLE_CHOICE

```jsx
function MultiChoice({ packageId, examId, userId, question }) {

  const onChange = (optionId, checked) => {
    sendActivity({
      packageId,
      examId,
      userId,
      questionId: question.id,
      questionType: "MULTIPLE_CHOICE",
      action: checked ? "SELECT" : "UNSELECT",
      value: optionId
    });
  };

  return (
    <>
      <h4>{question.text}</h4>
      {question.options.map(opt => (
        <label key={opt.id}>
          <input
            type="checkbox"
            onChange={e => onChange(opt.id, e.target.checked)}
          />
          {opt.text}
        </label>
      ))}
    </>
  );
}
```

---

### TEXT

```jsx
function TextQuestion({ packageId, examId, userId, question }) {

  let timer;

  const onChange = (e) => {
    clearTimeout(timer);
    const value = e.target.value;

    timer = setTimeout(() => {
      sendActivity({
        packageId,
        examId,
        userId,
        questionId: question.id,
        questionType: "TEXT",
        action: "TEXT_CHANGE",
        value
      });
    }, 800);
  };

  return (
    <>
      <h4>{question.text}</h4>
      <input type="text" onChange={onChange} />
    </>
  );
}
```

---

### NUMERIC

```jsx
function NumericQuestion({ packageId, examId, userId, question }) {

  const onChange = (e) => {
    sendActivity({
      packageId,
      examId,
      userId,
      questionId: question.id,
      questionType: "NUMERIC",
      action: "NUMERIC_CHANGE",
      value: e.target.value
    });
  };

  return (
    <>
      <h4>{question.text}</h4>
      <input type="number" onChange={onChange} />
    </>
  );
}
```

---

### DESCRIPTIVE

```jsx
function DescriptiveQuestion({ packageId, examId, userId, question }) {

  let timer;

  const onChange = (e) => {
    clearTimeout(timer);
    const value = e.target.value;

    timer = setTimeout(() => {
      sendActivity({
        packageId,
        examId,
        userId,
        questionId: question.id,
        questionType: "DESCRIPTIVE",
        action: "TEXT_CHANGE",
        value
      });
    }, 1500);
  };

  return (
    <>
      <h4>{question.text}</h4>
      <textarea rows={6} onChange={onChange} />
    </>
  );
}
```

---

## 🔵 BACKEND (Spring Boot) — UPDATED

### ActivityRequest DTO (UPDATED)

```java
@Data
public class ActivityRequest {

    private Long packageId;   // ✅ added
    private Long examId;
    private Long userId;
    private Long questionId;

    private QuestionType questionType;
    private String action;
    private Object value;
}
```

---

### Controller (NO LOGIC CHANGE)

```java
@RestController
@RequestMapping("/api/exam")
@RequiredArgsConstructor
public class ExamActivityController {

    private final ExamActivityService service;

    @PostMapping("/activity")
    public void activity(@RequestBody ActivityRequest req) {
        service.handle(req);
    }
}
```

---

### Service (UPDATED Redis key)

```java
@Service
@RequiredArgsConstructor
public class ExamActivityService {

    private final ExamActivityLogRepository logRepo;
    private final RedisTemplate<String, Object> redisTemplate;

    @Transactional
    public void handle(ActivityRequest req) {

        // 1️⃣ Save immutable history
        logRepo.save(ExamActivityLog.from(req));

        // 2️⃣ Update Redis live state
        updateRedis(req);
    }

    private void updateRedis(ActivityRequest req) {

        String redisKey = "exam:live:"
                + req.getPackageId() + ":"
                + req.getExamId() + ":"
                + req.getUserId();

        String qKey = req.getQuestionId().toString();

        switch (req.getQuestionType()) {

            case MULTIPLE_CHOICE -> {
                List<Long> selected =
                        (List<Long>) redisTemplate.opsForHash()
                                .get(redisKey, qKey);

                if (selected == null) selected = new ArrayList<>();

                Long optionId = Long.valueOf(req.getValue().toString());

                if ("SELECT".equals(req.getAction())) {
                    if (!selected.contains(optionId))
                        selected.add(optionId);
                } else {
                    selected.remove(optionId);
                }

                redisTemplate.opsForHash().put(redisKey, qKey, selected);
            }

            case SINGLE_CHOICE, TRUE_FALSE -> {
                redisTemplate.opsForHash().put(
                        redisKey,
                        qKey,
                        List.of(Long.valueOf(req.getValue().toString()))
                );
            }

            case TEXT, DESCRIPTIVE -> {
                redisTemplate.opsForHash().put(
                        redisKey,
                        qKey,
                        req.getValue().toString()
                );
            }

            case NUMERIC -> {
                redisTemplate.opsForHash().put(
                        redisKey,
                        qKey,
                        Double.valueOf(req.getValue().toString())
                );
            }
        }
    }
}
```

---

## 🟣 DATABASE — UPDATED ACTIVITY LOG

### Entity (UPDATED)

```java
@Entity
@Table(name = "exam_activity_log")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ExamActivityLog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long packageId;   // ✅ added
    private Long examId;
    private Long userId;
    private Long questionId;

    @Enumerated(EnumType.STRING)
    private QuestionType questionType;

    private String action;
    private String value;
    private Instant createdAt;

    public static ExamActivityLog from(ActivityRequest r) {
        return ExamActivityLog.builder()
                .packageId(r.getPackageId())
                .examId(r.getExamId())
                .userId(r.getUserId())
                .questionId(r.getQuestionId())
                .questionType(r.getQuestionType())
                .action(r.getAction())
                .value(String.valueOf(r.getValue()))
                .createdAt(Instant.now())
                .build();
    }
}
```

---

### SQL Table (UPDATED)

```sql
CREATE TABLE exam_activity_log (
  id BIGSERIAL PRIMARY KEY,
  package_id BIGINT,
  exam_id BIGINT,
  user_id BIGINT,
  question_id BIGINT,
  question_type VARCHAR(30),
  action VARCHAR(30),
  value TEXT,
  created_at TIMESTAMP
);
```

---

## 🔴 REDIS — FINAL LIVE STATE (UPDATED)

### Key

```
exam:live:{packageId}:{examId}:{userId}
```

### Example

```json
{
  "101": [2,4],
  "201": "Java is portable",
  "301": 42,
  "401": "Detailed explanation..."
}
```

---

## 🧠 FINAL SUMMARY (MEMORIZE)

* `packageId` added to:

  * ✅ Frontend payload
  * ✅ Backend DTO
  * ✅ Redis key
  * ✅ DB activity log
* Redis now **fully isolated per package**
* Zero behavior change, only **data safety improved**

---

### Want next?

1️⃣ Restore answers from Redis using `packageId`
2️⃣ Auto-submit + timeout logic
3️⃣ Final answer table design
4️⃣ Performance tuning (50K users)

Just tell me the number 👇
