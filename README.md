# Day-45-SQL-Practice-YOUR.HARVAD

# SQL Query Optimization and Indexing Journey

We began by examining a query that retrieved enrollment information for a specific student (`student_id = 13`). The database was scanning every row in the `enrollments` table, so we improved performance by creating:

CREATE INDEX enrollments_by_student_id ON enrollments (student_id);
🔍 Query Optimization for Course-Based Enrollments
To retrieve students enrolled in a specific course:
 
SELECT id, name
FROM students
WHERE id IN (
    SELECT student_id
    FROM enrollments
    WHERE course_id = (
        SELECT id
        FROM courses
        WHERE department = 'Computer Science'
        AND number = 50
        AND semester = 'Fall 2023'
    )
);
We observed that the query caused a full table scan on enrollments.course_id, so we added:

 
CREATE INDEX enrollments_by_course_id ON enrollments (course_id);
However, performance issues persisted due to full scans on the courses table. To fix this, we created a composite index:

 
CREATE INDEX courses_by_department_number_semester 
ON courses (department, number, semester);
📘 Index Order Matters
Later, we ran this query:

 
SELECT id, department, number, title
FROM courses
WHERE department = 'Computer Science'
AND semester = 'Spring 2024';
I mistakenly assumed the previous index would help. I learned that since semester wasn’t the leftmost column in the composite index, it wasn’t used. So I created:
 
CREATE INDEX course_by_semester ON courses (semester);
✅ Optimizing Requirements Queries
For this query:

 
SELECT requirements.name
FROM requirements
WHERE id = (
    SELECT requirement_id
    FROM satisfies
    WHERE course_id = (
        SELECT id
        FROM courses
        WHERE title = 'Advanced Databases'
        AND semester = 'Fall 2023'
    )
);
We optimized it by indexing:

 
CREATE INDEX satisfies_course_id ON satisfies (course_id);
🎯 Final Optimized Queries
These final queries made full use of the indexes:
 
SELECT requirements.name, COUNT(*) AS courses
FROM requirements
JOIN satisfies ON requirements.id = satisfies.requirement_id
WHERE satisfies.course_id IN (
    SELECT course_id
    FROM enrollments
    WHERE student_id = 8
)
GROUP BY requirements.name;
 
SELECT department, number, title
FROM courses
WHERE title LIKE 'History%'
AND semester = 'Fall 2023';
✍️ Key Takeaways
Thoughtful indexing significantly boosts SQL query performance.

Composite indexes must match the filter order to be effective.

Testing and interpreting query plans is critical.

Learned the real-world tradeoffs of query optimization strategies.

End of log.
