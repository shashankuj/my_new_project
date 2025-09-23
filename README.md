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


--Task 2 Data Insertion and Handling Nulls
create database reservers1;
use reservers1;

CREATE TABLE sailors(
    sid INT PRIMARY KEY,
    sname VARCHAR(30) NOT NULL,
    rating INT DEFAULT 0,  -- Default value for missing ratings
    age FLOAT DEFAULT 18.0 -- Default value for missing age
);

CREATE TABLE boats(
    bid INT PRIMARY KEY,
    bname VARCHAR(30) NOT NULL,
    color VARCHAR(30) DEFAULT 'white' -- Default value for missing color
);

CREATE TABLE reserves(
    sid INT,
    bid INT,
    days DATE DEFAULT (CURRENT_DATE),
    FOREIGN KEY(sid) REFERENCES sailors(sid),
    FOREIGN KEY(bid) REFERENCES boats(bid)
);

-- Insert sailors with some NULL values
INSERT INTO sailors VALUES(22, 'dustin', 7, 45.0);
INSERT INTO sailors VALUES(29, 'brutus', NULL, 33.0);  -- NULL rating
INSERT INTO sailors VALUES(31, 'lubber', 8, 55.5);
INSERT INTO sailors VALUES(32, 'andy', 8, NULL);       -- NULL age
INSERT INTO sailors VALUES(58, 'rusty', 10, 35.0);
INSERT INTO sailors VALUES(64, 'horatio', 7, 35.0);
INSERT INTO sailors VALUES(71, 'zorba', 10, 16.0);
INSERT INTO sailors VALUES(74, 'haratio', 9, 35.0);
INSERT INTO sailors VALUES(85, 'art', 3, 25.5);
INSERT INTO sailors VALUES(95, 'bob', NULL, NULL);     -- NULL rating and age

-- Insert boats with some NULL values
INSERT INTO boats VALUES(101, 'interlake', 'blue');
INSERT INTO boats VALUES(102, 'interlake', NULL);      -- NULL color
INSERT INTO boats VALUES(103, 'clipper', 'green');
INSERT INTO boats VALUES(104, 'marine', 'red');
INSERT INTO boats VALUES(105, 'voyager', DEFAULT);     -- Use default color

-- Insert reserves with some NULL values
INSERT INTO reserves VALUES(22, 101, '1998-10-10');
INSERT INTO reserves VALUES(64, 101, NULL);            -- NULL date
INSERT INTO reserves VALUES(22, 102, '1998-10-10');
INSERT INTO reserves VALUES(31, 102, '1998-10-11');
INSERT INTO reserves VALUES(64, 102, DEFAULT);         -- Use default date
INSERT INTO reserves VALUES(22, 103, '1998-08-10');
INSERT INTO reserves VALUES(31, 103, '1998-06-11');
INSERT INTO reserves VALUES(74, 103, '1998-08-09');
INSERT INTO reserves VALUES(22, 104, '1998-07-10');
INSERT INTO reserves VALUES(31, 104, NULL);            -- NULL date
update sailors set age=46.0 where sid=22;
update sailors set rating=2,age=34.0 where sid=29;
delete from sailors s where s.sid not in (select r.sid from reserves r);
