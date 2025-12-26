---
name: object-calisthenics
description: Apply this set of design rules for improving the maintainability, readability, testability, and comprehensibility of object-oriented code. Use when writing new classes and methods, refactoring code, identifying and resolving design smells and code reviewing.
---

# Object Calisthenics

Set of design rules for improving the maintainability, readability, testability, and comprehensibility of object-oriented code.

## When to use this skill

- Writing new classes and methods.
- Refactoring code.
- Identifying and resolving design smells.
- Code review.

## The rules

These rules are **constraints that force better design decisions**. Not all rules need to be applied all the time. Rule violations are often a smell of bad quality code.

### One level of indentation per method

Deep nesting reduces readability and indicates complex logic that should be decomposed.

**Bad**:

```python
class StudentReportGenerator:
    def __init__(self, grade_calculator: GradeCalculator):
        self._grade_calculator = grade_calculator

    def generate(self, students: Students):
        report = []

        for student in students:
            student_data = {"name": student.name, "subjects": []}

            for subject in student.subjects:
                student_data["subjects"].append({
                    "name": subject.name,
                    "grade": self._grade_calculator.calculate(subject.score)
                })

            report.append(student_data)

        return report
```

**Good**:

```python
class StudentReportGenerator:
    def __init__(self, grade_calculator: GradeCalculator):
        self._grade_calculator = grade_calculator

    def generate(self, students: Students):
        report = []

        for student in students:
            report.append(self._student_report(student))

        return report

    def _student_report(self, student: Student):
        subjects = []

        for subject in student.subjects:
            subjects.append(self._subject_data(subject))

        return {
            "name": student.name,
            "subjects": subjects
        }

    def _subject_data(self, subject: Subject):
        return {
            "name": subject.name,
            "grade": self._grade_calculator.calculate(subject.score)
        }
```

**Guideline**: _Extract Method refactoring. Split methods into smaller focused pieces._

### Do not use the `else` keyword

Nested conditionals create complex branching logic. Early returns and polymorphism produce clearer code.

**Bad**:

```python
class GradeCalculador:
    def calculate(self, score):
        if score >= 90:
            return "A"
        else:
            if score >= 70:
                return "B"
            else:
                return "C"
```

**Good - Early return**:

```python
class GradeCalculador:
    def calculate(self, score):
        if score >= 90:
            return "A"

        if score >= 70:
            return "B"

        return "C"
```

**Good - Guard clauses**:

```python
class GradeCalculador:
    def calculate(self, score):
        self._ensure_score_is_valid(score)

        if score >= 90:
            return "A"

        if score >= 70:
            return "B"

        return "C"

    def _ensure_score_is_valid(self, score):
        if score < 0 or score > 100:
            raise InvalidScore
```

**Alternative Solutions**:

- Polymorphism (Strategy/State patterns)
- Null Object pattern

### Wrap all primitives and strings

Prevents Primitive Obsession anti-pattern. Domain concepts deserve explicit types.

**Bad**:

```python
class Subject:
    def __init__(self, name: str, score: int) -> None:
        self._name = name
        self._score = score

    def is_passing(self) -> bool:
        return self._score >= 70

    def add_bonus(self, points: int) -> None:
        self._score = min(100, self._score + points)
```

**Good**:

```python
class Score:
    def __init__(self, value: int) -> None:
        self._value = value

    def is_passing(self) -> bool:
        return self._value >= 70

    def add_bonus(self, points: int) -> Score:
        return Score(min(100, self._value + points))


class Subject:
    def __init__(self, name: str, score: Score) -> None:
        self._name = name
        self._score = score

    def is_passing(self) -> bool:
        return self._score.is_passing()

    def add_bonus(self, points: int) -> None:
        self._score = self._score.add_bonus(points)
```

**Guideline**: _Apply it when the primitive has validation rules, business logic or related behaviors. Like `Email`, `PhoneNumber`, `Temperature`, `Distance` or `DateRange`_

### First class collections

Collection-specific behaviors need a dedicated home. Mixing collection logic with other responsibilities violates Single Responsibility Principle.

**Bad**:

```python
class Student:
    def __init__(self, name: str) -> None:
        self._name = name
        self._subjects = []

    def add_subject(self, subject: Subject) -> None:
        self._subjects.append(subject)

    def average_score(self) -> float:
        if not self._subjects:
            return 0.0

        total = sum(subject.score() for subject in self._subjects)

        return total / len(self._subjects)

    def passing_subjects(self) -> list[Subject]:
        return [subject for subject in self._subjects if subject.is_passing()]

    def failed_subjects(self) -> list[Subject]:
        return [subject for subject in self._subjects if not subject.is_passing()]
```

**Good**:

```python
class Subjects:
    def __init__(self) -> None:
        self._items = []

    def add(self, subject: Subject) -> None:
        self._items.append(subject)

    def total_score(self) -> int:
        if not self._items:
            return 0

        return sum(subject.score() for subject in self._items)

    def average_score(self) -> float:
        if not self._items:
            return 0.0

        return self.total_score() / len(self._items)

    def passing(self) -> list[Subject]:
        return [subject for subject in self._items if subject.is_passing()]

    def failed(self) -> list[Subject]:
        return [subject for subject in self._items if not subject.is_passing()]


class Student:
    def __init__(self, name: str) -> None:
        self._name = name
        self._subjects = Subjects()

    def add_subject(self, subject: Subject) -> None:
        self._subjects.add(subject)

    def average_score(self) -> float:
        return self._subjects.average_score()
```

**Guideline**: _Collection behaviors (filtering, mapping, aggregating) have a clear location._

### One dot per line

Method chaining couples code to object structure. "Talk to friends, not strangers.".

**Bad**:

```python
class Classroom:
    def __init__(self, students: list[Student]) -> None:
        self._students = students

    def top_student_name(self) -> str:
        top_student = max(self._students, key=lambda student: student.subjects().average_score())

        return top_student.name().value().title()
```

**Good**:

```python
class Name:
    def __init__(self, value: str) -> None:
        self._value = value

    def formatted(self) -> str:
        return self._value.title()

class Student:
    def __init__(self, name: Name, subjects: Subjects) -> None:
        self._name = name
        self._subjects = subjects

    def formatted_name(self) -> str:
        return self._name.formatted()

    def average_score(self) -> float:
        return self._subjects.average_score()

class Classroom:
    def __init__(self, students: list[Student]):
        self._students = students

    def top_student_name(self) -> str:
        top_student = max(self._students, key=lambda s: s.average_score())

        return top_student.formatted_name()
```

**Exception**: _Fluent interfaces are allowed:_

```python
StudentBuilder().named("Bender").with_subjects(["Maths", "Physics"]).build()
```

**Guideline**: _Tell objects what to do._

### Do not abbreviate

Abbreviations suggest code smells.

**Bad**:

```python
class StdRptGen:
    def gen_rpt(self, stds: list[Student]) -> dict:
        rpt = []

        for std in stds:
            std_data = {
                "nm": std.name(),
                "avg": std.average_score(),
                "sts": self._get_sts(std)
            }
            rpt.append(std_data)

        return rpt

    def _get_sts(self, std: Student) -> str:
        return "pass" if std.is_passing() else "fail"
```

**Good**:

```python
class StudentReportGenerator:
    def generate(self, students: list[Student]) -> dict:
        report = []

        for student in students:
            report.append({
                "name": student.name(),
                "average": student.average_score(),
                "status": self._determine_status(student)
            })

        return report

    def _determine_status(self, student: Student) -> str:
        if not student.is_passing():
            return "fail"

        return "pass"
```

**Guideline**: _If you can't find a good name, there's likely a design problem. Names should clearly communicate intent._

### Keep all entities small

Large files are harder to read, understand, and maintain. Size often indicates multiple responsibilities.

**Application**:

- Classes: ~100 lines maximum
- Modules/packages: ~10 files maximum

**Guideline**: _Adjust these limits to your project's context. A well-structured 130-line class may be fine. Use size as an indicator of potential multiple responsibilities, not as a strict rule._

### No classes with more than two instance variables

Forces high cohesion, better encapsulation, and explicit decomposition of responsibilities.

**Bad**:

```python
class Student:
    def __init__(
        self,
        first_name: str,
        last_name: str,
        email: str,
        age: int,
        subjects: Subjects
    ) -> None:
        self._first_name = first_name
        self._last_name = last_name
        self._email = email
        self._age = age
        self._subjects = subjects

    def full_name(self) -> str:
        return f"{self._first_name} {self._last_name}"

    def contact_info(self) -> str:
        return f"{self.full_name()}"
```

**Good**:

```python
class Name:
    def __init__(self, first_name: str, last_name: str) -> None:
        self._first_name = first_name
        self._last_name = last_name

    def full(self) -> str:
        return f"{self._first_name} {self._last_name}"


class ContactInfo:
    def __init__(self, name: Name, email: str) -> None:
        self._name = name
        self._email = email

    def formatted(self) -> str:
        return f"{self._name.full()}"


class PersonalInfo:
    def __init__(self, contact: ContactInfo, age: int) -> None:
        self._contact = contact
        self._age = age

    def formatted(self) -> str:
        return self._contact.formatted()


class Student:
    def __init__(self, personal_info: PersonalInfo, subjects: Subjects) -> None:
        self._personal_info = personal_info
        self._subjects = subjects

    def contact_info(self) -> str:
        return self._personal_info.formatted()
```

**Guideline**: _This rule forces fundamental rethinking of object design. "Two" is arbitrary but forces aggressive decomposition. Combined with Rule 3 (wrap primitives), it creates highly cohesive, focused classes._

### No getters/setters/properties

Don't expose internal state. Tell objects what to do, don't ask for their data.

**Bad**:

_Get data from object → Make decision → Update object._

```python
class Score:
    def __init__(self, value: int) -> None:
        self._value = value

    def get_value(self) -> int:
        return self._value

    def set_value(self, score: int) -> None:
        self._value = score


score = Score(10)
bonus = 20
new_score = score.get_value() + bonus
score.set_value(new_score)
```

**Good**:

_Tell object to perform action → Object makes decisions internally_

```python
class Score:
    def __init__(self, value: int) -> None:
        self._value = value

    def add_bonus(self, score: int) -> Self:
        return Score(self._value + score)


score = Score(10)
new_score = score.add_bonus(20)
```

**Exception**: _Read-only accessors for display purposes are acceptable._

```python
class Score:
    def __init__(self, value: int) -> None:
        self._value = value

    def add_bonus(self, score: int) -> Self:
        return Score(self._value + score)

    def value(self) -> int:  # OK - read-only for UI display
        return self._value
```

**Guideline**: _Tell objects what to do, don't ask for their data to make decisions outside the object._

## Application Strategy

### Progressive Adoption

Don't apply all rules at once. Start with the most impactful:

#### **High impact. Easy to Apply**

- [One level of indentation per method](#one-level-of-indentation-per-method).
- [Do not use the `else` keyword](#do-not-use-the-else-keyword)
- [Do not abbreviate](#do-not-abbreviate).

#### **High Impact, Moderate Difficulty**

- [One dot per line](#one-dot-per-line).
- [No getters/setters/properties](#no-getterssettersproperties).

#### **Requires Design Changes**:

- [Wrap all primitives and strings](#wrap-all-primitives-and-strings).
- [First class collections](#first-class-collections).
- [No classes with more than two instance variables](#no-classes-with-more-than-two-instance-variables).

### Using rules as smell detectors

Rule violations are often a smell of bad quality code:

- **One level of indentation per method violation** → Method too complex that needs decomposition.
- **Do not use the `else` keyword violation** → Consider polymorphism or state pattern.
- **One dot per line violation** → Too much coupling that violates encapsulation.
- **Keep all entities small violation** → Class/module doing too much.
- **No getters/setters/properties violation** → Procedural thinking in OOP system.

## Expected Outcomes

Applying these rules consistently produces code that is:

- **More maintainable**.
- **More readable**.
- **More testable**.
- **More comprehensible**.
