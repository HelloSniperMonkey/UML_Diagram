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

## 2. Use Case Diagram

```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

actor Student
actor "UG Student" as UG
actor "PG Student" as PG
actor "PhD Student" as PHD
actor Registrar
actor "Exam Cell" as ExamCell

UG --|> Student
PG --|> Student
PHD --|> Student

rectangle "HEI Registration and Result System" {
  usecase "Login" as UC1
  usecase "View Course Catalog" as UC2
  usecase "Add Course" as UC3
  usecase "Drop Course" as UC4
  usecase "Submit Registration" as UC5
  usecase "Validate Registration Rules" as UC6
  usecase "Check Seat Availability" as UC7
  usecase "Approve Special Request" as UC8
  usecase "Publish Grades" as UC9
  usecase "Calculate CGPA" as UC10
  usecase "View Result / Transcript" as UC11
  usecase "Convert Letter Grade to Grade Point" as UC12
}

Student -- UC1
Student -- UC2
Student -- UC3
Student -- UC4
Student -- UC5
Student -- UC11
Registrar -- UC8
ExamCell -- UC9
ExamCell -- UC10

UC3 .> UC6 : <<include>>
UC3 .> UC7 : <<include>>
UC4 .> UC6 : <<include>>
UC5 .> UC6 : <<include>>
UC10 .> UC12 : <<include>>
UC11 .> UC10 : <<include>>
UC8 .> UC6 : <<extend>>
@enduml
```

### Use case notes

- `Student` is a generalized actor, while `UG Student`, `PG Student`, and `PhD Student` are specialized actors.
- `Add Course`, `Drop Course`, and `Submit Registration` include `Validate Registration Rules`.
- `Approve Special Request` extends rule validation for exceptional cases such as overload or late registration.
- `View Result / Transcript` includes `Calculate CGPA`, because the final transcript view depends on computed grade points.

## 3. Class Diagram

```plantuml
@startuml
skinparam classAttributeIconSize 0

class Organization {
  -name: String
  -code: String
  +addStudent(s: Student): void
  +offerCourse(c: Course): void
}

abstract class Student {
  -studentId: String
  -name: String
  -email: String
  -cgpa: double
  +registerSemester(term: String): Registration
  +viewTranscript(): Transcript
  +getCGPA(): double
}

class UGStudent {
  -program: String
  -year: int
}

class PGStudent {
  -specialization: String
  -assistantship: boolean
}

class PhDStudent {
  -researchArea: String
  -supervisorName: String
}

interface CGPACalculable {
  +calculateCGPA(): double
}

class Registration {
  -registrationId: String
  -term: String
  -status: String
  +addCourse(offering: CourseOffering): boolean
  +dropCourse(offering: CourseOffering): boolean
  +submit(): void
}

class Course {
  -courseCode: String
  -title: String
  -credits: int
}

class CourseOffering {
  -sectionId: String
  -semester: String
  -capacity: int
  -enrolledCount: int
  +hasSeat(): boolean
}

class Enrollment {
  -status: String
  -letterGrade: String
  +assignGrade(g: String): void
}

class Transcript {
  -currentCGPA: double
  +calculateCGPA(): double
  +generateSummary(): String
}

class SemesterRecord {
  -term: String
  -sgpa: double
  +calculateSGPA(): double
}

class RegistrationPolicy {
  -maxCreditsUG: int
  -maxCreditsPG: int
  -maxCreditsPhD: int
  +validateAddDrop(s: Student, r: Registration, o: CourseOffering): boolean
  +validateCreditLoad(s: Student, r: Registration): boolean
}

class GradePointPolicy {
  +toPoint(letterGrade: String): double
}

UGStudent --|> Student
PGStudent --|> Student
PhDStudent --|> Student
Transcript ..|> CGPACalculable

Organization o-- "1" RegistrationPolicy
Organization o-- "1" GradePointPolicy
Organization o-- "*" Student
Organization o-- "*" Course
Course "1" o-- "*" CourseOffering

Student "1" --> "0..*" Registration : creates
Student "1" --> "1" Transcript : owns
Registration "1" *-- "0..*" Enrollment
Enrollment "*" --> "1" CourseOffering
Transcript "1" *-- "1..*" SemesterRecord
SemesterRecord "1" *-- "1..*" Enrollment

Registration ..> RegistrationPolicy : uses
Transcript ..> GradePointPolicy : uses
CourseOffering --> Course : of
@enduml
```

### Class diagram notes

- `Student` is an abstract superclass and `UGStudent`, `PGStudent`, and `PhDStudent` inherit from it.
- `Transcript` realizes the `CGPACalculable` interface.
- `Organization` aggregates students, courses, and policies.
- `Registration` composes `Enrollment` objects because enrollments are created for a specific registration context.
- `Transcript` composes `SemesterRecord` objects because semester records are part of one transcript.
- Multiplicity is explicitly shown to follow the lecture emphasis on associations and cardinality.

## 4. Sequence Diagram 1: Course Add / Drop during Registration

```plantuml
@startuml
actor Student
participant "Registration UI" as UI
participant "Registration Controller" as RC
participant "Registration" as REG
participant "Course Offering" as OFF
participant "Registration Policy" as POL

Student -> UI : selectAddOrDrop(courseCode, action)
UI -> RC : processRequest(studentId, courseCode, action)
RC -> REG : getActiveRegistration(studentId)
REG --> RC : registration
RC -> OFF : findOffering(courseCode)
OFF --> RC : offering
RC -> POL : validateAddDrop(student, registration, offering)
POL --> RC : valid / invalid

alt valid request and seat available
  RC -> OFF : hasSeat()
  OFF --> RC : true
  alt action = add
    RC -> REG : addCourse(offering)
    REG --> RC : success
  else action = drop
    RC -> REG : dropCourse(offering)
    REG --> RC : success
  end
  RC --> UI : showUpdatedRegistration()
  UI --> Student : registration updated
else invalid request or no seat
  RC --> UI : showError(reason)
  UI --> Student : request rejected
end
@enduml
```

## 5. Sequence Diagram 2: Semester-End CGPA Calculation

```plantuml
@startuml
actor "Exam Cell" as ExamCell
participant "Result Service" as RS
participant "Transcript" as TR
participant "Semester Record" as SR
participant "Enrollment" as EN
participant "Grade Point Policy" as GP
participant "Student" as ST

ExamCell -> RS : publishFinalGrades(studentId, term)
RS -> TR : calculateCGPA()
loop for each semester
  TR -> SR : calculateSGPA()
  loop for each enrollment
    SR -> EN : getLetterGrade()
    EN --> SR : letterGrade
    SR -> GP : toPoint(letterGrade)
    GP --> SR : gradePoint
  end
  SR --> TR : sgpa and earnedCredits
end
TR -> ST : update cgpa
ST --> TR : acknowledged
TR --> RS : final CGPA
RS --> ExamCell : transcript ready
@enduml
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
