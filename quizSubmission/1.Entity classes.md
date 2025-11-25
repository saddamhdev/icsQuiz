Below is a **complete, production-ready set of Entity Classes (Spring Boot + JPA + PostgreSQL)**
for your **Quiz + Submission System**, including:

* `User`
* `Quiz`
* `Question`
* `Submission`
* `SubmissionAnswer`
* (Optional) `QuizQuestion` mapping entity

All entities follow:
✔ JPA + Hibernate
✔ PostgreSQL-friendly types
✔ `jsonb` for multi-select answers
✔ Bi-directional relations where needed
✔ Lombok for clean code

---

# ✅ **1. User Entity**

```java
package com.quiz.app.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Column(unique = true)
    private String email;

    private String passwordHash;
}
```

---

# ✅ **2. Quiz Entity**

```java
package com.quiz.app.entity;

import jakarta.persistence.*;
import lombok.*;
import java.util.List;

@Entity
@Table(name = "quizzes")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Quiz {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private Integer totalQuestions;

    private Integer duration;  // in minutes

    private Boolean shuffle;

    @OneToMany(mappedBy = "quiz", cascade = CascadeType.ALL)
    private List<Question> questions;
}
```

---

# ✅ **3. Question Entity**

Uses **JSONB** to store options + correct answers.

```java
package com.quiz.app.entity;

import jakarta.persistence.*;
import lombok.*;
import org.hibernate.annotations.Type;

import java.util.List;

@Entity
@Table(name = "questions")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Question {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "quiz_id")
    private Quiz quiz;

    @Column(columnDefinition = "text")
    private String questionText;

    @Type(org.hibernate.type.JsonBinaryType.class)
    @Column(columnDefinition = "jsonb")
    private List<String> options;            // ["A", "B", "C", "D"]

    @Type(org.hibernate.type.JsonBinaryType.class)
    @Column(columnDefinition = "jsonb")
    private List<String> correctAnswers;     // ["A","C"] for multi-select
}
```

---

# ✅ **4. Submission Entity**

Heavy-write table → optimized.

```java
package com.quiz.app.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.Instant;
import java.util.List;

@Entity
@Table(name = "submissions")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Submission {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    @ManyToOne
    @JoinColumn(name = "quiz_id")
    private Quiz quiz;

    private Instant startTime;
    private Instant endTime;
    private Integer duration;

    private Instant submittedAt;

    private Integer score;

    @Column(length = 20)
    private String status;   // "submitted", "processing", "done"

    @OneToMany(mappedBy = "submission", cascade = CascadeType.ALL)
    private List<SubmissionAnswer> answers;
}
```

---

# ✅ **5. SubmissionAnswer Entity**

Supports **multi-select** answers with JSONB.

```java
package com.quiz.app.entity;

import jakarta.persistence.*;
import lombok.*;
import org.hibernate.annotations.Type;

import java.util.List;

@Entity
@Table(name = "submission_answers")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class SubmissionAnswer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "submission_id")
    private Submission submission;

    @ManyToOne
    @JoinColumn(name = "question_id")
    private Question question;

    @Type(org.hibernate.type.JsonBinaryType.class)
    @Column(columnDefinition = "jsonb")
    private List<String> selectedOptions;  // ["A"], ["A","C"], or []

    private Boolean correct;

    private Integer timeTaken;  // seconds spent
}
```

---

# 🧩 **Optional: QuizQuestion (Mapping Table)**

If your quiz has many many questions (bank → selected subset)

```java
@Entity
@Table(name = "quiz_questions")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class QuizQuestion {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name="quiz_id")
    private Quiz quiz;

    @ManyToOne
    @JoinColumn(name="question_id")
    private Question question;

    private Integer orderIndex;
}
```

---

# ⚙️ Add Dependencies for JSONB Support (IMPORTANT)

**pom.xml**

```xml
<dependency>
    <groupId>com.vladmihalcea</groupId>
    <artifactId>hibernate-types-60</artifactId>
    <version>2.21.1</version>
</dependency>
```

---

# ⭐ Ready to Continue?

I can give you next:

### ✔ DTOs

### ✔ Request Payload classes

### ✔ Submission Service

### ✔ WebFlux Controller

### ✔ Kafka Producer + Consumer

### ✔ PostgreSQL Schema (SQL file)

### ✔ Locust Load Test Script (20k users)

Just tell me — **which part you want next?**
