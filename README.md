--          === STUDENT LEARNING MANAGEMENT SYSTEM (LMS) ===          --

--            === Create Database ===              --
CREATE DATABASE IF NOT EXISTS StudentLMS;
USE StudentLMS;


--         === TABLES ===        --

-- Users table (Abstract for Student/Teacher)
CREATE TABLE Users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    user_type ENUM('Student', 'Teacher', 'Admin') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP NULL,
    is_active BOOLEAN DEFAULT TRUE
);

-- Students table
CREATE TABLE Students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT UNIQUE,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender ENUM('Male', 'Female', 'Other') NOT NULL,
    phone VARCHAR(20),
    address TEXT,
    enrollment_date DATE NOT NULL,
    current_gpa DECIMAL(3,2) DEFAULT 0.00,
    total_credits INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE,
    CONSTRAINT chk_gpa_range CHECK (current_gpa >= 0 AND current_gpa <= 4.00)
);

-- Teachers table
CREATE TABLE Teachers (
    teacher_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT UNIQUE,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    qualification VARCHAR(100),
    department_id INT,
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2),
    office_location VARCHAR(50),
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Departments table
CREATE TABLE Departments (
    department_id INT PRIMARY KEY AUTO_INCREMENT,
    department_code VARCHAR(10) UNIQUE NOT NULL,
    department_name VARCHAR(100) NOT NULL,
    head_teacher_id INT,
    established_date DATE,
    FOREIGN KEY (head_teacher_id) REFERENCES Teachers(teacher_id)
);

-- Courses table
CREATE TABLE Courses (
    course_id INT PRIMARY KEY AUTO_INCREMENT,
    course_code VARCHAR(20) UNIQUE NOT NULL,
    course_name VARCHAR(100) NOT NULL,
    department_id INT,
    credits INT NOT NULL,
    description TEXT,
    max_students INT DEFAULT 30,
    current_enrollment INT DEFAULT 0,
    semester ENUM('Fall', 'Spring', 'Summer') NOT NULL,
    academic_year YEAR NOT NULL,
    created_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (department_id) REFERENCES Departments(department_id),
    FOREIGN KEY (created_by) REFERENCES Teachers(teacher_id),
    CONSTRAINT chk_credits CHECK (credits > 0 AND credits <= 6),
    CONSTRAINT chk_enrollment CHECK (current_enrollment <= max_students)
);

-- Enrollments table
CREATE TABLE Enrollments (
    enrollment_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrollment_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    enrollment_status ENUM('Active', 'Dropped', 'Completed') DEFAULT 'Active',
    grade_letter CHAR(2) NULL,
    final_score DECIMAL(5,2) NULL,
    semester ENUM('Fall', 'Spring', 'Summer') NOT NULL,
    academic_year YEAR NOT NULL,
    FOREIGN KEY (student_id) REFERENCES Students(student_id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES Courses(course_id) ON DELETE CASCADE,
    UNIQUE KEY unique_enrollment (student_id, course_id, semester, academic_year),
    CONSTRAINT chk_final_score CHECK (final_score >= 0 AND final_score <= 100)
);

-- Assignments table
CREATE TABLE Assignments (
    assignment_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    assignment_type ENUM('Homework', 'Project', 'Quiz', 'Exam') NOT NULL,
    max_points DECIMAL(5,2) NOT NULL,
    due_date DATETIME NOT NULL,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (course_id) REFERENCES Courses(course_id) ON DELETE CASCADE,
    CONSTRAINT chk_due_date CHECK (due_date > created_date)
);

-- Submissions table
CREATE TABLE Submissions (
    submission_id INT PRIMARY KEY AUTO_INCREMENT,
    assignment_id INT NOT NULL,
    student_id INT NOT NULL,
    submission_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    file_path VARCHAR(500),
    content TEXT,
    points_earned DECIMAL(5,2) NULL,
    submission_status ENUM('Submitted', 'Late', 'Graded', 'Missing') DEFAULT 'Submitted',
    feedback TEXT,
    graded_by INT NULL,
    graded_date TIMESTAMP NULL,
    FOREIGN KEY (assignment_id) REFERENCES Assignments(assignment_id),
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (graded_by) REFERENCES Teachers(teacher_id),
    UNIQUE KEY unique_submission (assignment_id, student_id)
);

-- Grades table (detailed gradebook)
CREATE TABLE Grades (
    grade_id INT PRIMARY KEY AUTO_INCREMENT,
    enrollment_id INT NOT NULL,
    assignment_id INT,
    grade_value DECIMAL(5,2) NOT NULL,
    weight_percentage DECIMAL(5,2) DEFAULT 100.00,
    grade_type ENUM('Assignment', 'Quiz', 'Midterm', 'Final', 'Participation') NOT NULL,
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (enrollment_id) REFERENCES Enrollments(enrollment_id),
    FOREIGN KEY (assignment_id) REFERENCES Assignments(assignment_id),
    CONSTRAINT chk_grade_value CHECK (grade_value >= 0 AND grade_value <= 100)
);

-- Attendance table
CREATE TABLE Attendance (
    attendance_id INT PRIMARY KEY AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    attendance_date DATE NOT NULL,
    status ENUM('Present', 'Absent', 'Late', 'Excused') NOT NULL,
    notes TEXT,
    marked_by INT,
    marked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (student_id) REFERENCES Students(student_id),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id),
    FOREIGN KEY (marked_by) REFERENCES Teachers(teacher_id),
    UNIQUE KEY unique_attendance (student_id, course_id, attendance_date)
);

-- Announcements table
CREATE TABLE Announcements (
    announcement_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    announcement_type ENUM('General', 'Assignment', 'Exam', 'Event', 'Urgent') DEFAULT 'General',
    posted_by INT NOT NULL,
    posted_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expiry_date DATE NULL,
    is_pinned BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (course_id) REFERENCES Courses(course_id) ON DELETE CASCADE,
    FOREIGN KEY (posted_by) REFERENCES Teachers(teacher_id)
);

-- Materials table
CREATE TABLE Materials (
    material_id INT PRIMARY KEY AUTO_INCREMENT,
    course_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    file_path VARCHAR(500),
    file_type VARCHAR(50),
    file_size BIGINT,
    uploaded_by INT NOT NULL,
    upload_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_public BOOLEAN DEFAULT TRUE,
    download_count INT DEFAULT 0,
    FOREIGN KEY (course_id) REFERENCES Courses(course_id) ON DELETE CASCADE,
    FOREIGN KEY (uploaded_by) REFERENCES Teachers(teacher_id)
);

-- Logs table (for error/activity logging)
CREATE TABLE Logs (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    log_level ENUM('ERROR', 'WARN', 'INFO', 'DEBUG') NOT NULL,
    user_id INT NULL,
    action VARCHAR(100) NOT NULL,
    message TEXT NOT NULL,
    ip_address VARCHAR(45),
    user_agent TEXT,
    log_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
);

-- Transactions table (audit trail)
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    transaction_type VARCHAR(50) NOT NULL,
    table_name VARCHAR(50) NOT NULL,
    record_id INT NOT NULL,
    old_data JSON NULL,
    new_data JSON NULL,
    performed_by INT NULL,
    transaction_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (performed_by) REFERENCES Users(user_id)
);

-- Reports metadata table
CREATE TABLE Reports (
    report_id INT PRIMARY KEY AUTO_INCREMENT,
    report_name VARCHAR(100) NOT NULL,
    report_type ENUM('Transcript', 'Roster', 'GradeSheet', 'Attendance', 'Statistical') NOT NULL,
    generated_by INT NOT NULL,
    generation_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    parameters JSON,
    file_path VARCHAR(500),
    download_count INT DEFAULT 0,
    FOREIGN KEY (generated_by) REFERENCES Users(user_id)
);


--            === CONSTRAINTS ===            -- 


-- Add foreign key for Teachers to Departments
ALTER TABLE Teachers
ADD CONSTRAINT fk_teacher_department
FOREIGN KEY (department_id) REFERENCES Departments(department_id);

-- Add check constraint for phone number format
ALTER TABLE Students
ADD CONSTRAINT chk_phone_format 
CHECK (phone IS NULL OR phone REGEXP '^[0-9+\-\s()]{10,20}$');

-- Add check constraint for email format
ALTER TABLE Users
ADD CONSTRAINT chk_email_format 
CHECK (email REGEXP '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Add check constraint for future dates
ALTER TABLE Courses
ADD CONSTRAINT chk_academic_year 
CHECK (academic_year >= 2020 AND academic_year <= 2030);

-- Add default constraint for enrollment status
ALTER TABLE Enrollments
ALTER COLUMN enrollment_status SET DEFAULT 'Active';

-- Add not null constraint for critical fields
ALTER TABLE Students
MODIFY COLUMN enrollment_date DATE NOT NULL;

ALTER TABLE Courses
MODIFY COLUMN course_code VARCHAR(20) NOT NULL;

ALTER TABLE Assignments
MODIFY COLUMN due_date DATETIME NOT NULL;


--           === VIEWS ===         --


--  Student Grades Summary
CREATE VIEW StudentGradesView AS
SELECT 
    s.student_id,
    CONCAT(s.first_name, ' ', s.last_name) AS student_name,
    c.course_code,
    c.course_name,
    e.final_score,
    e.grade_letter,
    CASE 
        WHEN e.final_score >= 90 THEN 'A'
        WHEN e.final_score >= 80 THEN 'B'
        WHEN e.final_score >= 70 THEN 'C'
        WHEN e.final_score >= 60 THEN 'D'
        ELSE 'F'
    END AS calculated_grade
FROM Students s
JOIN Enrollments e ON s.student_id = e.student_id
JOIN Courses c ON e.course_id = c.course_id
WHERE e.enrollment_status = 'Completed';

--  Course Enrollment Summary
CREATE VIEW CourseEnrollmentView AS
SELECT 
    c.course_id,
    c.course_code,
    c.course_name,
    d.department_name,
    CONCAT(t.first_name, ' ', t.last_name) AS instructor_name,
    c.current_enrollment,
    c.max_students,
    ROUND((c.current_enrollment * 100.0 / c.max_students), 2) AS enrollment_percentage
FROM Courses c
JOIN Departments d ON c.department_id = d.department_id
LEFT JOIN Teachers t ON c.created_by = t.teacher_id
ORDER BY c.current_enrollment DESC;

--  Teacher Course Load
CREATE VIEW TeacherCourseView AS
SELECT 
    t.teacher_id,
    CONCAT(t.first_name, ' ', t.last_name) AS teacher_name,
    d.department_name,
    COUNT(DISTINCT c.course_id) AS total_courses,
    SUM(c.credits) AS total_credits,
    SUM(c.current_enrollment) AS total_students
FROM Teachers t
JOIN Departments d ON t.department_id = d.department_id
LEFT JOIN Courses c ON t.teacher_id = c.created_by
GROUP BY t.teacher_id, t.first_name, t.last_name, d.department_name;

--  Attendance Summary (Monthly)
CREATE VIEW AttendanceSummaryView AS
SELECT 
    s.student_id,
    CONCAT(s.first_name, ' ', s.last_name) AS student_name,
    c.course_code,
    MONTH(a.attendance_date) AS month,
    YEAR(a.attendance_date) AS year,
    COUNT(*) AS total_classes,
    SUM(CASE WHEN a.status = 'Present' THEN 1 ELSE 0 END) AS present_count,
    SUM(CASE WHEN a.status = 'Absent' THEN 1 ELSE 0 END) AS absent_count,
    SUM(CASE WHEN a.status = 'Late' THEN 1 ELSE 0 END) AS late_count,
    ROUND((SUM(CASE WHEN a.status IN ('Present', 'Late') THEN 1 ELSE 0 END) * 100.0 / COUNT(*)), 2) AS attendance_percentage
FROM Attendance a
JOIN Students s ON a.student_id = s.student_id
JOIN Courses c ON a.course_id = c.course_id
GROUP BY s.student_id, c.course_id, MONTH(a.attendance_date), YEAR(a.attendance_date);

--  Upcoming Assignments
CREATE VIEW AssignmentDeadlineView AS
SELECT 
    a.assignment_id,
    a.title,
    c.course_code,
    c.course_name,
    a.assignment_type,
    a.max_points,
    a.due_date,
    DATEDIFF(a.due_date, NOW()) AS days_remaining,
    CASE 
        WHEN DATEDIFF(a.due_date, NOW()) < 0 THEN 'Overdue'
        WHEN DATEDIFF(a.due_date, NOW()) = 0 THEN 'Due Today'
        WHEN DATEDIFF(a.due_date, NOW()) <= 7 THEN 'Due Soon'
        ELSE 'Upcoming'
    END AS deadline_status
FROM Assignments a
JOIN Courses c ON a.course_id = c.course_id
WHERE a.due_date > NOW() - INTERVAL 30 DAY
ORDER BY a.due_date ASC;

--  Student Performance Analysis
CREATE VIEW StudentPerformanceAnalysis AS
SELECT 
    s.student_id,
    CONCAT(s.first_name, ' ', s.last_name) AS student_name,
    COUNT(DISTINCT e.course_id) AS courses_taken,
    AVG(e.final_score) AS average_score,
    s.current_gpa,
    COUNT(DISTINCT a.assignment_id) AS assignments_submitted,
    AVG(sub.points_earned) AS avg_assignment_score
FROM Students s
LEFT JOIN Enrollments e ON s.student_id = e.student_id
LEFT JOIN Submissions sub ON s.student_id = sub.student_id
LEFT JOIN Assignments a ON sub.assignment_id = a.assignment_id
GROUP BY s.student_id;


--             === STORED PROCEDURES ===           --


--  Calculate GPA for a student
DELIMITER //
CREATE PROCEDURE sp_CalculateGPA(
    IN p_student_id INT,
    OUT p_gpa DECIMAL(3,2),
    OUT p_total_credits INT
)
BEGIN
    DECLARE total_points DECIMAL(10,2);
    DECLARE total_weighted_credits INT;
    SELECT 
        SUM(CASE 
            WHEN e.grade_letter = 'A' THEN 4.0
            WHEN e.grade_letter = 'B' THEN 3.0
            WHEN e.grade_letter = 'C' THEN 2.0
            WHEN e.grade_letter = 'D' THEN 1.0
            ELSE 0.0
        END * c.credits),
        SUM(c.credits)
    INTO total_points, total_weighted_credits
    FROM Enrollments e
    JOIN Courses c ON e.course_id = c.course_id
    WHERE e.student_id = p_student_id 
    AND e.enrollment_status = 'Completed'
    AND e.grade_letter IS NOT NULL;
    IF total_weighted_credits > 0 THEN
        SET p_gpa = ROUND(total_points / total_weighted_credits, 2);
    ELSE
        SET p_gpa = 0.00;
    END IF;
    
    SET p_total_credits = total_weighted_credits;
    
    UPDATE Students 
    SET current_gpa = p_gpa, 
        total_credits = p_total_credits
    WHERE student_id = p_student_id;
    
    SELECT p_gpa AS gpa, p_total_credits AS total_credits;
END //
DELIMITER ;

-- Generate Course Report
DELIMITER //
CREATE PROCEDURE sp_GetCourseReport(
    IN p_course_id INT,
    IN p_semester VARCHAR(20),
    IN p_year YEAR
)
BEGIN
    SELECT 
        c.course_code,
        c.course_name,
        c.credits,
        d.department_name,
        CONCAT(t.first_name, ' ', t.last_name) AS instructor,
        c.current_enrollment,
        c.max_students
    FROM Courses c
    JOIN Departments d ON c.department_id = d.department_id
    LEFT JOIN Teachers t ON c.created_by = t.teacher_id
    WHERE c.course_id = p_course_id;
    SELECT 
        COUNT(*) AS total_students,
        AVG(e.final_score) AS average_score,
        MIN(e.final_score) AS min_score,
        MAX(e.final_score) AS max_score,
        SUM(CASE WHEN e.grade_letter = 'A' THEN 1 ELSE 0 END) AS a_count,
        SUM(CASE WHEN e.grade_letter = 'B' THEN 1 ELSE 0 END) AS b_count,
        SUM(CASE WHEN e.grade_letter = 'C' THEN 1 ELSE 0 END) AS c_count,
        SUM(CASE WHEN e.grade_letter = 'D' THEN 1 ELSE 0 END) AS d_count,
        SUM(CASE WHEN e.grade_letter = 'F' THEN 1 ELSE 0 END) AS f_count
    FROM Enrollments e
    WHERE e.course_id = p_course_id 
    AND e.enrollment_status = 'Completed';
    SELECT 
        s.student_id,
        CONCAT(s.first_name, ' ', s.last_name) AS student_name,
        e.final_score,
        e.grade_letter,
        s.current_gpa
    FROM Enrollments e
    JOIN Students s ON e.student_id = s.student_id
    WHERE e.course_id = p_course_id
    AND e.enrollment_status = 'Completed'
    ORDER BY e.final_score DESC
    LIMIT 10;
END //
DELIMITER ;

--  Archive Old Data
DELIMITER //
CREATE PROCEDURE sp_ArchiveOldData(
    IN p_years_old INT
)
BEGIN
    DECLARE archive_date DATE;
    SET archive_date = DATE_SUB(CURDATE(), INTERVAL p_years_old YEAR);
    CREATE TABLE IF NOT EXISTS Enrollments_Archive LIKE Enrollments;
    CREATE TABLE IF NOT EXISTS Attendance_Archive LIKE Attendance;
    CREATE TABLE IF NOT EXISTS Submissions_Archive LIKE Submissions;
    
    START TRANSACTION;
    
    INSERT INTO Enrollments_Archive
    SELECT * FROM Enrollments 
    WHERE enrollment_date < archive_date 
    AND enrollment_status = 'Completed';
    
    DELETE FROM Enrollments 
    WHERE enrollment_date < archive_date 
    AND enrollment_status = 'Completed';

    INSERT INTO Attendance_Archive
    SELECT * FROM Attendance 
    WHERE attendance_date < archive_date;
    
    DELETE FROM Attendance 
    WHERE attendance_date < archive_date;
    
    INSERT INTO Submissions_Archive
    SELECT * FROM Submissions 
    WHERE submission_date < archive_date;
    
    DELETE FROM Submissions 
    WHERE submission_date < archive_date;
    
    -- Commit transaction
    COMMIT;
    
    SELECT 
        'Archiving completed' AS status,
        CURDATE() AS archive_date,
        p_years_old AS years_old;
END //
DELIMITER ;

--  Student Transcript Generation
DELIMITER //
CREATE PROCEDURE sp_GenerateTranscript(
    IN p_student_id INT
)
BEGIN
    SELECT 
        s.student_id,
        CONCAT(s.first_name, ' ', s.last_name) AS student_name,
        s.enrollment_date,
        s.current_gpa,
        s.total_credits
    FROM Students s
    WHERE s.student_id = p_student_id;
    SELECT 
        c.course_code,
        c.course_name,
        c.credits,
        e.enrollment_date,
        e.final_score,
        e.grade_letter,
        CASE 
            WHEN e.grade_letter = 'A' THEN 4.0
            WHEN e.grade_letter = 'B' THEN 3.0
            WHEN e.grade_letter = 'C' THEN 2.0
            WHEN e.grade_letter = 'D' THEN 1.0
            ELSE 0.0
        END AS grade_points,
        d.department_name
    FROM Enrollments e
    JOIN Courses c ON e.course_id = c.course_id
    JOIN Departments d ON c.department_id = d.department_id
    WHERE e.student_id = p_student_id
    AND e.enrollment_status = 'Completed'
    ORDER BY e.enrollment_date DESC;

    SELECT 
        c.semester,
        c.academic_year,
        COUNT(*) AS courses_taken,
        SUM(c.credits) AS total_credits,
        ROUND(AVG(e.final_score), 2) AS avg_score,
        ROUND(AVG(CASE 
            WHEN e.grade_letter = 'A' THEN 4.0
            WHEN e.grade_letter = 'B' THEN 3.0
            WHEN e.grade_letter = 'C' THEN 2.0
            WHEN e.grade_letter = 'D' THEN 1.0
            ELSE 0.0
        END), 2) AS semester_gpa
    FROM Enrollments e
    JOIN Courses c ON e.course_id = c.course_id
    WHERE e.student_id = p_student_id
    AND e.enrollment_status = 'Completed'
    GROUP BY c.semester, c.academic_year
    ORDER BY c.academic_year DESC, 
             FIELD(c.semester, 'Fall', 'Spring', 'Summer');
END //
DELIMITER ;


--           === TRIGGERS ===           -- 


-- Before Insert on Enrollments - Check seat availability
DELIMITER //
CREATE TRIGGER trg_before_enrollment_insert
BEFORE INSERT ON Enrollments
FOR EACH ROW
BEGIN
    DECLARE available_seats INT;
    DECLARE current_enroll INT;
    
    SELECT current_enrollment, max_students 
    INTO current_enroll, available_seats
    FROM Courses 
    WHERE course_id = NEW.course_id;
    
    IF current_enroll >= available_seats THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Course is full. Cannot enroll more students.';
    END IF;
    
    IF NEW.enrollment_date IS NULL THEN
        SET NEW.enrollment_date = CURDATE();
    END IF;
END //
DELIMITER ;

--  After Update on Grades - Update student's GPA automatically
DELIMITER //
CREATE TRIGGER trg_after_grade_update
AFTER UPDATE ON Grades
FOR EACH ROW
BEGIN
    DECLARE v_student_id INT;
    DECLARE v_gpa DECIMAL(3,2);
    DECLARE v_total_credits INT;
    
    SELECT e.student_id INTO v_student_id
    FROM Enrollments e
    WHERE e.enrollment_id = NEW.enrollment_id;
    
    CALL sp_CalculateGPA(v_student_id, v_gpa, v_total_credits);
END //
DELIMITER ;

--  After Insert on Enrollments - Update course enrollment count
DELIMITER //
CREATE TRIGGER trg_after_enrollment_insert
AFTER INSERT ON Enrollments
FOR EACH ROW
BEGIN
    UPDATE Courses 
    SET current_enrollment = current_enrollment + 1
    WHERE course_id = NEW.course_id;
END //
DELIMITER ;

--  Before Insert on Submissions - Check due date
DELIMITER //
CREATE TRIGGER trg_before_submission_insert
BEFORE INSERT ON Submissions
FOR EACH ROW
BEGIN
    DECLARE v_due_date DATETIME;
    
    SELECT due_date INTO v_due_date
    FROM Assignments 
    WHERE assignment_id = NEW.assignment_id;
    
    IF NEW.submission_date > v_due_date THEN
        SET NEW.submission_status = 'Late';
    END IF;
END //
DELIMITER ;

--            === SAMPLE DATA INSERTION ===             --  

--  departments
INSERT INTO Departments (department_code, department_name, established_date) VALUES
('CS', 'Computer Science', '2020-01-01'),
('MATH', 'Mathematics', '2020-01-01'),
('PHYS', 'Physics', '2020-01-01'),
('ENG', 'English', '2020-01-01'),
('BUS', 'Business', '2020-01-01');

--  users
INSERT INTO Users (username, password_hash, email, user_type) VALUES
('admin', 'hashed_password', 'admin@lms.edu', 'Admin'),
('john.doe', 'hashed_password', 'john.doe@student.edu', 'Student'),
('jane.smith', 'hashed_password', 'jane.smith@student.edu', 'Student'),
('prof.jones', 'hashed_password', 'jones@lms.edu', 'Teacher'),
('dr.smith', 'hashed_password', 'smith@lms.edu', 'Teacher');

--  students
INSERT INTO Students (user_id, first_name, last_name, date_of_birth, gender, phone, enrollment_date) VALUES
(2, 'John', 'Doe', '2000-05-15', 'Male', '+1234567890', '2023-09-01'),
(3, 'Jane', 'Smith', '2001-03-22', 'Female', '+1234567891', '2023-09-01');

--  teachers
INSERT INTO Teachers (user_id, first_name, last_name, department_id, hire_date) VALUES
(4, 'Michael', 'Jones', 1, '2020-08-15'),
(5, 'Sarah', 'Smith', 1, '2021-01-10');

-- Update department heads
UPDATE Departments SET head_teacher_id = 1 WHERE department_id = 1;

--  courses
INSERT INTO Courses (course_code, course_name, department_id, credits, max_students, semester, academic_year, created_by) VALUES
('CS101', 'Introduction to Programming', 1, 3, 30, 'Fall', 2024, 1),
('CS201', 'Data Structures', 1, 4, 25, 'Fall', 2024, 1),
('MATH101', 'Calculus I', 2, 3, 40, 'Fall', 2024, 2),
('ENG101', 'Composition', 4, 3, 35, 'Fall', 2024, 2);

--  enrollments
INSERT INTO Enrollments (student_id, course_id, enrollment_status, semester, academic_year) VALUES
(1, 1, 'Active', 'Fall', 2024),
(1, 2, 'Active', 'Fall', 2024),
(2, 1, 'Active', 'Fall', 2024),
(2, 3, 'Active', 'Fall', 2024);
--  assignments
INSERT INTO Assignments (course_id, title, assignment_type, max_points, due_date) VALUES
(1, 'Programming Basics', 'Homework', 100, DATE_ADD(NOW(), INTERVAL 7 DAY)),
(1, 'Midterm Exam', 'Exam', 100, DATE_ADD(NOW(), INTERVAL 30 DAY)),
(2, 'Linked List Implementation', 'Project', 100, DATE_ADD(NOW(), INTERVAL 14 DAY)),
(3, 'Calculus Problems', 'Homework', 100, DATE_ADD(NOW(), INTERVAL 10 DAY)),
(1, 'Final Project', 'Project', 100, DATE_ADD(NOW(), INTERVAL 60 DAY));
--  submissions
INSERT INTO Submissions (assignment_id, student_id, points_earned, submission_status) VALUES
(1, 1, 95.5, 'Graded'),
(1, 2, 88.0, 'Graded'),
(3, 1, 92.0, 'Graded');

-- grades
INSERT INTO Grades (enrollment_id, assignment_id, grade_value, grade_type) VALUES
(1, 1, 95.5, 'Assignment'),
(3, 1, 88.0, 'Assignment'),
(2, 3, 92.0, 'Assignment');

--  attendance
INSERT INTO Attendance (student_id, course_id, attendance_date, status, marked_by) VALUES
(1, 1, '2024-09-10', 'Present', 1),
(2, 1, '2024-09-10', 'Present', 1),
(1, 1, '2024-09-12', 'Late', 1);

-- announcements
INSERT INTO Announcements (course_id, title, content, posted_by, is_pinned) VALUES
(1, 'Welcome to CS101', 'Welcome to Introduction to Programming...', 1, TRUE),
(NULL, 'System Maintenance', 'The LMS will be down for maintenance...', 1, FALSE);

-- materials
INSERT INTO Materials (course_id, title, description, uploaded_by) VALUES
(1, 'Syllabus', 'Course syllabus document', 1),
(1, 'Lecture 1 Slides', 'Introduction to programming concepts', 1);

--  logs
INSERT INTO Logs (log_level, user_id, action, message) VALUES
('INFO', 1, 'APP_START', 'Application started'),
('INFO', 2, 'LOGIN', 'User logged in'),
('INFO', 3, 'LOGOUT', 'User logged out'),
('WARN', 1, 'MEMORY_HIGH', 'Memory usage at 85%'),
('INFO', 4, 'UPLOAD', 'File uploaded successfully'),
('ERROR', NULL, 'CONNECTION_LOST', 'Database connection lost'),
('INFO', 2, 'ENROLLMENT', 'Course enrollment completed'),
('INFO', 1, 'BACKUP', 'Daily backup completed'),
('INFO', 3, 'PROFILE_UPDATE', 'Profile information updated'),
('INFO', 4, 'GRADE_SUBMIT', 'Grades submitted for CS101');

-- transactions
INSERT INTO Transactions (transaction_type, table_name, record_id, performed_by) VALUES
('CREATE', 'Students', 1, 1),
('UPDATE', 'Courses', 2, 4),
('DELETE', 'Enrollments', 3, 1),
('VIEW', 'Grades', 4, 2),
('EXPORT', 'Reports', 5, 1),
('PRINT', 'Transcripts', 6, 3),
('APPROVE', 'Assignments', 7, 4),
('REJECT', 'Submissions', 8, 4),
('ARCHIVE', 'OldData', 9, 1),
('RESTORE', 'Backup', 10, 1);


--        === INDEXES FOR PERFORMANCE ===           --


CREATE INDEX idx_student_name ON Students(first_name, last_name);
CREATE INDEX idx_course_code ON Courses(course_code);
CREATE INDEX idx_enrollment_student ON Enrollments(student_id);
CREATE INDEX idx_enrollment_course ON Enrollments(course_id);
CREATE INDEX idx_submission_assignment ON Submissions(assignment_id);
CREATE INDEX idx_submission_student ON Submissions(student_id);
CREATE INDEX idx_attendance_date ON Attendance(attendance_date);
CREATE INDEX idx_assignment_due ON Assignments(due_date);
CREATE INDEX idx_log_timestamp ON Logs(log_timestamp);
CREATE INDEX idx_user_email ON Users(email);
