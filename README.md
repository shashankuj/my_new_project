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

