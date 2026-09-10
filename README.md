# Student Attendance Management System

A web-based Student Attendance Management System for **Precious Blood Kilungu
Secondary School**, built in Java with **Spring Boot**, **Spring Security**,
**Spring Data JPA**, **Thymeleaf**, and **MySQL** — matching the three-tier
architecture and database design described in the project's design document.

## Features

- **Role-based access control** — Admin, Lecturer, and Student each get a
  separate, protected area of the app (`/admin/**`, `/lecturer/**`,
  `/student/**`), enforced by Spring Security.
- **Admin**: register/edit/delete students and lecturers, create courses and
  assign a lecturer to each, enroll students in courses, reset passwords,
  view an institution-wide attendance report.
- **Lecturer**: see assigned courses, take attendance for any date (marks are
  editable — resubmitting the same date updates the existing records), view
  full attendance history per course.
- **Student**: see overall and per-course attendance percentage, view full
  attendance history, update contact details, change password.
- Passwords are hashed with **BCrypt**; nothing is ever stored in plain text.

## Tech stack

| Layer       | Technology                                   |
|-------------|-----------------------------------------------|
| Front-end   | Thymeleaf + Bootstrap 5                       |
| Back-end    | Java 17, Spring Boot 3.3, Spring MVC          |
| Security    | Spring Security (form login, RBAC, BCrypt)    |
| Persistence | Spring Data JPA / Hibernate                   |
| Database    | MySQL (production) / H2 in-memory (dev/demo)  |
| Build       | Maven                                         |

## Quick start (no database setup needed)

The app ships pre-configured to run on an **in-memory H2 database**, seeded
with demo data, so you can try it immediately:

```bash
mvn spring-boot:run
```

Then open **http://localhost:8080**. Demo accounts (seeded in `data.sql`):

| Role     | Username        | Password            |
|----------|-----------------|---------------------|
| Admin    | `Victoradmin`   | `victoradmin123`    |
| Lecturer | `rmutheu`       | `lecturerRaven123`  |
| Student  | `Mutisiya`      | `student123`        |

The H2 database resets every time the app restarts (it's in-memory) — useful
for demos, not for real data. To inspect it directly while running, visit
`http://localhost:8080/h2-console` (JDBC URL: `jdbc:h2:mem:attendancedb`,
user `sa`, empty password).

## Running with MySQL (production)

1. Create the database:
   ```sql
   CREATE DATABASE attendance_db;
   ```
2. Edit `src/main/resources/application-mysql.properties` and set your MySQL
   username/password.
3. Run with the `mysql` profile:
   ```bash
   mvn spring-boot:run -Dspring-boot.run.profiles=mysql
   ```
   or, after packaging:
   ```bash
   mvn clean package
   java -jar target/attendance-management-system-1.0.0.jar --spring.profiles.active=mysql
   ```

Hibernate will auto-create the schema (`ddl-auto=update`) on first run, and
`data.sql` will seed the same demo accounts as above.

## Opening in an IDE (NetBeans / IntelliJ / Eclipse)

This is a standard Maven project — in any of these IDEs, choose
**File → Open Project** (or **Import → Maven Project**) and point it at this
folder. The IDE will download dependencies and let you run
`AttendanceManagementSystemApplication.main()` directly.

## Project structure

```
src/main/java/com/school/attendance/
├── AttendanceManagementSystemApplication.java   # entry point
├── config/            # Spring Security config, login success handler
├── model/             # JPA entities: User, Student, Lecturer, Course, Attendance
├── repository/        # Spring Data JPA repositories
├── service/           # business logic (registration, attendance %, etc.)
└── controller/         # AuthController, AdminController, LecturerController, StudentController

src/main/resources/
├── application.properties          # default (H2) config
├── application-mysql.properties    # MySQL profile
├── data.sql                        # demo seed data
├── static/css/style.css
└── templates/                      # Thymeleaf views (login, admin/, lecturer/, student/)
```

## Database design

Matches Chapter 8 of the design document:

- **users** — login credentials + role (ADMIN / LECTURER / STUDENT)
- **students** — profile, linked 1:1 to a `users` row, many-to-many with `courses`
- **lecturers** — profile, linked 1:1 to a `users` row, one-to-many with `courses`
- **courses** — code, name, assigned lecturer
- **attendance** — one row per (student, course, date), with status
  (PRESENT / ABSENT / LATE / EXCUSED)

## Notes on scope

This build implements the core modules from the design document: authentication,
attendance recording/editing, student & lecturer management, course
management, and reporting. A few items from the fuller design doc — PDF/Excel
export, email/SMS notifications, and database backup/restore tooling — are
natural next additions but are not included in this first build, so the
codebase stays focused and easy to extend.

## Security notes for production

Before deploying for real use:
- Change all demo passwords, or remove `data.sql` entirely.
- Set `spring.jpa.hibernate.ddl-auto=validate` once your schema is stable
  (keep `update` only during development).
- Serve over HTTPS and set `server.servlet.session.cookie.secure=true`.
- Store the MySQL password via an environment variable rather than committing
  it in `application-mysql.properties`.
