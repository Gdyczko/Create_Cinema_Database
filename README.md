# Create_Cinema_Database

Description of the database:

The database has been created for the purpose of analyzing and efficiently managing the business of a cinema. It collects information about movies and their genres, directors, sold tickets, types of halls, as well as employees and their leaves. This enables us to possess and analyze various pieces of information, such as which type of movie (genre) is most frequently watched, whether the duration of the film matters in the selection process, whether the director's surname matters, which employee sells the most tickets, in which hall the most movies have been screened, etc. In general, with the help of this database, we are able to determine the level of cinema functioning, as well as better understand the needs of customers and adjust the services provided accordingly to their needs.

## Creating tables:
1. Movies - This table contains information about movies that have been, are, or will be screened in the cinema. It includes details such as their titles, durations, release dates, as well as the start and end dates of the movie's screening in the cinema.
```sql
CREATE TABLE Movies (
    Movie_Id INT PRIMARY KEY,
    Title VARCHAR(50) NOT NULL,
    Duration TIME,
    Release_Date DATE,
    Screening_Start_Date DATE,
    Screening_End_Date DATE
)
```
2. Directors - The table stores basic information about directors, including their first names, last names, and dates of birth.
```sql
CREATE TABLE Directors (
    Director_Id INT PRIMARY KEY,
    First_Name VARCHAR(50),
    Second_Name VARCHAR(50),
    Surname VARCHAR(50),
    Date_of_Birth DATE
)
```
3. Directors_Movies - This table connects the table with movie information to the table with director information using a foreign key. This allows us, with the appropriate query, to assign a specific movie to a specific director without duplicating the same director information across multiple movies.
```sql
CREATE TABLE Directors_Movies (
    Director_Id INT NOT NULL REFERENCES Directors (Director_Id),
    Movie_Id INT NOT NULL REFERENCES Movies (Movie_Id),
    PRIMARY KEY (Director_Id, Movie_Id)
)
```
4. Genres - The table stores information about existing movie genres.
```sql
CREATE TABLE Genres (
    Genre_Id INT PRIMARY KEY,
    Genre_Name VARCHAR(50) NOT NULL
)
```
5. Movies_Genres - The table assigns appropriate genres to respective movies, utilizing primary keys from the Movies table and the Genres table, and stores them in this table as foreign keys. The combination of these two keys forms the primary key for this table (the primary key consists of 2 foreign keys).
```sql
CREATE TABLE Movies_Genres (
    Movie_Id INT NOT NULL,
    Genre_Id INT NOT NULL,
    PRIMARY KEY (Movie_Id, Genre_Id),
    FOREIGN KEY (Movie_Id) REFERENCES Movies (Movie_Id),
    FOREIGN KEY (Genre_Id) REFERENCES Genres (Genre_Id)
)
```
6. Tickets - The table contains information about the ticket number, its price, as well as details about the row and seat it was sold for. Additionally, a CHECK constraint has been added to this table to prevent an employee from entering a price below 10 PLN (we do not sell tickets cheaper than this because it is not profitable), and to ensure that the seat and row are entered correctly (neither the seat nor the row can have values such as 0, as there is no seat or row with such a number in the cinema).
```sql
CREATE TABLE Tickets (
    Ticket_Id INT PRIMARY KEY,
    Price DECIMAL NOT NULL CHECK (Price > 10.00),
    Row_Number INT NOT NULL CHECK (Row_Number > 0),
    Seat_Number INT NOT NULL CHECK (Seat_Number > 0)
)
```
7. Employees - The table contains information about employees: first name, last name, date of birth with a CHECK constraint to hire only adults in 2023. The table also includes information about the position held and the salary of employees.
```sql
CREATE TABLE Employees (
    Employee_Id INT PRIMARY KEY,
    First_Name VARCHAR(50) NOT NULL,
    Surname VARCHAR(50) NOT NULL,
    Date_of_Birth DATE CHECK (YEAR(Date_of_Birth) < 2004),
    Position VARCHAR(50),
    Salary DECIMAL CHECK (Salary BETWEEN 1000 AND 10000)
)
```
8. Rooms - The table contains information about the room number, the number of seats in the room, and the type of the room (whether it is for 3D movies, IMAX, etc.).
```sql
CREATE TABLE Rooms (
    Room_Id INT PRIMARY KEY,
    Number_of_Seats INT NOT NULL,
    Room_Type VARCHAR(50)
)
```
9.  Employees_Tickets - This table contains information about employees and the tickets sold by them. The table utilizes foreign keys, which together form the primary key for this table.
```sql
CREATE TABLE Employees_Tickets (
    Employee_Id INT NOT NULL,
    Ticket_Id INT NOT NULL,
    PRIMARY KEY (Employee_Id, Ticket_Id),
    FOREIGN KEY (Employee_Id) REFERENCES Employees(Employee_Id),
    FOREIGN KEY (Ticket_Id) REFERENCES Tickets(Ticket_Id)
)
```
10.  Rooms_Movies - This table contains information about which movie is screened in which room. The table is built with foreign keys, which together form the primary key for this table.
```sql
CREATE TABLE Rooms_Movies (
    Room_Id INT NOT NULL,
    Movie_Id INT NOT NULL,
    PRIMARY KEY (Room_Id, Movie_Id),
    FOREIGN KEY (Room_Id) REFERENCES Rooms(Room_Id),
    FOREIGN KEY (Movie_Id) REFERENCES Movies(Movie_Id)
)
```
11.  Tickets_Rooms - This table contains information about which ticket was sold for which room. The table is built with foreign keys, and the primary key for this table is a combination of these foreign keys.
```sql
CREATE TABLE Tickets_Rooms (
    Ticket_Id INT NOT NULL,
    Room_Id INT NOT NULL,
    PRIMARY KEY (Ticket_Id, Room_Id),
    FOREIGN KEY (Ticket_Id) REFERENCES Tickets(Ticket_Id),
    FOREIGN KEY (Room_Id) REFERENCES Rooms(Room_Id)
)
```
12.  Vacations - The table contains information about employees' vacations: which employee is going on vacation, who will substitute for them, the vacation dates, and the type of vacation.
```sql
CREATE TABLE Vacations (
    Vacation_Id INT PRIMARY KEY,
    Employee_Id INT REFERENCES Employees(Employee_Id),
    Vacation_Type NVARCHAR(50),
    Start_Date DATE,
    End_Date DATE,
    Substitute_Id INT REFERENCES Employees(Employee_Id)
)
```
## Views:
1. This view will display the names of genres alongside the count of movies belonging to each genre, which have a duration of less than 2 hours.
```sql
CREATE VIEW [Movies_Duration_Less_Than_2_Hours]
AS
SELECT G.Genre_Name,
       COUNT(*) AS [Number_of_Movies]
FROM Movies AS M
JOIN Movies_Genres AS MG ON (M.Movie_Id = MG.Movie_Id)
JOIN Genres AS G ON (MG.Genre_Id = G.Genre_Id)
WHERE M.Duration < '02:00:00'
GROUP BY G.Genre_Name
GO
```
2. This view will display the first name and last name of the director along with the title of their movie, which has not been screened at the analyzed cinema.
```sql
CREATE VIEW [Directors_Whose_Movie_Was_Not_Screened]
AS
SELECT D.First_Name,
       D.Surname,
       M.Title
FROM Movies AS M
JOIN Directors_Movies AS DM ON (M.Movie_Id = DM.Movie_Id)
JOIN Directors AS D ON (DM.Director_Id = D.Director_Id)
WHERE M.Screening_Start_Date IS NULL
AND M.Screening_End_Date IS NULL
GO
```
3. This view will display the last names of directors who were under 40 years old when their movie premiered. The view will also display the titles of these movies.
Sure, here's the revised SQL code using English aliases:
```sql
CREATE VIEW [Directors_Under_40_At_Premiere]
AS
SELECT D.First_Name,
       D.Surname,
       M.Title,
       D.Date_of_Birth,
       M.Release_Date
FROM Directors AS D
JOIN Directors_Movies AS DM ON (D.Director_Id = DM.Director_Id)
JOIN Movies AS M ON (DM.Movie_Id = M.Movie_Id)
WHERE DATEDIFF(YEAR, D.Date_of_Birth, M.Release_Date) < 40
GO
```
4.This view will display information about the average duration of movies that premiered in 2021. The duration will be shown in minutes.
```sql
CREATE VIEW [Average_Duration_of_2021_Movies_Sold_Tickets]
AS
SELECT AVG(DATEDIFF(MINUTE, '00:00:00', Duration)) AS [Average_duration_of_movies_in_minutes]
FROM Movies
WHERE YEAR(Release_Date) = 2021
GO
```
5. This view will display the first name, last name, and position of the employee along with the titles of movies for which the employee sold tickets.
```sql
CREATE VIEW [Employee_Movies_Sold_Tickets]
AS
SELECT E.First_Name,
       E.Surname,
       E.Position,
       M.Title
FROM Employees AS E
JOIN Employees_Tickets AS ET ON (E.Employee_Id = ET.Employee_Id)
JOIN Tickets_Rooms AS TR ON (ET.Ticket_Id = TR.Ticket_Id)
JOIN Rooms_Movies AS RM ON (TR.Room_Id = RM.Room_Id)
JOIN Movies AS M ON (RM.Movie_Id = M.Movie_Id)
GO
```
6. The following view will display information about room types - their categories (IMAX, 3D, VIP, etc.) and how many rooms of each type are in a particular cinema.
```sql
CREATE VIEW [Room_Types_in_a_Cinema]
AS
SELECT Room_Type,
       COUNT(*) AS [Number_of_rooms_of_each_type]
FROM Rooms
GROUP BY (Room_Type)
GO
```
## Procedures:
1. Here's the procedure "P_Add_New_Movie" that adds a new movie and its attributes to the "Movies" table, provided that a movie with the given ID and title does not already exist in the table. Otherwise, it sends a message to the user indicating that the movie already exists in the database.

```sql
CREATE PROC P_Add_New_Movie (
    @Movie_Id INT,
    @Movie_Title VARCHAR(50),
    @Duration TIME,
    @Release_Date DATE,
    @Screening_Start_Date DATE,
    @Screening_End_Date DATE
)
AS
IF (NOT EXISTS (
    SELECT *
    FROM Movies
    WHERE Movie_Id = @Movie_Id AND Movie_Title = @Movie_Title
))
INSERT INTO Movies VALUES (
    @Movie_Id,
    @Movie_Title,
    @Duration,
    @Release_Date,
    @Screening_Start_Date,
    @Screening_End_Date
)
ELSE BEGIN
    DECLARE @Message NVARCHAR(256)
    SET @Message = FORMATMESSAGE('This movie already exists in the database.')
    PRINT @Message
END
GO
```
2. The following procedure adds a new employee, provided that such an employee
does not already exist in the database.
```sql
CREATE PROC P_Add_New_Employee (
    @Employee_Id INT,
    @First_Name VARCHAR(50),
    @Surname VARCHAR(50),
    @Date_of_Birth INT,
    @Position VARCHAR(50),
    @Salary DECIMAL
)
AS
BEGIN
    IF NOT EXISTS (
        SELECT *
        FROM Employees
        WHERE Employee_Id = @Employee_Id
    )
    BEGIN
        INSERT INTO Employees (Employee_Id, First_Name, Surname,
        Date_of_Birth, Position, Salary)
        VALUES (@Employee_Id, @First_Name, @Surname,
        @Date_of_Birth, @Position, @Salary)

        PRINT 'New employee has been added.'
    END
    ELSE
    BEGIN
        PRINT 'Employee with the specified ID already exists.'
    END
END
GO
```
3. The following procedure, upon the user providing a movie title, displays the number of days the movie was screened in the cinema via a message. In the case of a movie that was not screened or does not have specified start or end dates for screening, the user receives a message: "The number of days cannot be provided for the given movie title".
```sql
CREATE PROCEDURE P_Movie_Screening_Duration
    @Movie_Title VARCHAR(50)
AS
BEGIN
    DECLARE @Days_Count INT;
    
    SELECT @Days_Count = DATEDIFF(DAY, Screening_Start_Date, Screening_End_Date)
    FROM Movies
    WHERE Title = @Movie_Title;

    IF @Days_Count IS NOT NULL
    BEGIN
        PRINT 'The movie "' + @Movie_Title + '" was screened for ' + CAST(@Days_Count AS VARCHAR(10)) + ' days.';
    END
    ELSE
    BEGIN
        PRINT 'The number of days cannot be provided for the given movie title.';
    END
END
GO
```
## Functions:
1. The following function displays the first names, last names, positions, and salaries of employees who earn above the value provided by the user.
```sql
CREATE FUNCTION F_Employees_Salaries (@MinAmount MONEY) RETURNS TABLE AS
RETURN (
    SELECT First_Name
        , Surname
        , Position
        , Salary
    FROM Employees
    WHERE Salary > @MinAmount
)
GO
```
2. The following function will display the number of tickets sold for a specific type of room provided by the user.
```sql
CREATE FUNCTION F_Number_of_Sold_Tickets (
    @Room_Type VARCHAR(50)
)
RETURNS @Result TABLE (
    [Room_Type] VARCHAR(50),
    [Number_of_Sold_Tickets] INT
)
AS
BEGIN
    INSERT INTO @Result
    SELECT R.Room_Type,
           COUNT(*)
    FROM Rooms AS R
    JOIN Tickets_Rooms AS TR ON (R.Room_Id = TR.Room_Id)
    WHERE Room_Type = @Room_Type
    GROUP BY R.Room_Type
RETURN
END
GO
```
3. This function returns a table with movie titles and their premiere dates for the director whose last name is provided by the user.
```sql
CREATE FUNCTION F_Movies_and_Premiere_Dates (@Director_Surname VARCHAR(50)) 
RETURNS TABLE 
AS 
RETURN (
    SELECT M.Title
         , M.Release_Date
         , D.Surname
    FROM Movies AS M
    JOIN Directors_Movies AS DM ON (DM.Movie_Id = M.Movie_Id)
    JOIN Directors AS D ON (D.Director_Id = DM.Director_Id)
    WHERE D.Surname = @Director_Surname
)
GO
```
