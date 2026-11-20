# CinemaHub – Task board (learn-by-doing)

## How we work
1. You do a task **yourself** (Google, Baeldung, docs, BroCode are all fine).
2. Commit on a branch -> tell Claude "review task X" -> Claude reads your code and lists what is missing/wrong
   and explains WHY. Claude does not hand over finished code unless you are stuck after trying.
3. Fix the gaps, open a Pull Request, your partner reviews it, merge to `main`.
Each task has: **Goal**, **Learn**, **Done when**. Tick the box when merged.

## The split
| Person A (you) – "Catalog side" | Person B (partner) – "Booking side" |
|---|---|
| Genre, Movie (the Many-to-Many) | Hall, Seat |
| Customer | Screening |
| Review | Booking + double-booking protection |
| Swagger, validation, global error handler | @Query for free seats, concurrency integration test |
| Unit tests for Movie/Review services | Unit tests for Booking service |
A does more *breadth* (simpler tasks), B does the *hardest* logic. Both write README parts + ER diagram together.
If it feels unfair later, A takes the free-seats @Query.

Rule to avoid merge conflicts: **only touch your own entities/files**. Shared files (pom.xml, application.properties)
are changed only in small, separate PRs that are merged quickly.

---

## Phase 0 – Setup (finish by ~22 Nov, before Test 1 crunch)
- [ ] **0.1 (A)** Clone repo, generate project at start.spring.io, push on branch `setup/skeleton`, PR, merge.
  - Learn: what Maven/pom.xml is, what a Spring Boot starter is, git branch/PR basics.
  - Done when: `main` contains pom.xml, `CinemaHubApplication`, and the empty packages (controller, service, repository, entity, dto, mapper, config, exception – add a `package-info.java` or `.gitkeep` so Git keeps them).
- [ ] **0.2 (B)** Install JDK 25, IntelliJ, PostgreSQL; create DB `cinemahub`; clone repo.
- [ ] **0.3 (A+B)** `application.properties` reads DB user/password from environment variables; app starts and connects.
  - Learn: JDBC URL, `spring.jpa.hibernate.ddl-auto=update`, `spring.jpa.show-sql=true`.
  - Done when: both of you run the app locally with no errors.
- [ ] **0.4 (A)** Add `application-test.properties` with H2 and a test `CinemaHubApplicationTests` that loads the context.

## Phase 1 – Entities + repositories (28 Nov – 6 Dec)
**A**
- [ ] **A1** `Genre` entity (id, name unique) + `GenreRepository`.
  - Learn: `@Entity`, `@Id`, `@GeneratedValue`, `JpaRepository`.
- [ ] **A2** `Movie` entity (title, description, durationMin, releaseYear) + `MovieRepository`.
- [ ] **A3** Many-to-Many Movie <-> Genre.
  - Learn: `@ManyToMany`, `@JoinTable`, owning side vs. `mappedBy`, why `Set` not `List`.
  - Done when: Hibernate creates `movie_genre` table; a small test saves a movie with 2 genres.
- [ ] **A4** `Customer` entity (name, email unique) + repository.

**B**
- [ ] **B1** `Hall` entity (name, rows, seatsPerRow) + repository.
- [ ] **B2** `Seat` entity (rowNo, seatNo) Many-to-One -> Hall.
  - Learn: `@ManyToOne`, `@JoinColumn`, `fetch = LAZY`, unique constraint (hall, row, seat).
- [ ] **B3** *(after A2 is merged)* `Screening` entity (startTime, price) Many-to-One -> Movie and -> Hall.

## Phase 2 – DTO, Mapper, Service, Controller for your entities (7 – 20 Dec, light work: Test 2 prep!)
Same pattern for each entity – learn it once with Genre/Hall, then repeat.
**A**
- [ ] **A5** Genre: `GenreRequest`/`GenreResponse` DTOs (Java `record`), mapper, `GenreService`, `GenreController` (GET all, GET by id, POST, PUT, DELETE).
  - Learn: why we never return entities directly, `@RestController`, `@RequestMapping`, `@PathVariable`, `@RequestBody`, `ResponseEntity`, HTTP status codes.
  - Done when: you can call all endpoints from Postman/curl.
- [ ] **A6** Movie CRUD (request carries `genreIds`, response shows genre names).
- [ ] **A7** Customer CRUD.

**B**
- [ ] **B4** Hall CRUD; creating a Hall also generates its Seats (rows x seatsPerRow).
- [ ] **B5** Screening CRUD (create validates movie + hall exist).

## Phase 3 – Business logic (21 Dec – 3 Jan)
**A**
- [ ] **A8** `Review` entity (stars 1–5, text) Many-to-One -> Movie and -> Customer; endpoints to add/list reviews of a movie.
- [ ] **A9** Custom `@Query` #1: top-rated movies (average stars, min N reviews). Also search movies by title/genre.
  - Learn: JPQL vs. SQL, `@Param`, `GROUP BY`, `AVG`.
- [ ] **A10** Validation: `@Valid`, `@NotBlank`, `@Size`, `@Min/@Max`, `@Email` on all your request DTOs.

**B**
- [ ] **B6** `Booking` entity (status CONFIRMED/CANCELLED, createdAt) -> Screening, Seat, Customer; **unique (screening, seat)**.
- [ ] **B7** `BookingService.book(...)` with `@Transactional`: reject if screening started, reject if seat taken (HTTP 409).
  - Learn: transactions, `DataIntegrityViolationException`, race conditions.
- [ ] **B8** Cancel booking (frees the seat).
- [ ] **B9** Custom `@Query` #2: free seats of a screening.
  - Learn: subquery / `NOT EXISTS` / LEFT JOIN.

## Phase 4 – Quality (4 – 12 Jan)
- [ ] **A11** `@RestControllerAdvice` `GlobalExceptionHandler`: 404 (not found), 400 (validation errors list), 409 (conflict), 500 (fallback); consistent JSON error body.
  - B then adds their own `SeatAlreadyBookedException` etc. in a small PR.
- [ ] **A12** Swagger/OpenAPI: springdoc dependency, `OpenApiConfig` (title/description), `@Operation` + `@Tag` on controllers.
- [ ] **A13** Unit tests (JUnit 5 + Mockito) for Movie and Review services.
- [ ] **B10** Unit tests for BookingService (happy path, seat taken, past screening).
- [ ] **B11** **Integration test**: `@SpringBootTest`, N threads (`ExecutorService` + `CountDownLatch`) book the same seat -> exactly 1 success, rest get conflict.
  - Learn: threads from the Test 2 lectures – this is the exact same material.
- [ ] **AB12** README (run, DB setup, Swagger link, business-logic explanation), ER diagram in `docs/`, `data.sql` sample data.
- [ ] **AB13** Final check: fresh clone -> `./mvnw test` passes -> app runs. Send link + faculty numbers.

## Suggested calendar
| Dates | A | B |
|---|---|---|
| until 27 Nov | 0.1, 0.3 only (study for Test 1) | 0.2, 0.3 |
| 28 Nov – 6 Dec | A1–A4 | B1–B3 |
| 7 – 20 Dec | A5 (then A6/A7 if time) | B4, B5 – mostly Test 2 prep |
| 21 Dec – 3 Jan | A8–A10 | B6–B9 |
| 4 – 12 Jan | A11–A13, AB12 | B10, B11, AB13 |
