# Software Requirements Specification (SRS / FRD)

**Project Name:** Test2 — College Library Book Issuing System
**System Under Specification:** College Library Book Issuing System (web-based)
**Source Template:** `templates/Software_Requirements_Specification_Template_FONT_SRS_v1_0.doc` (FONT SRS v1.0)
**Source BRD:** `docs/BRD_Test2.md` in the BRD Agent repository (College Library Book Issuing System)
**Issue Reference:** AGEN-8 ("Generate FRD for Test Project")
**Document Version:** 0.1 (Draft)
**Document Status:** Draft for review
**Prepared By:** FRD Agent

---

## 1. Introduction

### 1.1 Purpose

The purpose of this Software Requirements Specification (SRS/FRD) is to define the functional and
non-functional requirements for the **College Library Book Issuing System** — a web-based
application that allows a college library to issue books to students, record book returns, and
track due dates. The system is owned and hosted by the college on its own server.

This document is the basis for solution design, development and system acceptance testing. It
translates the business requirements captured in the Business Requirements Document
(`docs/BRD_Test2.md`, requirements `FR-001` to `FR-016` and `NFR-001` to `NFR-019`) into testable
system requirements. Each system requirement in this document carries a unique ID (`SR-xxx`) and,
where applicable, a traceability note back to the corresponding BRD requirement ID so that every
requirement can later be tested with a PASS/FAIL result against the Acceptance Test Plan.

### 1.2 Scope

This document covers a single application: an automated library book issuing system for a single
college campus. In scope are:

- Issuing books to students and recording the associated loan (student, book copy, issue date, due date).
- Returning books and closing the associated loan with the actual return date.
- Tracking due dates and identifying overdue loans.
- Maintaining the master data required for lending (books, students) and configuring the loan period.
- A limited, self-service (view-only) student view of a student's own loans and due dates.

Out of scope for this release (deferred unless explicitly agreed by the business):

- Book acquisitions, full cataloguing and shelf-management standards.
- Online catalogue browsing, reservations and self-service checkout kiosks.
- Notifications/reminders of due dates (candidate for a future release).
- Fines and payment processing for overdue books (candidate for a future release).
- Inter-library loans and multi-campus/institutional federations.

This SRS/FRD document itself does not include the Acceptance Test Plan (ATP); the ATP template is
referenced only to ensure requirement phrasing supports later PASS/FAIL testing.

### 1.3 Overview of the Document

This SRS is organised in accordance with the FONT SRS v1.0 template structure:

- Section 1 — Introduction (Purpose, Scope, Overview of the Document)
- Section 2 — System Context (Product Perspective, System Overview & Context Diagram, Operational
  Concepts and Scenarios, System Interfaces, Memory Constraints, Operations, Site Adaptation
  Requirements, Product Functions)
- Section 3 — Constraints (Constraints, Assumptions and Dependencies, Apportioning of Requirements)
- Section 4 — Specific Requirements (Organizational, External Interface, Functional
  Requirements/System Features with IDs `SR-001` to `SR-016`, Performance, Acceptance Criteria,
  Logical Database, Design Constraints, Testing, Compliance to Standards)
- Section 5 — Software System Attributes (Reliability, Availability, Safety, Environmental,
  Security, Maintainability, Portability, Installability, Usability, Other Requirements)
- Section 6 — Organizing Specific Requirements
- Section 7 — Validation
- Section 8 — References

---

## 2. System Context

### 2.1 Product Perspective

The College Library Book Issuing System is a **new build**; there is no existing system to
integrate with and no legacy data migration is required. It is a web-based application **hosted on
the college's own server** and used by the library lending desk and by students.

The system's context consists of:

1. **Librarians** — the operational users who perform issue and return transactions and monitor
   overdue loans.
2. **Students** — borrowers of books who view their own loans and due dates.
3. **College Library Management / Administration / IT** — set lending policy, approve scope, and
   host, support and maintain the system.
4. **College IT infrastructure** — the server and, where applicable, the college network used to
   host and reach the application.

The system is a self-contained application whose main internal components are: the core lending
functions (issue, return, due-date tracking) and the master-data functions (books, students, loan
period configuration). See 2.2.

### 2.2 System Overview & Context Diagram Description

The system is the central element of the context. Its immediate environment consists of:

- **Librarian users** — interact with the system through a web-based user interface to issue books,
  return books and view active/overdue loans.
- **Student users** — interact through the web-based user interface to view their own active loans
  and due dates (read-only).
- **College-hosted server** — hosts the web application and its database; data is stored and
  processed within the college's own infrastructure.

Data flow: a librarian selects a student and an available book copy → the system creates a loan
record with the issue date and a calculated due date → the loan is tracked as active → on return,
the system records the return date and closes the loan → the book copy becomes available again.
Overdue loans (active loans whose due date is earlier than today) are identified and listed for
follow-up.

Since this is a new build with no external systems, there are no external data sources or targets
in the context diagram; the college server, librarians and students make up the complete boundary.

### 2.3 Operational Concepts and Scenarios

#### 2.3.1 Scenario 1 — Issue a Book (Nominal)
A librarian selects a registered student and an available book copy. The system verifies that the
book copy is not on an active loan, creates a loan record containing the student, the book copy,
the issue date (today) and the calculated due date, and displays a confirmation showing the
student, book and due date. The book copy's status becomes "on loan/not available".

#### 2.3.2 Scenario 2 — Issue Attempt for an Unavailable Book
A librarian attempts to issue a book copy that is already on an active loan. The system rejects
the attempt with a clear message (e.g. "book already on loan") and does not create a loan record.

#### 2.3.3 Scenario 3 — Return a Book (Nominal)
A librarian records the return of an issued book. The system closes the associated active loan,
records the actual return date, and sets the book copy back to available so it can be issued again.

#### 2.3.4 Scenario 4 — Overdue Identification
A librarian opens the overdue-loans view. The system lists all active loans whose due date is
earlier than today, so the library can follow up on books not returned on time. When a book that
is being returned was overdue (return date later than due date), the system indicates this to the
librarian.

#### 2.3.5 Scenario 5 — Student Self-Service View
A student logs in and views his/her own active loans and due dates. The student can see no other
student's loan data and cannot perform lending functions.

#### 2.3.6 Operating Hours Scenario
The system is expected to be available and in normal operation during library opening hours:
**9:00 AM to 6:00 PM, Monday to Saturday**. Any requirement beyond this timetable (e.g. after-hours
read-only access) is To be defined.

### 2.4 System Interfaces

#### 2.4.1 User Interfaces

- **UI-001** — A web-based user interface for **Librarians** supporting: issue a book, return a
  book, view all active loans (with due dates), view the overdue-loans list, and maintain student
  and book master data, in no more than a small, defined number of steps (target to be confirmed).
  Each issue transaction shall display a confirmation of the student, book and due date.
  Validation failures (e.g. book already on loan) shall be shown as clear, understandable messages.
- **UI-002** — A web-based, **read-only self-service view for Students** showing only the
  student's own active loans and their due dates.
- Detailed screen layouts, look-and-feel and presentation standards: **To be defined**.

#### 2.4.2 Hardware Interfaces

- **HID-001** — The system shall run on the college's own server (web application server and
  database server). Specific server hardware/OS specifications: **To be defined**.
- **HID-002** — Optional (candidate idea, TBD with business): barcode scanning of book copies and
  student ID cards to speed up issue/return data entry. **To be defined** / Not applicable unless
  agreed.

#### 2.4.3 Software Interfaces

- **SID-001** — The system shall be implemented as a web application accessible through a standard
  web browser on the college network. Technology stack (application framework, web server,
  database platform): **To be defined**.
- **SID-002** — No integration with external systems is required for this release (new build, no
  existing system to integrate with). Any future integration with college student records, e-mail
  or identity systems: **To be defined** (see BRD NFR-012).
- **SID-003** — If barcode capture is agreed (HID-002), the associated scanner driver/hardware
  interface: **To be defined**.

#### 2.4.4 Communication Interfaces

- **CID-001** — Client–server communication between the web browser and the web application shall
  use standard web protocols (HTTP/HTTPS) over the college network. TLS/HTTPS use for all traffic:
  recommended, specific policy **To be defined** by college IT.
- **CID-002** — No machine-to-machine communication interfaces are required for this release.
  Security policy for network access (firewall, VPN): **To be defined**.

### 2.5 Memory Constraints

- **MEM-001** — At expected volumes (~5,000 book copies, ~2,000 students, ~50 issue/return
  transactions per day), the database and application shall operate within the capacity of the
  college-hosted server. Specific database size, memory and storage requirements: **To be defined**
  once the technology stack is confirmed.

### 2.6 Operations

- **OPR-001** — The system shall be operated and supported by the college (or its IT function).
  Operational procedures for start/stop, monitoring and support: **To be defined**.
- **OPR-002** — Scheduled backups of loan data shall be performed so that no completed issue or
  return is lost following a failure (see SR-026, traced to BRD NFR-015).

### 2.7 Site Adaptation Requirements

- **SAD-001** — The system is deployed at a single site (the college) and is not expected to
  require installation at multiple sites. Adaptation for additional sites, languages, date formats
  or locales: **Not applicable** for this release (single-locale college environment).

### 2.8 Product Functions

The product provides the following high-level functions:

- **F-01** Issue a book to a student, creating a tracked loan with a calculated due date.
- **F-02** Return a book, closing the loan and making the copy available again, and flagging
  overdue returns.
- **F-03** Track due dates, list all active loans, and identify overdue loans.
- **F-04** Maintain student and book master data, and configure the loan period.
- **F-05** Provide a read-only student view of a student's own loans and due dates.

Detailed requirements for these functions are specified in Section 4.5, each traced to the
corresponding BRD requirement.

---

## 3. Constraints

### 3.1 Constraints

- **Con-001** — The system shall be a web-based application hosted on the college's own server.
- **Con-002** — This is a new build; there is no existing system to integrate with, so the system
  shall not depend on any external or legacy system to deliver its core functions.
- **Con-003** — The system shall operate within library opening hours: 9:00 AM to 6:00 PM, Monday
  to Saturday.
- **Con-004** — Budget and delivery timeframes: **To be defined** by the project sponsor.
- **Con-005** — Technology platform: **To be defined** (only the web-based deployment model is
  fixed by the business).
- **Con-006** — The system shall comply with the college's library policy (to be confirmed) and
  applicable data-protection obligations related to student personal data.

### 3.2 Assumptions and Dependencies

Assumptions:

- **Asm-001** — A student is uniquely identifiable within the college (e.g. by student ID).
- **Asm-002** — A book copy is uniquely identifiable within the library (e.g. by accession/barcode
  number).
- **Asm-003** — The library collection and borrower base already exist; since this is a new build,
  there is no backlog of existing loans to migrate (any pre-existing informal loan registers will be
  re-recorded at cutover if the library requires it — To be confirmed).
- **Asm-004** — Only issued books can be returned; only available book copies can be issued.
- **Asm-005** — A single book copy can be on loan to only one student at a time.

Dependencies:

- **Dep-001** — Confirmation of the college library lending rules (loan period, renewals, fines):
  **To be defined**. The loan period is required to set the default due-date calculation.
- **Dep-002** — Availability of student identity data or a mechanism to register students:
  **To be defined**.
- **Dep-003** — Confirmation of hosting, infrastructure and support arrangements within the college:
  **To be defined**.

### 3.3 Apportioning of Requirements

- **APP-001** — The core lending functions (issue, return, due-date tracking) and the master-data
  functions required to operate them are mandatory for the initial release and cannot be deferred.
- **APP-002** — The student self-service view (SR-013) is a "Should Have" per the BRD; it may be
  phased within the initial delivery if resourcing requires, but its absence must not affect
  librarian lending operations.
- **APP-003** — All requirements marked "Must Have" in Section 4 are hard constraints for release
  acceptance. Requirements marked "Should Have" are included in the baseline but may be scheduled
  in a later increment with business agreement.

---

## 4. Specific Requirements

### 4.1 Organizational Requirements

#### 4.1.1 Business Requirements

These are the business-level requirements carried forward from the BRD (Section 5.7 of
`docs/BRD_Test2.md`). The system shall enable the library to record the issue and return of books,
track due dates, and identify overdue loans, so that the lending operation is accurate and
traceable. Detailed traceable system requirements are specified in Section 4.5.

#### 4.1.2 User Requirements

- **User Req-001** — Librarians shall be able to perform issue, return and overdue-tracking
  functions quickly and accurately with minimal training.
- **User Req-002** — Students shall be able to view their own active loans and due dates without
  interfering with library operations.

### 4.2 External Interface Requirements

The system shall provide:

- **Ext-001** — A web browser–based interface for librarians (issue, return, active/overdue loan
  views, master-data maintenance). Interface details: **To be defined**.
- **Ext-002** — A web browser–based, read-only interface for students limited to their own loans.
  Interface details: **To be defined**.
- **Ext-003** — No interfaces to external or third-party systems are required for this release.

### 4.3 Functional Requirements / System Features

Each requirement is a single, testable statement with a unique ID (`SR-xxx`), a priority, an
acceptance criterion expressed for PASS/FAIL testing, and a traceability note back to the BRD
requirement ID. Functional requirements `SR-001` to `SR-016` trace directly to BRD `FR-001` to
`FR-016`. Requirements traced to BRD `NFR-xxx` IDs appear in Sections 4.4, 4.5 (attributes) and 5,
as appropriate.

#### 4.3.1 Core Function — Issue a Book

| ID | Requirement | Priority | BRD Trace |
| --- | --- | --- | --- |
| SR-001 | The system shall allow a librarian to issue a book to a student by selecting a student and a book copy. | Must Have | FR-001 |
| SR-002 | The system shall create a loan record on issue, capturing the student, book copy, issue date and calculated due date. | Must Have | FR-002 |
| SR-003 | The system shall prevent issuing a book copy that is already on loan to another student, or otherwise unavailable, and shall display a clear message. | Must Have | FR-003 |
| SR-004 | The system shall calculate and record the due date on issue, based on the configured loan period. | Must Have | FR-004 |
| SR-005 | The system shall provide confirmation of the completed issue, including the student, book and due date. | Should Have | FR-005 |

Acceptance criteria (testable PASS/FAIL):

- **SR-001 — PASS** if a librarian can complete an issue transaction by selecting any registered
  student and any available book copy; otherwise **FAIL**.
- **SR-002 — PASS** if, after a successful issue, a loan record exists containing the student ID,
  book copy ID, issue date (today) and a calculated due date; otherwise **FAIL**.
- **SR-003 — PASS** if the system rejects an issue attempt where the chosen book copy is already on
  an active loan (or otherwise unavailable) and provides a clear message; otherwise **FAIL**.
- **SR-004 — PASS** if the due date equals the issue date plus the configured loan period and is
  stored on the loan; otherwise **FAIL**.
- **SR-005 — PASS** if the UI displays a confirmation with the student, book and due date after
  each issue; otherwise **FAIL**.

#### 4.3.2 Core Function — Return a Book

| ID | Requirement | Priority | BRD Trace |
| --- | --- | --- | --- |
| SR-006 | The system shall allow a librarian to record the return of an issued book. | Must Have | FR-006 |
| SR-007 | The system shall close the associated loan on return, recording the actual return date. | Must Have | FR-007 |
| SR-008 | The system shall set the book copy back to available status once its loan is closed. | Must Have | FR-008 |
| SR-009 | The system shall clearly indicate when a returned book was overdue (return date later than due date). | Should Have | FR-009 |

Acceptance criteria (testable PASS/FAIL):

- **SR-006 — PASS** if a librarian can complete a return transaction for any loan that is currently
  active; otherwise **FAIL**.
- **SR-007 — PASS** if, after a return, the loan shows the actual return date and is no longer
  active; otherwise **FAIL**.
- **SR-008 — PASS** if the book copy can immediately be issued again after return; otherwise
  **FAIL**.
- **SR-009 — PASS** if the system flags/indicates overdue status for a returned loan whose return
  date is later than its due date; otherwise **FAIL**.

#### 4.3.3 Core Function — Track Due Dates / Overdue Loans

| ID | Requirement | Priority | BRD Trace |
| --- | --- | --- | --- |
| SR-010 | The system shall store and display the due date for every active loan. | Must Have | FR-010 |
| SR-011 | The system shall provide a view/report of all active loans, including their due dates. | Must Have | FR-011 |
| SR-012 | The system shall identify and list overdue loans (due date earlier than today and loan still active). | Must Have | FR-012 |
| SR-013 | The system shall allow a student to view only their own active loans and their due dates. | Should Have | FR-013 |

Acceptance criteria (testable PASS/FAIL):

- **SR-010 — PASS** if the due date shown on each active loan matches the date calculated at issue;
  otherwise **FAIL**.
- **SR-011 — PASS** if a librarian can retrieve a list of all active loans with due dates from the
  system; otherwise **FAIL**.
- **SR-012 — PASS** if a librarian can retrieve a list containing exactly the active loans whose due
  date is earlier than today; otherwise **FAIL**.
- **SR-013 — PASS** if a student is able to view only their own active loans and due dates, and
  cannot see other students' loans; otherwise **FAIL**.

#### 4.3.4 Core Function — Maintain Master Data

| ID | Requirement | Priority | BRD Trace |
| --- | --- | --- | --- |
| SR-014 | The system shall allow registered students to be added, updated and identified for lending. | Must Have | FR-014 |
| SR-015 | The system shall allow book copies to be added, updated and identified for lending. | Must Have | FR-015 |
| SR-016 | The system shall allow the loan period (used for due-date calculation) to be configured by an authorised administrator. | Should Have | FR-016 |

Acceptance criteria (testable PASS/FAIL):

- **SR-014 — PASS** if a student record can be created/updated and is then selectable in an issue
  transaction; otherwise **FAIL**.
- **SR-015 — PASS** if a book copy record can be created/updated and is then selectable in an issue
  transaction; otherwise **FAIL**.
- **SR-016 — PASS** if changing the configured loan period changes the due date calculated for
  subsequent issues; otherwise **FAIL**.

**Traceability summary (SR ↔ FR):**

| SR | SR-001 | SR-002 | SR-003 | SR-004 | SR-005 | SR-006 | SR-007 | SR-008 | SR-009 | SR-010 | SR-011 | SR-012 | SR-013 | SR-014 | SR-015 | SR-016 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FR | FR-001 | FR-002 | FR-003 | FR-004 | FR-005 | FR-006 | FR-007 | FR-008 | FR-009 | FR-010 | FR-011 | FR-012 | FR-013 | FR-014 | FR-015 | FR-016 |

### 4.4 Performance Requirements

- **SRR-P01** — At the expected transaction volume of approximately **50 issue/return transactions
  per day** and with an initial dataset of approximately **5,000 books** and **2,000 students**, the
  system shall complete routine issue and return transactions without perceptible delay on the
  operational network (target indicative response time, to be confirmed with the business).
  Traced to BRD NFR-013.
- **SRR-P02** — The system shall handle the expected volumes (5,000 books, 2,000 students,
  ~50 transactions/day) within the capacity of the college-hosted server without degradation of the
  core functions. Traced to BRD NFR-017.
- **SRR-P03** — Concurrent-user and exact response-time targets: **To be defined**; the volumes
  above imply a small number of concurrent operator users at the lending desk.

Acceptance: **PASS** if routine transactions complete within the confirmed limits at the confirmed
volumes; otherwise **FAIL**.

### 4.5 Acceptance Criteria

The system is accepted when each requirement in Sections 4.3, 4.4 and 5 passes its individual
PASS/FAIL acceptance criterion, verified through the Acceptance Test Plan, and when the
organizational acceptance (Section 6.6) is satisfied by the college. Every requirement carries its
own testable criterion; no functional requirement marked "Must Have" may fail at acceptance.

### 4.6 Logical Database Requirements

The system shall maintain the following core entities, derived from the BRD (business objects):

| Entity | Key Attributes (initial) | Notes |
| --- | --- | --- |
| Student | Student ID (unique), name, department, contact details | Identifies the borrower |
| Book (copy) | Accession/barcode number (unique), title, author, availability status | Identifies a borrowable copy |
| Loan | Loan ID, student, book (copy), issue date, due date, return date | Created at issue, closed at return |
| Librarian | Librarian ID/user account, name, role | Identifies who performs actions (see SR-024) |

Relationships: a Student may hold many Loans; a Book (copy) is held by at most one active Loan at a
time; a Loan refers to exactly one Student and one Book; each issue/return action is attributable
to a Librarian user account. A data model will be confirmed during design. Logical database
design, indexing and retention/archiving rules: **To be defined**.

### 4.7 Design Constraints

- **Des-001** — The system shall be built as a web application and deployed on the college's own
  server (Con-001). All other technology choices (platform, language, database) are **To be
  defined**.
- **Des-002** — Date and time handling shall support the college's local timezone and single
  locale; date arithmetic for due dates shall be calendar-day based as per the configured loan
  period (BRD assumptions).
- **Des-003** — The due-date calculation shall be driven by a configurable loan period (SR-016) and
  must not require code change to alter (business rule externalised).

### 4.8 Testing Requirements

- **Tst-001** — Each requirement in this SRS shall be verifiable; verification methods and test
  cases shall be recorded in the Acceptance Test Plan with traceability back to the `SR-xxx` and
  `FR-xxx`/`NFR-xxx` IDs.
- **Tst-002** — Test coverage shall include: issue and return transactions (valid and invalid
  paths), due-date calculation for configured loan periods, overdue-loan identification, student
  data isolation, and master-data maintenance.
- **Tst-003** — The college shall perform user acceptance testing (UAT) covering the core lending
  functions (SR-001 to SR-016), with librarians and students as the representative user groups.
- **Tst-004** — Testing shall be performed against the expected volumes (5,000 books, 2,000
  students, ~50 transactions/day) to validate the performance requirements in Section 4.4.

### 4.9 Compliance to Standards

- **Std-001** — The system shall comply with the college's library policy (to be confirmed) and
  applicable data-protection obligations relating to student personal data (see SR-023, traced to
  BRD NFR-005).
- **Std-002** — Web application accessibility and conformance standards (e.g. W3C/A-grade
  accessibility): **To be defined**.
- **Std-003** — Organisation-specific development and release standards of the implementing
  vendor: **To be defined**.

---

## 5. Software System Attributes

Requirements in this section are traced to the BRD `NFR-xxx` requirements where applicable.

### 5.1 Reliability

- **SR-025** — Loan data shall be backed up and recoverable so that no completed issue or return is
  lost following a failure. Priority: Must Have. Traced to BRD NFR-015.
  - Acceptance: **PASS** if a recovered backup retains all completed transactions (issues and
    returns); otherwise **FAIL**.
- **SR-026** — The system shall not lose active loan records on application or server restart, and
  shall not permit double-issue of a book copy due to concurrent actions. Priority: Must Have.
  Traced to BRD NFR-015 / FR-003.
  - Acceptance: **PASS** if active loans persist across a restart and no double-issue can be created
    for the same copy; otherwise **FAIL**.

### 5.2 Availability

- **SR-027** — The system shall be available during library opening hours: **9:00 AM to 6:00 PM,
  Monday to Saturday**. Exact uptime target and maintenance windows beyond this: **To be defined**.
  Priority: Must Have. Traced to BRD NFR-014.
  - Acceptance: **PASS** if the system is available for the agreed opening hours over the agreed
    measurement period; otherwise **FAIL**.

### 5.3 Safety

- **SR-028** — No specific safety requirement is identified for this administrative (non-physical)
  software system. Standard office ergonomics apply; no additional statutory health-and-safety
  obligations exist beyond those applying to routine administrative work. Priority: Not applicable.
  Traced to BRD NFR-010.

### 5.4 Environmental

- **SR-029** — No specific environmental requirement is identified beyond normal operation of the
  college-hosted server (power, cooling, physical security) as provided by college IT. Priority:
  Not applicable. Traced to BRD NFR-010.

### 5.5 Security

Input data validation:

- **SR-030** — The system shall validate all input used to create or update student, book and loan
  records (required fields, identifier uniqueness, valid-date checks) and shall display clear
  validation messages for invalid input. Priority: Must Have. Traced to BRD NFR-009 / FR-003.
  - Acceptance: **PASS** if invalid input is rejected with a clear message and no partial/invalid
    record is saved; otherwise **FAIL**.

Control of internal processing:

- **SR-031** — The system shall restrict issue and return functions to authorised **librarian**
  accounts; students shall be denied access to these functions. Priority: Must Have. Traced to BRD
  NFR-003.
  - Acceptance: **PASS** if an unauthorised (non-librarian) account cannot perform librarian
    functions; otherwise **FAIL**.
- **SR-032** — The system shall restrict **student** accounts to viewing only their own loans and
  due dates and shall prevent a student from viewing another student's loans. Priority: Must Have.
  Traced to BRD NFR-004.
  - Acceptance: **PASS** if a student cannot view another student's loan data by any means exposed
    by the system; otherwise **FAIL**.
- **SR-033** — The system shall record, for each issue and return, **who** performed the action and
  **when** (audit trail). Priority: Must Have. Traced to BRD NFR-001.
  - Acceptance: **PASS** if every issue/return is traceable to a user ID and timestamp; otherwise
    **FAIL**.
- **SR-034** — The system shall record changes to student and book master data (who and when).
  Priority: Should Have. Traced to BRD NFR-002.
  - Acceptance: **PASS** if each master-data change is traceable to a user ID and timestamp;
    otherwise **FAIL**.

Output data validation:

- **SR-035** — System outputs (confirmations, active-loan lists, overdue-loan lists and the student
  self-service view) shall contain only data consistent with the underlying loans, with no
  student-identifiable data exposed beyond the intended recipient. Priority: Must Have. Traced to
  BRD NFR-004 / NFR-005.
  - Acceptance: **PASS** if outputs are consistent with stored loan data and no student sees data
    beyond their own loans; otherwise **FAIL**.

Privacy and confidentiality:

- **SR-036** — Student personal data shall be handled in line with applicable data-protection
  requirements and college policy. Priority: Must Have. Traced to BRD NFR-005.
  - Acceptance: **PASS** if handling of student data complies with the confirmed college
    data-protection policy; otherwise **FAIL**.

### 5.6 Maintainability

- **SR-037** — The system shall be built so that the configurable loan period and other business
  rules (due-date calculation) can be maintained without code changes, and so that the application
  can be maintained and supported by the college/vendor. Specific maintainability standards
  (code/design reviews, modularity): **To be defined**. Priority: Should Have.
  - Acceptance: **PASS** if configuration changes and routine maintenance can be performed without
    code changes and per the agreed maintenance procedure; otherwise **FAIL**.

### 5.7 Portability

- **SR-038** — The system shall run on the server infrastructure chosen by the college; no
  multi-platform portability is required for this single-site, single-locale deployment. Any
  portability constraints arising from the eventual technology choice: **To be defined**. Priority:
  Not applicable / To be defined.

### 5.8 Installability

- **SR-039** — The system shall be installable on the college-hosted server following a defined
  installation/release procedure, deliverable as user and technical documentation. Priority:
  Should Have. Traced to BRD NFR-016.
  - Acceptance: **PASS** if installation and upgrade can be completed by following the delivered
    documentation; otherwise **FAIL**.

### 5.9 Usability

- **SR-040** — The core issue and return operations shall be completable in a small, defined number
  of steps (target to be confirmed in UAT). Priority: Should Have. Traced to BRD NFR-008.
  - Acceptance: **PASS** if usability targets are met in user acceptance testing; otherwise **FAIL**.
- **SR-041** — The system shall display clear, understandable messages for validation failures (e.g.
  book already on loan). Priority: Should Have. Traced to BRD NFR-009.
  - Acceptance: **PASS** if the clear-message criteria are met in UAT; otherwise **FAIL**.

### 5.10 Other Requirements

- **SR-042** — User and technical documentation for the system shall be provided (operating and user
  guides). Priority: Should Have. Traced to BRD NFR-016.
  - Acceptance: **PASS** if operating and user guides are delivered and accepted; otherwise **FAIL**.
- **SR-043** — Librarians shall receive training covering issue, return and overdue tracking before
  go-live, and students shall receive brief guidance on viewing their own loans and due dates.
  Priority: Must Have (librarians) / Should Have (students). Traced to BRD NFR-018 / NFR-019.
  - Acceptance: **PASS** if all librarians complete training before go-live and can execute the core
    functions unaided; otherwise **FAIL**.
- **SR-044** — The student interface (access channel, authentication mechanism, portal/intranet
  placement) used to reach the self-service view: **To be defined** with the college. Traced to BRD
  NFR-007.

---

## 6. Organizing Specific Requirements

### 6.1 System Mode

The system operates in a single normal operational mode (lending desk in daily operation) with no
defined alternative system modes (e.g. training or maintenance modes). Additional modes are
**To be defined**.

### 6.2 User Class

| User Class | Description | Functions Available |
| --- | --- | --- |
| Librarian | Library staff operating the lending desk | Issue (SR-001 to SR-005), Return (SR-006 to SR-009), active/overdue loans (SR-010 to SR-012), master data (SR-014 to SR-016) |
| Student | Borrowers of books | View own active loans and due dates (SR-013) only; read-only |
| Administrator (authorised) | Configure loan period (SR-016); likely a librarian/IT role — **To be confirmed** | Configuration function (SR-016) and, as applicable, loan-period administration |

### 6.3 Objects

The principal system objects are **Student**, **Book (copy)**, **Loan** and **Librarian/user
account**, as defined in Section 4.6 (Logical Database Requirements). Each object carries the
attributes and relationships defined there.

### 6.4 Features

The system's features are grouped as follows:

1. Issue a book (SR-001 to SR-005).
2. Return a book (SR-006 to SR-009).
3. Track due dates / identify overdue loans (SR-010 to SR-012).
4. Student self-service view (SR-013).
5. Maintain master data (SR-014 to SR-016).
6. Audit, security and data protection (SR-030 to SR-036 and Section 5).

### 6.5 Implementation Schedule (Optional)

A single-phase delivery of features 1–3 (core lending) plus feature 5 (master data) is the
recommended baseline. Feature 4 (student view, SR-013) is "Should Have" and may be scheduled as an
early increment. Detailed implementation schedule and phased plan: **To be defined**.

### 6.6 Acceptance

Acceptance of the system is confirmed when:

- All "Must Have" functional requirements (SR-001 to SR-004, SR-006 to SR-008, SR-010 to SR-012,
  SR-014, SR-015) pass their acceptance criteria in testing/UAT.
- "Should Have" requirements (SR-005, SR-009, SR-013, SR-016, SR-040 to SR-044) pass or are
  formally deferred with business agreement.
- Security and data-protection requirements (SR-030 to SR-036) pass.
- The college library signs off the acceptance (business-level acceptance criteria per BRD
  Section 5.7).

### 6.7 Derived Requirements

- **DR-001** — The system shall record, for every issue and return, the acting librarian and
  timestamp (derived from FR-001/FR-006 and NFR-001); specified as SR-033.
- **DR-002** — The system shall prevent the same book copy appearing in two active loans (derived
  from FR-003 and the single-copy-on-loan business rule); specified as SR-003.
- **DR-003** — The system shall treat a loan as closed once a return date is recorded (derived from
  FR-007), which in turn drives availability (FR-008) and active-loan reporting (FR-011); specified
  as SR-007/SR-008/SR-011.

---

## 7. Validation

### 7.1 Validation Strategy

The strategy validates that the delivered system meets the business intent captured in the BRD.
Validation consists of:

1. Requirements verification — each SR requirement is checked against its BRD trace and
   its PASS/FAIL criterion.
2. User acceptance testing (UAT) — librarians and students exercise the live system against the
   scenarios in Section 2.3.
3. Confirmation of the documented open questions/assumptions by the college before validation
   completes.

### 7.2 Validation Criteria

The system is validated when all "Must Have" requirements pass, the overdue and due-date tracking
behaves correctly against configured loan periods, student data isolation is confirmed, and the
college (library management) signs off acceptance as defined in Section 6.6.

### 7.3 Validation Constraints

- Validation is constrained by the availability of confirmed inputs (lending policy/loan period,
  student identity source, hosting/support arrangements — see Open Questions).
- Validation should be performed during library operating hours against representative data.
- Exact operational uptime targets and measurement windows must be confirmed before validation.

---

## 8. References

| Reference | Description |
| --- | --- |
| `docs/BRD_Test2.md` (BRD Agent repository) | Business Requirements Document — Test2 Project (College Library Book Issuing System); source of requirements `FR-001` to `FR-016` and `NFR-001` to `NFR-019` |
| `templates/Software_Requirements_Specification_Template_FONT_SRS_v1_0.doc` | Standard Software Requirements Specification template used as the basis for this document |
| `templates/Acceptance_Test_Plan_FONT_ATP_v1_0.doc` | Standard Acceptance Test Plan template; used as reference for PASS/FAIL traceability of requirements |
| AGEN-8 (this issue) | Task description: system details (roles, entities, hosting, volumes, operating hours) |

---

## Appendix — Open Questions (Requiring Human Input)

The following items need human input to complete this SRS/FRD:

1. College library lending policy — loan period/duration, renewals and fines for overdue books
   (affects SR-004/SR-016 default due-date calculation).
2. Source of student identity data and the process for registering students (affects SR-014).
3. Confirmation that there is no existing loan backlog to record at go-live (new build assumption).
4. Hosting, infrastructure and support arrangements within the college (technology stack, server,
   database, monitoring).
5. Technology platform and web framework for the web application.
6. Recommended/required use of HTTPS and authentication mechanism (login credentials, student
   access channel) — affects SR-031/SR-032/SR-044.
7. Exact performance targets (response times, concurrent users).
8. Document owner, document date, and named stakeholder representatives for review.
9. Whether the student self-service view (SR-013) is required in the first release or may be
   phased (Should Have per BRD).
10. Accessibility conformance standards (Std-002) and any college signage/policy obligations.