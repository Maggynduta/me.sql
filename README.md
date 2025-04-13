# me.sql
me.sql
-- Step 1: Create the database
CREATE DATABASE IF NOT EXISTS BookstoreDB;
USE BookstoreDB;

-- Step 2: Create Authors table
CREATE TABLE IF NOT EXISTS Authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    bio TEXT
);

-- Step 3: Create Books table
CREATE TABLE IF NOT EXISTS Books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    genre VARCHAR(100),
    price DECIMAL(10, 2),
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES Authors(author_id)
);

-- Step 4: Create Customers table
CREATE TABLE IF NOT EXISTS Customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    email VARCHAR(150) UNIQUE,
    phone_number VARCHAR(20),
    address TEXT
);

-- Step 5: Create Orders table
CREATE TABLE IF NOT EXISTS Orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

-- Step 6: Create OrderDetails table (junction table)
CREATE TABLE IF NOT EXISTS OrderDetails (
    order_detail_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    book_id INT,
    quantity INT,
    price DECIMAL(10, 2),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (book_id) REFERENCES Books(book_id)
);

-- Step 7: Create Shipping table
CREATE TABLE IF NOT EXISTS Shipping (
    shipping_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT,
    shipping_address TEXT,
    shipping_date DATE,
    delivery_status VARCHAR(50),
    FOREIGN KEY (order_id) REFERENCES Orders(order_id)
);

-- Step 8: Create users and roles
-- Create an admin user with full access
CREATE USER IF NOT EXISTS 'admin_user'@'localhost' IDENTIFIED BY 'AdminPass123!';
GRANT ALL PRIVILEGES ON BookstoreDB.* TO 'admin_user'@'localhost';

-- Create a staff user with limited access (e.g., can insert, update but not drop tables)
CREATE USER IF NOT EXISTS 'staff_user'@'localhost' IDENTIFIED BY 'StaffPass123!';
GRANT SELECT, INSERT, UPDATE ON BookstoreDB.* TO 'staff_user'@'localhost';

-- Create a viewer user with read-only access
CREATE USER IF NOT EXISTS 'viewer_user'@'localhost' IDENTIFIED BY 'ViewPass123!';
GRANT SELECT ON BookstoreDB.* TO 'viewer_user'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;

-- Step 9: Insert sample data

-- Insert Authors
INSERT INTO Authors (first_name, last_name, bio) VALUES
('George', 'Orwell', 'British writer and journalist.'),
('J.K.', 'Rowling', 'Author of the Harry Potter series.');

-- Insert Books
INSERT INTO Books (title, genre, price, author_id) VALUES
('1984', 'Dystopian', 15.99, 1),
('Harry Potter and the Sorcerer\'s Stone', 'Fantasy', 25.50, 2);

-- Insert Customers
INSERT INTO Customers (first_name, last_name, email, phone_number, address) VALUES
('Alice', 'Smith', 'alice@example.com', '0712345678', '123 Nairobi Rd'),
('Bob', 'Jones', 'bob@example.com', '0723456789', '456 Mombasa Rd');

-- Insert Orders
INSERT INTO Orders (customer_id, order_date, total_amount) VALUES
(1, '2025-04-10', 15.99),
(2, '2025-04-11', 25.50);

-- Insert OrderDetails
INSERT INTO OrderDetails (order_id, book_id, quantity, price) VALUES
(1, 1, 1, 15.99),
(2, 2, 1, 25.50);

-- Insert Shipping info
INSERT INTO Shipping (order_id, shipping_address, shipping_date, delivery_status) VALUES
(1, '123 Nairobi Rd', '2025-04-12', 'Delivered'),
(2, '456 Mombasa Rd', '2025-04-13', 'Pending');

-- Step 10: Sample Queries

-- 1. List all books with their authors
SELECT b.title, b.genre, b.price, CONCAT(a.first_name, ' ', a.last_name) AS author
FROM Books b
JOIN Authors a ON b.author_id = a.author_id;

-- 2. Find all orders by a specific customer
SELECT o.order_id, o.order_date, o.total_amount
FROM Orders o
JOIN Customers c ON o.customer_id = c.customer_id
WHERE c.email = 'alice@example.com';

-- 3. Get total number of books sold and revenue
SELECT SUM(od.quantity) AS total_books_sold, SUM(od.price * od.quantity) AS total_revenue
FROM OrderDetails od;

-- 4. Get all pending shipments
SELECT s.order_id, s.shipping_address, s.shipping_date
FROM Shipping s
WHERE s.delivery_status = 'Pending';

-- 5. Get top 1 customer by total amount spent
SELECT c.first_name, c.last_name, SUM(o.total_amount) AS total_spent
FROM Customers c
JOIN Orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id
ORDER BY total_spent DESC
LIMIT 1;
