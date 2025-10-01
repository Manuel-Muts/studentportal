#phase1 Declarations=Jac+LLM
import from byllm.llm { Model }
glob llm = Model(model_name="gemini/gemini-2.0-flash", api_key="AIzaSyDdxnJz9rjZYIm_z84nXTNFJr3EkgajPys", verbose=False);

# Function definition
def cbc_feedback(marks: dict, total_marks: int) -> str {
    # Generates feedback for a student's performance in a CBC exam.
    feedback = llm.complete(
        prompt=f"Student scored {marks} out of {total_marks}. Give short CBC feedback highlighting strengths and areas of improvement.",
        max_tokens=100
    );
    return feedback;
}
# Walker definitions
walker StudentPortal {
    has admission_number: str;
    has name: str;
    has marks: dict;
    has total: int = 0;

    can calculate_total with student entry;
    can show_results with student entry;
}

walker teachers {
    can add_marks with teacher entry;
}

# Implementation of walker: add marks to a student
impl teachers.add_marks {
    # Example: Add marks to a student's record
    # You can customize this logic as needed
    if "marks" in here {
        for subject in here.marks {
            mark = here.marks[subject];
            here.marks[subject] = mark;
        }
        here.total = sum(here.marks.values());
    }
}

# Node definitions
node student {
    has admission_number: str;
    has name: str;
    has marks: dict;
    has total: int = 0;
}

node teacher {
    has name: str;
}

# Edge definition
edge student_entry{
   has from_teacher: teacher;
   has to: Student;
}

# Implementation of walker: calculate total marks
impl StudentPortal.calculate_total {
    self.total = sum(self.marks.values());
    here.total = self.total;
}

# Walker implementation: show student results + CBC feedback
impl StudentPortal.show_results {
    print(f"Student: {self.name} ({self.admission_number})");
    print("Subject Marks:");
    for subject in self.marks {
        print(f"  {subject}: {self.marks[subject]}");
    }
    print(f"Total Marks: {self.total}");

    # Use LLM to generate CBC feedback
    feedback = cbc_feedback(self.marks, self.total);
    print("Feedback:");
    print(feedback);
}

# Entry point to run the program
with entry:__main__ {
    s1 = root spawn StudentPortal("ADM001", "Amina", {"Mathematics": 85, "English": 70, "Kiswahili": 78, "Science": 90, "Social Studies": 75});
    s2 = root spawn StudentPortal("ADM002", "Brian", {"Mathematics": 60, "English": 55, "Kiswahili": 50, "Science": 65, "Social Studies": 58});
    walk(StudentPortal.show_results, s1);
    walk(StudentPortal.show_results, s2);
}
