
# BCI Management System with Encapsulation
from __future__ import annotations
from datetime import datetime
from enum import Enum
from typing import List, Optional


class DeviceStatus(Enum):
    AVAILABLE = "available"
    IN_USE = "in_use"
    MAINTENANCE = "maintenance"


class Patient:
    def __init__(self, patient_id: str, name: str, age: int):
        self.__patient_id = patient_id
        self.name = name
        self.age = age

    
  def patient_id(self) -> str:
        return self.__patient_id

    
  def name(self) -> str:
        return self.__name

  @name.setter
    def name(self, value: str) -> None:
        if not value or not value.strip():
            raise ValueError("name must be non-empty")
        self.__name = value.strip()

    
   def age(self) -> int:
        return self.__age

   @age.setter
    def age(self, value: int) -> None:
        if not isinstance(value, int) or not (0 <= value <= 130):
            raise ValueError("age must be an integer between 0 and 130")
        self.__age = value

  def __repr__(self) -> str:
        return f"Patient(id={self.patient_id!r}, name={self.name!r}, age={self.age})"


class Device:
    def __init__(self, device_id: str, model: str, status: DeviceStatus = DeviceStatus.AVAILABLE):
        self.__device_id = device_id
        self.model = model
        self.status = status

    
   def device_id(self) -> str:
        return self.__device_id

    
  def model(self) -> str:
        return self.__model

  @model.setter
    def model(self, value: str) -> None:
        if not value or not value.strip():
            raise ValueError("model must be non-empty")
        self.__model = value.strip()
    def status(self) -> DeviceStatus:
        return self.__status

  @status.setter
    def status(self, value: DeviceStatus) -> None:
        if not isinstance(value, DeviceStatus):
            raise ValueError("invalid status")
        self.__status = value

  def __repr__(self) -> str:
        return f"Device(id={self.device_id!r}, model={self.model!r}, status={self.status.value!r})"


class Session:
    def __init__(self, session_id: str, patient: Patient, device: Device):
        self.__session_id = session_id
        self.__patient = patient
        self.__device = device
        self.__started_at: Optional[datetime] = None
        self.__ended_at: Optional[datetime] = None

    
  def session_id(self) -> str:
        return self.__session_id

    
  def patient(self) -> Patient:
        return self.__patient

    
  def device(self) -> Device:
        return self.__device

    
   def started_at(self) -> Optional[datetime]:
        return self.__started_at

    
  def ended_at(self) -> Optional[datetime]:
        return self.__ended_at

  def start(self) -> None:
        if self.__started_at is not None:
            raise RuntimeError("session already started")
        if self.__device.status != DeviceStatus.AVAILABLE:
            raise RuntimeError("device not available")
        self.__device.status = DeviceStatus.IN_USE
        self.__started_at = datetime.utcnow()

  def end(self) -> None:
        if self.__started_at is None:
            raise RuntimeError("session not started")
        if self.__ended_at is not None:
            raise RuntimeError("session already ended")
        self.__ended_at = datetime.utcnow()
        self.__device.status = DeviceStatus.AVAILABLE

   def __repr__(self) -> str:
        return (
            f"Session(id={self.session_id!r}, patient={self.patient.patient_id!r}, "
            f"device={self.device.device_id!r}, started_at={self.started_at}, ended_at={self.ended_at})"
        )


class PaymentStatus(Enum):
    PENDING = "pending"
    PAID = "paid"
    FAILED = "failed"


class Payment:
    def __init__(self, payment_id: str, student: "Student", amount: float, status: PaymentStatus = PaymentStatus.PENDING, date: Optional[datetime] = None):
        self.__payment_id = payment_id
        self.__student = student
        self.amount = amount
        self.status = status
        self.date = date or datetime.utcnow()

  def payment_id(self) -> str:
        return self.__payment_id

  def student(self) -> "Student":
        return self.__student

  def amount(self) -> float:
        return self.__amount

   @amount.setter
    def amount(self, value: float) -> None:
        if not isinstance(value, (int, float)) or value < 0:
            raise ValueError("amount must be a non-negative number")
        self.__amount = float(value)

  def status(self) -> PaymentStatus:
        return self.__status

   @status.setter
    def status(self, value: PaymentStatus) -> None:
        if not isinstance(value, PaymentStatus):
            raise ValueError("invalid payment status")
        self.__status = value

  def date(self) -> datetime:
        return self.__date

  @date.setter
    def date(self, value: datetime) -> None:
        if not isinstance(value, datetime):
            raise ValueError("date must be a datetime")
        self.__date = value

   def mark_paid(self) -> None:
        self.status = PaymentStatus.PAID
        self.date = datetime.utcnow()

   def __repr__(self) -> str:
        return f"Payment(id={self.payment_id!r}, student={self.student.student_id!r}, amount={self.amount}, status={self.status.value!r})"


class Course:
    def __init__(self, course_id: str, title: str, credits: int):
        self.__course_id = course_id
        self.title = title
        self.credits = credits
        self.__lecturer: Optional[Lecturer] = None
        self.__students: List[Student] = []

  def course_id(self) -> str:
        return self.__course_id

  def title(self) -> str:
        return self.__title

   @title.setter
    def title(self, value: str) -> None:
        if not value or not value.strip():
            raise ValueError("title must be non-empty")
        self.__title = value.strip()

   def credits(self) -> int:
        return self.__credits

  @credits.setter
    def credits(self, value: int) -> None:
        if not isinstance(value, int) or value <= 0:
            raise ValueError("credits must be a positive integer")
        self.__credits = value

   def lecturer(self) -> Optional["Lecturer"]:
        return self.__lecturer

  def assign_lecturer(self, lecturer: "Lecturer") -> None:
        if lecturer is None:
            raise ValueError("lecturer required")
        self.__lecturer = lecturer
        if self not in lecturer.courses:
            lecturer._add_course_internal(self)

 def students(self) -> List["Student"]:
        return list(self.__students)

  def enroll_student(self, student: "Student") -> None:
        if student in self.__students:
            return
        self.__students.append(student)
        if self not in student.courses:
            student._add_course_internal(self)

  def remove_student(self, student: "Student") -> None:
        if student in self.__students:
            self.__students.remove(student)
            student._remove_course_internal(self)

  def __repr__(self) -> str:
        return f"Course(id={self.course_id!r}, title={self.title!r}, credits={self.credits})"


class Lecturer:
    def __init__(self, lecturer_id: str, name: str):
        self.__lecturer_id = lecturer_id
        self.name = name
        self.__courses: List[Course] = []
        self.__department: Optional[Department] = None

    
  def lecturer_id(self) -> str:
        return self.__lecturer_id

   def name(self) -> str:
        return self.__name

   @name.setter
    def name(self, value: str) -> None:
        if not value or not value.strip():
            raise ValueError("name must be non-empty")
        self.__name = value.strip()

  def courses(self) -> List[Course]:
        return list(self.__courses)

  def _add_course_internal(self, course: Course) -> None:
        if course not in self.__courses:
            self.__courses.append(course)

  def assign_course(self, course: Course) -> None:
        course.assign_lecturer(self)

  def department(self) -> Optional["Department"]:
        return self.__department

  def set_department(self, dept: "Department") -> None:
        self.__department = dept
        if self not in dept.lecturers:
            dept._add_lecturer_internal(self)

  def __repr__(self) -> str:
        return f"Lecturer(id={self.lecturer_id!r}, name={self.name!r})"


class Student:
    def __init__(self, student_id: str, name: str, age: int):
        self.__student_id = student_id
        self.name = name
        self.age = age
        self.__department: Optional[Department] = None
        self.__courses: List[Course] = []
        self.__payments: List[Payment] = []

  def student_id(self) -> str:
        return self.__student_id

  def name(self) -> str:
        return self.__name

  @name.setter
    def name(self, value: str) -> None:
        if not value or not value.strip():
            raise ValueError("name must be non-empty")
        self.__name = value.strip()

  def age(self) -> int:
        return self.__age

  @age.setter
    def age(self, value: int) -> None:
        if not isinstance(value, int) or not (0 <= value <= 130):
            raise ValueError("age must be an integer between 0 and 130")
        self.__age = value

  def department(self) -> Optional[Department]:
        return self.__department

  def set_department(self, dept: Department) -> None:
        self.__department = dept
        if self not in dept.students:
            dept._add_student_internal(self)

  def courses(self) -> List[Course]:
        return list(self.__courses)

  def _add_course_internal(self, course: Course) -> None:
        if course not in self.__courses:
            self.__courses.append(course)

  def _remove_course_internal(self, course: Course) -> None:
        if course in self.__courses:
            self.__courses.remove(course)

  def enroll(self, course: Course) -> None:
        course.enroll_student(self)

  def payments(self) -> List[Payment]:
        return list(self.__payments)

  def make_payment(self, payment_id: str, amount: float) -> Payment:
        payment = Payment(payment_id, self, amount)
        self.__payments.append(payment)
        return payment

  def __repr__(self) -> str:
        return f"Student(id={self.student_id!r}, name={self.name!r}, age={self.age})"


class Department:
    def __init__(self, dept_id: str, name: str):
        self.__dept_id = dept_id
        self.name = name
        self.__lecturers: List[Lecturer] = []
        self.__students: List[Student] = []
        self.__courses: List[Course] = []

  def dept_id(self) -> str:
        return self.__dept_id

  def name(self) -> str:
        return self.__name

  @name.setter
    def name(self, value: str) -> None:
        if not value or not value.strip():
            raise ValueError("department name must be non-empty")
        self.__name = value.strip()

  def lecturers(self) -> List[Lecturer]:
        return list(self.__lecturers)

  def _add_lecturer_internal(self, lecturer: Lecturer) -> None:
        if lecturer not in self.__lecturers:
            self.__lecturers.append(lecturer)

  def add_lecturer(self, lecturer: Lecturer) -> None:
        lecturer.set_department(self)

  def students(self) -> List[Student]:
        return list(self.__students)

  def _add_student_internal(self, student: Student) -> None:
        if student not in self.__students:
            self.__students.append(student)

  def add_student(self, student: Student) -> None:
        student.set_department(self)

  def courses(self) -> List[Course]:
        return list(self.__courses)

  def add_course(self, course: Course) -> None:
        if course not in self.__courses:
            self.__courses.append(course)

  def __repr__(self) -> str:
        return f"Department(id={self.dept_id!r}, name={self.name!r})"
