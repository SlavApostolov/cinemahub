# CinemaHub – Spring Boot project plan (team of 2–3)

Deadline: **16.01.2027** (send GitHub link + faculty numbers early – earlier = more time to fix gaps)
Test 1: **27.11.2026** | Test 2: **20.12.2026**

## 1. What it is
A REST backend ("server") for a cinema: movies, screenings, seat booking, reviews.
Not just a movie catalogue – the **seat booking** gives us the required non-CRUD business process
(two people must not be able to book the same seat at the same time).

## 2. Requirements → how we cover them
| Requirement | Our answer |
|---|---|
| 5+ entities | Movie, Genre, Hall, Seat, Screening, Customer, Booking, Review (8) |
| 1 Many-to-Many | Movie <-> Genre |
| 1 Many-to-One | Screening -> Movie, Screening -> Hall, Booking -> Screening, Booking -> Customer, Review -> Movie |
| Controller/Service/Repository/DTO/Mapper | package-by-layer (see below); Mapper = MapStruct or hand-written |
| 1 custom @Query | e.g. "free seats for a screening", "top-rated movies" |
| Validation + @RestControllerAdvice | `@Valid`, `@NotBlank`, `@Min`...; one `GlobalExceptionHandler` returning JSON errors |
| Swagger/OpenAPI | `springdoc-openapi-starter-webmvc-ui` -> `/swagger-ui.html` |
| Tests + 1 integration test | JUnit 5 + Mockito for services; `@SpringBootTest` for the booking scenario |
| Business logic | Booking a seat: no double booking, even under concurrent requests |

## 3. Data model (simple version)
- **Genre**(id, name)
- **Movie**(id, title, description, durationMin, releaseYear, rating) – M:N Genre
- **Hall**(id, name, rows, seatsPerRow)
- **Seat**(id, hall_id, rowNo, seatNo)
- **Screening**(id, movie_id, hall_id, startTime, price)
- **Customer**(id, name, email unique)
- **Booking**(id, screening_id, seat_id, customer_id, status [CONFIRMED/CANCELLED], createdAt)
  - **UNIQUE(screening_id, seat_id)** -> the database itself refuses a double booking
- **Review**(id, movie_id, customer_id, stars 1–5, text)

## 4. The "hard" part: preventing double booking
Layers of defense (explain this at the defense, it's the impressive bit):
1. DB unique constraint `(screening_id, seat_id)` – last line of defense.
2. `@Transactional` in `BookingService.book(...)`.
3. Catch `DataIntegrityViolationException` -> return HTTP 409 "Seat already taken".
4. (Optional bonus) `@Version` optimistic locking or pessimistic lock on Screening.
Integration test: 2+ threads (ExecutorService + CountDownLatch) book the same seat -> exactly 1 succeeds.
Also: cancelled booking frees the seat; can't book a screening that already started.

## 5. Packages
```
com.cinemahub
 ├─ controller   (MovieController, BookingController, ...)
 ├─ service
 ├─ repository   (extends JpaRepository)
 ├─ entity
 ├─ dto          (request/response records)
 ├─ mapper
 └─ exception    (custom exceptions + GlobalExceptionHandler)
```

## 6. Setup (day 1)
1. Install **JDK 25** (Temurin) + **IntelliJ IDEA** + **PostgreSQL** (as the course slides say).
   Maven: you don't need to install it – start.spring.io generates `mvnw`.
2. https://start.spring.io -> Maven, Java, latest stable Spring Boot, deps:
   Spring Web, Spring Data JPA, PostgreSQL Driver, Validation, H2 (tests), Lombok (optional).
   Add springdoc-openapi manually in `pom.xml`.
3. Create GitHub repo, add teammate, protect `main`, work in feature branches + pull requests.
4. Tests use **H2** (no setup for anyone); dev uses PostgreSQL.

## 7. Timeline (about 9 weeks, at ~4–5 h/week)
| Dates | Goal |
|---|---|
| 16–27 Nov | Test 1 prep (Basics, Methods, Classes, Spring Boot). Only: repo + Spring Initializr project that runs |
| 28 Nov–6 Dec | Entities + repositories + Movie/Genre CRUD (the M:N). Swagger working |
| 7–20 Dec | Test 2 prep (Collections, Maven, Interfaces, Lambdas, Concurrency). Light work: DTOs + mappers |
| 21 Dec–3 Jan | Hall/Seat/Screening/Customer, **booking service** + concurrency handling |
| 4–10 Jan | Validation, global exception handler, @Query, Reviews |
| 8–12 Jan | Tests (unit + integration), README, cleanup. **Submit link by ~12 Jan, earlier is better** |
| to 16 Jan | Buffer / fix gaps |
Note: Test 2 includes **Concurrency & Threads** – that is exactly what the booking feature needs, so learning one helps the other.

## 8. Work split (2 people)
- **A:** Movie, Genre, Review, Swagger, global error handling
- **B:** Hall, Seat, Screening, Customer, Booking + concurrency test
- Both: review each other's PRs (so you BOTH understand all the code – the teacher may ask).

## 9. Optional extras (only if ahead)
- Pagination/search on movies; OMDb/TMDB external API to import movie info; Spring Security login.
  Not required for a team of 2–3.

## 10. Learning order (for you, while BroCode runs)
Classes/OOP -> Collections -> Exceptions -> Threads/synchronization -> then Spring Boot basics
(controller -> service -> repository -> JPA relationships). For Spring, a ~2–3 h Spring Boot REST tutorial
(Baeldung / Spring official guides) beats long general Java videos for this project.
