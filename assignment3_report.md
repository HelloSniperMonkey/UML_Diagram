# Assignment III: OOAD with UML for HEI Registration and CGPA System

**Student:** Soumyajyoti Mohanta  
**Roll No.:** 2301AI23  
**Course:** CS 3205 OOP  
**Topic:** UML-based analysis and design for a Higher Education Institute (HEI)

## 1. Problem Statement

The system models a Higher Education Institute such as an IIT, NIT, or Central University. The main requirements are:

- represent the organization with minimal details,
- represent students of type `UG`, `PG`, and `PhD`,
- support course add/drop during semester registration,
- calculate CGPA at semester end,
- show the design using UML use case, class, and sequence diagrams.

The design follows the class notes: actors are placed outside the system boundary, use cases are placed inside the boundary, class diagrams show attributes, operations, multiplicity, and relationships, and sequence diagrams show object interaction over time.

> Note: the diagrams below use Mermaid for Markdown preview rendering. The editable UML source remains in the standalone `.puml` files.

## 2. Use Case Diagram

```mermaid
flowchart LR
    UG[UG Student] --> Student[Student]
    PG[PG Student] --> Student
    PHD[PhD Student] --> Student
    Registrar[Registrar]
    ExamCell[Exam Cell]

    subgraph HEI[HEI Registration and Result System]
        direction TB
        UC1([Login])
        UC2([View Course Catalog])
        UC3([Add Course])
        UC4([Drop Course])
        UC5([Submit Registration])
        UC6([Validate Registration Rules])
        UC7([Check Seat Availability])
        UC8([Approve Special Request])
        UC9([Publish Grades])
        UC10([Calculate CGPA])
        UC11([View Result / Transcript])
        UC12([Convert Letter Grade to Grade Point])
    end

    Student --- UC1
    Student --- UC2
    Student --- UC3
    Student --- UC4
    Student --- UC5
    Student --- UC11
    Registrar --- UC8
    ExamCell --- UC9
    ExamCell --- UC10

    UC3 -.->|<<include>>| UC6
    UC3 -.->|<<include>>| UC7
    UC4 -.->|<<include>>| UC6
    UC5 -.->|<<include>>| UC6
    UC10 -.->|<<include>>| UC12
    UC11 -.->|<<include>>| UC10
    UC8 -.->|<<extend>>| UC6
```

### Use case notes

- `Student` is a generalized actor, while `UG Student`, `PG Student`, and `PhD Student` are specialized actors.
- `Add Course`, `Drop Course`, and `Submit Registration` include `Validate Registration Rules`.
- `Approve Special Request` extends rule validation for exceptional cases such as overload or late registration.
- `View Result / Transcript` includes `Calculate CGPA`, because the final transcript view depends on computed grade points.

## 3. Class Diagram

```mermaid
classDiagram
    class Organization {
        +name: String
        +code: String
        +addStudent(s: Student)
        +offerCourse(c: Course)
    }

    class Student {
        <<abstract>>
        +studentId: String
        +name: String
        +email: String
        +cgpa: double
        +registerSemester(term: String)
        +viewTranscript()
        +getCGPA()
    }

    class UGStudent {
        +program: String
        +year: int
    }

    class PGStudent {
        +specialization: String
        +assistantship: boolean
    }

    class PhDStudent {
        +researchArea: String
        +supervisorName: String
    }

    class CGPACalculable {
        <<interface>>
        +calculateCGPA()
    }

    class Registration {
        +registrationId: String
        +term: String
        +status: String
        +addCourse(offering: CourseOffering)
        +dropCourse(offering: CourseOffering)
        +submit()
    }

    class Course {
        +courseCode: String
        +title: String
        +credits: int
    }

    class CourseOffering {
        +sectionId: String
        +semester: String
        +capacity: int
        +enrolledCount: int
        +hasSeat()
    }

    class Enrollment {
        +status: String
        +letterGrade: String
        +assignGrade(g: String)
    }

    class Transcript {
        +currentCGPA: double
        +calculateCGPA()
        +generateSummary()
    }

    class SemesterRecord {
        +term: String
        +sgpa: double
        +calculateSGPA()
    }

    class RegistrationPolicy {
        +maxCreditsUG: int
        +maxCreditsPG: int
        +maxCreditsPhD: int
        +validateAddDrop(s: Student, r: Registration, o: CourseOffering)
        +validateCreditLoad(s: Student, r: Registration)
    }

    class GradePointPolicy {
        +toPoint(letterGrade: String)
    }

    Student <|-- UGStudent
    Student <|-- PGStudent
    Student <|-- PhDStudent
    CGPACalculable <|.. Transcript

    Organization "1" o-- "1" RegistrationPolicy : owns
    Organization "1" o-- "1" GradePointPolicy : owns
    Organization "1" o-- "*" Student : has
    Organization "1" o-- "*" Course : offers
    Course "1" o-- "*" CourseOffering : schedules

    Student "1" --> "0..*" Registration : creates
    Student "1" --> "1" Transcript : owns
    Registration "1" *-- "0..*" Enrollment : contains
    Enrollment "*" --> "1" CourseOffering : for
    Transcript "1" *-- "1..*" SemesterRecord : contains
    SemesterRecord "1" *-- "1..*" Enrollment : summarizes

    Registration ..> RegistrationPolicy : uses
    Transcript ..> GradePointPolicy : uses
    CourseOffering --> Course : of
```

### Class diagram notes

- `Student` is an abstract superclass and `UGStudent`, `PGStudent`, and `PhDStudent` inherit from it.
- `Transcript` realizes the `CGPACalculable` interface.
- `Organization` aggregates students, courses, and policies.
- `Registration` composes `Enrollment` objects because enrollments are created for a specific registration context.
- `Transcript` composes `SemesterRecord` objects because semester records are part of one transcript.
- Multiplicity is explicitly shown to follow the lecture emphasis on associations and cardinality.

## 4. Sequence Diagram 1: Course Add / Drop during Registration

```mermaid
sequenceDiagram
    actor Student
    participant UI as Registration UI
    participant RC as Registration Controller
    participant REG as Registration
    participant CO as Course Offering
    participant RP as Registration Policy

    Student->>UI: selectAddOrDrop(courseCode, action)
    UI->>RC: processRequest(studentId, courseCode, action)
    RC->>REG: getActiveRegistration(studentId)
    REG-->>RC: registration
    RC->>CO: findOffering(courseCode)
    CO-->>RC: offering
    RC->>RP: validateAddDrop(student, registration, offering)
    RP-->>RC: valid / invalid

    alt valid request and seat available
        RC->>CO: hasSeat()
        CO-->>RC: true
        alt action = add
            RC->>REG: addCourse(offering)
            REG-->>RC: success
        else action = drop
            RC->>REG: dropCourse(offering)
            REG-->>RC: success
        end
        RC-->>UI: showUpdatedRegistration()
        UI-->>Student: registration updated
    else invalid request or no seat
        RC-->>UI: showError(reason)
        UI-->>Student: request rejected
    end
```

## 5. Sequence Diagram 2: Semester-End CGPA Calculation

```mermaid
sequenceDiagram
    actor EC as Exam Cell
    participant RS as Result Service
    participant TR as Transcript
    participant SR as Semester Record
    participant ENR as Enrollment
    participant GP as Grade Point Policy
    participant STU as Student

    EC->>RS: publishFinalGrades(studentId, term)
    RS->>TR: calculateCGPA()
    loop for each semester
        TR->>SR: calculateSGPA()
        loop for each enrollment
            SR->>ENR: getLetterGrade()
            ENR-->>SR: letterGrade
            SR->>GP: toPoint(letterGrade)
            GP-->>SR: gradePoint
        end
        SR-->>TR: sgpa and earnedCredits
    end
    TR->>STU: update cgpa
    STU-->>TR: acknowledged
    TR-->>RS: final CGPA
    RS-->>EC: transcript ready
```

## 6. Design Justification

### Merits

- The model separates static structure and dynamic behavior clearly.
- Inheritance is used naturally for `UG`, `PG`, and `PhD` students.
- Composition is used where lifecycle dependency exists, such as `Transcript` to `SemesterRecord`.
- The design is modular because policies for registration and grade-point conversion are separated from student data.
- The model supports extension for future entities such as instructor, department, hostel, and fee system.

### Demerits

- The organization is intentionally modeled with limited detail, so real HEI complexity is abstracted away.
- Exceptional workflows such as waitlisting, audit registration, backlog registration, and thesis credits are simplified.
- The sequence diagrams show major interactions, not all low-level validations and database actions.
- A single `RegistrationPolicy` may become large if too many institute-specific rules are added.

### Justification

- A generalized `Student` actor and superclass reduces repetition and matches the UML note on generalization.
- `include` is used in the use case diagram for mandatory reusable behavior such as validation and grade conversion.
- `extend` is used for `Approve Special Request` because it occurs only in exceptional conditions.
- Interface realization through `CGPACalculable` demonstrates abstraction and supports alternate CGPA strategies if required later.
- Multiplicity and ownership relations are shown explicitly because the lecture slides emphasize associations, composition, aggregation, and cardinality.

## 7. Conclusion

The proposed UML design captures the required HEI features: student categorization, registration-time course add/drop, and semester-end CGPA computation. It follows the notation and relationship patterns discussed in class, while keeping the model sufficiently realistic for object-oriented analysis and design.
