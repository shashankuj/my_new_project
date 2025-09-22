-- Task 1: Database Setup and Schema Design for a Library System
DROP DATABASE IF EXISTS library_db;
CREATE DATABASE library_db;
USE library_db; -- For MySQL. Use \c library_db for PostgreSQL, or connect manually in other tools.

-- 1. Table for storing author information
CREATE TABLE authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    nationality VARCHAR(100),
    birth_year YEAR
);

-- 2. Table for storing book information (independent of a specific physical copy)
CREATE TABLE books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    isbn VARCHAR(17) UNIQUE, -- ISBN-13 format can have hyphens
    publication_year YEAR,
    author_id INT NOT NULL,
    genre VARCHAR(100),
    -- Define the relationship: A book is written by one author.
    -- An author can write many books.
    FOREIGN KEY (author_id) REFERENCES authors(author_id) ON DELETE CASCADE
);

-- 3. Table for storing members/borrowers
CREATE TABLE members (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20),
    date_of_birth DATE,
    membership_date DATE NOT NULL DEFAULT (CURRENT_DATE)
);

-- 4. Table for storing individual physical copies of books
-- This allows the library to have multiple copies of the same book.
CREATE TABLE book_copies (
    copy_id INT AUTO_INCREMENT PRIMARY KEY,
    book_id INT NOT NULL,
    acquisition_date DATE DEFAULT (CURRENT_DATE),
    -- Status: 'Available', 'Loaned', 'Under Maintenance'
    status ENUM('Available', 'Loaned', 'Under Maintenance') DEFAULT 'Available',
    -- Define the relationship: A copy is of one book. A book can have many copies.
    FOREIGN KEY (book_id) REFERENCES books(book_id) ON DELETE CASCADE
);

-- 5. Table for tracking book loans (the core transactional table)
CREATE TABLE book_loans (
    loan_id INT AUTO_INCREMENT PRIMARY KEY,
    copy_id INT NOT NULL,
    member_id INT NOT NULL,
    loan_date DATE NOT NULL DEFAULT (CURRENT_DATE),
    due_date DATE NOT NULL,
    return_date DATE NULL, -- NULL if the book is not yet returned
    -- Define relationships:
    -- A loan is for one specific copy. A copy can be loaned out many times over its life.
    FOREIGN KEY (copy_id) REFERENCES book_copies(copy_id),
    -- A loan is taken by one member. A member can have many loans (current or historical).
    FOREIGN KEY (member_id) REFERENCES members(member_id)
);

INSERT INTO authors (first_name, last_name, nationality, birth_year) VALUES
('George', 'Orwell', 'British', 1903),
('J.K.', 'Rowling', 'British', 1965),
('Stephen', 'King', 'American', 1947),
('Agatha', 'Christie', 'British', 1990),
('Haruki', 'Murakami', 'Japanese', 1949),
('Jane', 'Austen', 'British', 1985),
('Ernest', 'Hemingway', 'American', 1999),
('Gabriel', 'Garcia Marquez', 'Colombian', 1927),
('J.R.R.', 'Tolkien', 'British', 1992),
('Leo', 'Tolstoy', 'Russian', 1928);
INSERT INTO books (title, isbn, publication_year, author_id, genre) VALUES
-- George Orwell books
('1984', '978-0451524935', 1949, 1, 'Dystopian'),
('Animal Farm', '978-0451526342', 1945, 1, 'Political Satire'),

-- J.K. Rowling books
('Harry Potter and the Sorcerer''s Stone', '978-0439708180', 1997, 2, 'Fantasy'),
('Harry Potter and the Chamber of Secrets', '978-0439064873', 1998, 2, 'Fantasy'),

-- Stephen King books
('The Shining', '978-0307743657', 1977, 3, 'Horror'),
('IT', '978-1501142970', 1986, 3, 'Horror'),
('The Stand', '978-0307743688', 1978, 3, 'Post-Apocalyptic'),

-- Agatha Christie books
('Murder on the Orient Express', '978-0062693662', 1934, 4, 'Mystery'),
('And Then There Were None', '978-0062073488', 1939, 4, 'Mystery'),

-- Haruki Murakami books
('Norwegian Wood', '978-0375704024', 1987, 5, 'Fiction'),
('Kafka on the Shore', '978-1400079278', 2002, 5, 'Magical Realism'),

-- Jane Austen books
('Pride and Prejudice', '978-0141439518', 1813, 6, 'Romance'),
('Sense and Sensibility', '978-0141439662', 1811, 6, 'Romance'),

-- Ernest Hemingway books
('The Old Man and the Sea', '978-0684801223', 1952, 7, 'Fiction'),
('A Farewell to Arms', '978-0684801469', 1929, 7, 'War Novel'),

-- Gabriel Garcia Marquez books
('One Hundred Years of Solitude', '978-0060883287', 1967, 8, 'Magical Realism'),
('Love in the Time of Cholera', '978-0307389732', 1985, 8, 'Romance'),

-- J.R.R. Tolkien books
('The Lord of the Rings', '978-0544003415', 1954, 9, 'Fantasy'),
('The Hobbit', '978-0547928227', 1937, 9, 'Fantasy'),

-- Leo Tolstoy books
('War and Peace', '978-1400079988', 1869, 10, 'Historical Fiction'),
('Anna Karenina', '978-0143035008', 1877, 10, 'Romance');
INSERT INTO members (first_name, last_name, email, phone_number, date_of_birth, membership_date) VALUES
('John', 'Smith', 'john.smith@email.com', '555-0101', '1990-05-15', '2023-01-10'),
('Sarah', 'Johnson', 'sarah.johnson@email.com', '555-0102', '1985-08-22', '2023-01-12'),
('Michael', 'Brown', 'michael.brown@email.com', '555-0103', '1992-12-03', '2023-01-15'),
('Emily', 'Davis', 'emily.davis@email.com', '555-0104', '1988-03-18', '2023-02-01'),
('David', 'Wilson', 'david.wilson@email.com', '555-0105', '1995-07-30', '2023-02-05'),
('Jennifer', 'Miller', 'jennifer.miller@email.com', '555-0106', '1991-11-12', '2023-02-10'),
('Robert', 'Taylor', 'robert.taylor@email.com', '555-0107', '1987-04-25', '2023-02-15'),
('Lisa', 'Anderson', 'lisa.anderson@email.com', '555-0108', '1993-09-08', '2023-03-01'),
('Christopher', 'Thomas', 'chris.thomas@email.com', '555-0109', '1989-01-20', '2023-03-05'),
('Amanda', 'Jackson', 'amanda.jackson@email.com', '555-0110', '1994-06-14', '2023-03-10'),
('Daniel', 'White', 'daniel.white@email.com', '555-0111', '1990-10-28', '2023-03-15'),
('Michelle', 'Harris', 'michelle.harris@email.com', '555-0112', '1986-02-09', '2023-04-01'),
('Kevin', 'Martin', 'kevin.martin@email.com', '555-0113', '1992-08-17', '2023-04-05'),
('Stephanie', 'Thompson', 'stephanie.thompson@email.com', '555-0114', '1988-12-05', '2023-04-10'),
('James', 'Garcia', 'james.garcia@email.com', '555-0115', '1995-04-19', '2023-04-15');
INSERT INTO book_copies (book_id, acquisition_date, status) VALUES
-- Multiple copies of popular books
(1, '2023-01-15', 'Available'),  -- 1984 Copy 1
(1, '2023-02-20', 'Loaned'),     -- 1984 Copy 2
(1, '2023-03-10', 'Available'),  -- 1984 Copy 3

(3, '2023-01-20', 'Loaned'),     -- Harry Potter 1 Copy 1
(3, '2023-02-15', 'Available'),  -- Harry Potter 1 Copy 2

(4, '2023-01-25', 'Available'),  -- Harry Potter 2 Copy 1
(4, '2023-03-05', 'Under Maintenance'), -- Harry Potter 2 Copy 2

(5, '2023-02-01', 'Loaned'),     -- The Shining Copy 1
(5, '2023-03-12', 'Available'),  -- The Shining Copy 2

(6, '2023-02-10', 'Available'),  -- IT Copy 1
(6, '2023-03-18', 'Available'),  -- IT Copy 2

(8, '2023-02-15', 'Loaned'),     -- Murder on Orient Express Copy 1
(8, '2023-03-20', 'Available'),  -- Murder on Orient Express Copy 2

(10, '2023-02-20', 'Available'), -- Norwegian Wood Copy 1
(11, '2023-02-25', 'Loaned'),    -- Kafka on Shore Copy 1

(12, '2023-03-01', 'Available'), -- Pride Prejudice Copy 1
(12, '2023-03-15', 'Available'), -- Pride Prejudice Copy 2

(13, '2023-03-05', 'Loaned'),    -- Sense Sensibility Copy 1
(14, '2023-03-08', 'Available'), -- Old Man Sea Copy 1

(15, '2023-03-10', 'Available'), -- Farewell Arms Copy 1
(16, '2023-03-12', 'Loaned'),    -- 100 Years Solitude Copy 1
(16, '2023-03-25', 'Available'), -- 100 Years Solitude Copy 2

(17, '2023-03-15', 'Available'), -- Love Cholera Copy 1
(18, '2023-03-18', 'Available'), -- Lord Rings Copy 1
(18, '2023-04-01', 'Under Maintenance'), -- Lord Rings Copy 2

(19, '2023-03-20', 'Loaned'),    -- Hobbit Copy 1
(19, '2023-04-05', 'Available'), -- Hobbit Copy 2

(20, '2023-03-22', 'Available'), -- War Peace Copy 1
(20, '2023-04-08', 'Available'), -- War Peace Copy 2

(2, '2023-03-25', 'Available'),  -- Animal Farm Copy 1
(7, '2023-03-28', 'Available'),  -- The Stand Copy 1
(9, '2023-04-02', 'Available');  -- And Then None Copy 1
INSERT INTO book_loans (copy_id, member_id, loan_date, due_date, return_date) VALUES
-- Returned loans
(2, 1, '2024-01-10', '2024-01-24', '2024-01-22'),  -- John Smith returned 1984
(4, 2, '2024-01-12', '2024-01-26', '2024-01-25'),  -- Sarah Johnson returned Harry Potter 1
(8, 3, '2024-01-15', '2024-01-29', '2024-01-28'),  -- Michael Brown returned The Shining
(12, 4, '2024-02-01', '2024-02-15', '2024-02-14'), -- Emily Davis returned Murder Express
(14, 5, '2024-02-05', '2024-02-19', '2024-02-18'), -- David Wilson returned Kafka
(17, 6, '2024-02-10', '2024-02-24', '2024-02-23'), -- Jennifer Miller returned Sense Sensibility
(19, 7, '2024-02-15', '2024-02-29', '2024-02-28'), -- Robert Taylor returned Farewell Arms
(21, 8, '2024-03-01', '2024-03-15', '2024-03-14'), -- Lisa Anderson returned 100 Years Solitude
(24, 9, '2024-03-05', '2024-03-19', '2024-03-18'), -- Christopher Thomas returned Hobbit

-- Active loans (not returned yet)
(2, 10, '2024-03-20', '2024-04-03', NULL),  -- Amanda Jackson has 1984
(4, 11, '2024-03-22', '2024-04-05', NULL),  -- Daniel White has Harry Potter 1
(8, 12, '2024-03-25', '2024-04-08', NULL),  -- Michelle Harris has The Shining
(12, 13, '2024-03-28', '2024-04-11', NULL), -- Kevin Martin has Murder Express
(14, 14, '2024-04-01', '2024-04-15', NULL), -- Stephanie Thompson has Kafka
(17, 15, '2024-04-03', '2024-04-17', NULL), -- James Garcia has Sense Sensibility

-- More returned loans for history
(1, 3, '2023-12-01', '2023-12-15', '2023-12-14'),
(3, 5, '2023-12-05', '2023-12-19', '2023-12-18'),
(5, 7, '2023-12-10', '2023-12-24', '2023-12-23'),
(7, 9, '2023-12-15', '2023-12-29', '2023-12-28'),
(9, 11, '2023-12-20', '2024-01-03', '2024-01-02'),
(11, 13, '2024-01-05', '2024-01-19', '2024-01-18'),
(13, 15, '2024-01-08', '2024-01-22', '2024-01-21'),
(15, 2, '2024-01-15', '2024-01-29', '2024-01-28'),
(18, 4, '2024-01-20', '2024-02-03', '2024-02-02');
-- Check all available books
SELECT b.title, a.first_name, a.last_name, bc.copy_id 
FROM books b 
JOIN authors a ON b.author_id = a.author_id 
JOIN book_copies bc ON b.book_id = bc.book_id 
WHERE bc.status = 'Available';

-- Find currently borrowed books and who has them
SELECT m.first_name, m.last_name, b.title, bl.loan_date, bl.due_date 
FROM book_loans bl 
JOIN members m ON bl.member_id = m.member_id 
JOIN book_copies bc ON bl.copy_id = bc.copy_id 
JOIN books b ON bc.book_id = b.book_id 
WHERE bl.return_date IS NULL;

-- Count books by author
SELECT a.first_name, a.last_name, COUNT(b.book_id) as book_count 
FROM authors a 
LEFT JOIN books b ON a.author_id = b.author_id 
GROUP BY a.author_id;

-- Find overdue books
SELECT m.first_name, m.last_name, b.title, bl.due_date 
FROM book_loans bl 
JOIN members m ON bl.member_id = m.member_id 
JOIN book_copies bc ON bl.copy_id = bc.copy_id 
JOIN books b ON bc.book_id = b.book_id 
WHERE bl.return_date IS NULL AND bl.due_date < CURDATE();
DESCRIBE books;
-- First, we need to temporarily remove the foreign key constraints
ALTER TABLE book_copies DROP FOREIGN KEY book_copies_ibfk_1;

-- Modify the publication_year column to INT
ALTER TABLE books MODIFY publication_year INT;

-- Re-add the foreign key constraint
ALTER TABLE book_copies ADD FOREIGN KEY (book_id) REFERENCES books(book_id) ON DELETE CASCADE;
