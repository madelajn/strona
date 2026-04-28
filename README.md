1. Wyświetl wszystkie osoby zatrudnione w maju i zarabiające powyżej 5000. Posortuj wyniki alfabetycznie po nazwisku.

SELECT *
FROM employees
WHERE EXTRACT(MONTH FROM hire_date) = 5
  AND salary > 5000
ORDER BY last_name;



2. Dla każdego stanowiska wyświetl maksymalne zarobki pracowników na nim zatrudnionych. Pokaż tylko stanowiska Accountant, Public Accountant, Purchasing Manager, Purchasing Clerk, Stock Manager, Stock Clerk. Nie używaj warunku AND.

SELECT j.job_title, MAX(e.salary)
FROM employees e
JOIN jobs j ON e.job_id = j.job_id
WHERE j.job_title IN ('Accountant', 'Public Accountant', 'Purchasing Manager', 
                      'Purchasing Clerk', 'Stock Manager', 'Stock Clerk')
GROUP BY j.job_title;



3. Wyświetl nazwy działów, które zatrudniają pracowników.

SELECT DISTINCT d.department_name
FROM departments d
JOIN employees e ON d.department_id = e.department_id;



4. Wyświetl wszystkich pracowników zatrudnionych w ciągu 3 miesięcy od założenia firmy (za datę założenia firmy przyjmij najwcześniejszą datę zatrudnienia pracownika).

SELECT *
FROM employees
WHERE hire_date <= (SELECT ADD_MONTHS(MIN(hire_date), 3) FROM employees);


5. Wskaż prezesa firmy.

SELECT *
FROM employees
WHERE manager_id IS NULL;


6. Dodaj nowy dział Advertisements znajdujący się we Wrocławiu i przenieś do niego wszystkich pracowników z działów Public Relations oraz Marketing.

-- 1. Dodanie lokalizacji Wrocław (musi mieć unikalne ID, np. 3300)
INSERT INTO locations (location_id, city, country_id) 
VALUES (3300, 'Wrocław', 'PL');

-- 2. Dodanie działu Advertisements przypisanego do nowej lokalizacji
INSERT INTO departments (department_id, department_name, location_id)
VALUES (280, 'Advertisements', 3300);

-- 3. Przeniesienie pracowników z PR i Marketingu do nowego działu
UPDATE employees
SET department_id = 280
WHERE department_id IN (
    SELECT department_id 
    FROM departments 
    WHERE department_name IN ('Public Relations', 'Marketing')
);


7. Stwórz tabelę rooms, o polach room_id, location_id, room_type, number_of_seats. Pole room_id ma być kluczem głównym, a pole location_id kluczem obcym wskazującym odpowiednie pole z tabeli locations. Ustaw domyślną wartość pola room_type na office oraz sprawdź, czy wpisywane wartości pola number_of_seats nie są ujemne.

CREATE TABLE ROOMS (
    ROOM_ID NUMBER,
    LOCATION_ID NUMBER(4),
    ROOM_TYPE VARCHAR2(50) DEFAULT 'office',
    NUMBER_OF_SEATS NUMBER,
    -- Definicja klucza głównego
    CONSTRAINT pk_rooms PRIMARY KEY (ROOM_ID),
    -- Definicja klucza obcego
    CONSTRAINT fk_rooms_locations FOREIGN KEY (LOCATION_ID) 
        REFERENCES LOCATIONS (LOCATION_ID),
    -- Warunek sprawdzający (nieujemna liczba miejsc)
    CONSTRAINT check_seats_positive CHECK (NUMBER_OF_SEATS >= 0)
);



------------------------------------------

1. Wyswietl wszystkie działy znajdujące się w Europie.

SELECT d.department_name
FROM departments d
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
JOIN regions r ON c.region_id = r.region_id
WHERE r.region_name = 'Europe';


2.  Wyświetl imię, nazwisko i zarobki każdego pracownika, który został zatrudniony wcześniej niż swój manager. Posortuj wyniki alfabetycznie po nazwisku.

SELECT e.first_name, e.last_name, e.salary
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.hire_date < m.hire_date
ORDER BY e.last_name;


3.  Wyświetl wszystkie osoby, które zostały zatrudnione w ciągu 20 miesięcy od założenia firmy. Za datę założenia firmy przyjmij datę zatrudnienia pierwszego pracownika.

SELECT *
FROM employees
WHERE hire_date <= (SELECT ADD_MONTHS(MIN(hire_date), 20) FROM employees);


4.  Wyświetl osoby zarabiające najwięcej w swoim dziale, niebędące
managerami.

SELECT first_name, last_name, salary, department_id
FROM employees e1
WHERE salary = (
    SELECT MAX(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
)
AND employee_id NOT IN (
    SELECT manager_id 
    FROM employees 
    WHERE manager_id IS NOT NULL
);



5. Dodaj nowy dział University znajdujący się we Wrocławiu. Zatrudnij w nim z dniem dzisiejszym naszego rektora na takim samym stanowisku, co Steven King.

-- Krok 1: Dodanie lokalizacji Wrocław (jeśli nie dodałaś jej wcześniej)
INSERT INTO locations (location_id, city, country_id)
VALUES (3300, 'Wrocław', 'PL');

-- Krok 2: Dodanie działu University
INSERT INTO departments (department_id, department_name, location_id)
VALUES (300, 'University', 3300);

-- Krok 3: Zatrudnienie rektora
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, job_id, department_id, salary)
VALUES (
    999, 
    'Rektor', 
    'Magnificencja', 
    'REKTOR', 
    SYSDATE, 
    (SELECT job_id FROM employees WHERE last_name = 'King' AND first_name = 'Steven'), 
    300, 
    25000
);



6.  Dodaj do każdego maila pracownika końcówkę @gmail.com.

UPDATE employees
SET email = email || '@gmail.com';


7.  Utwórz tabelę big_departments o strukturze tabeli departments,  która będzie zawierała wszystkie działy mające więcej niż 3 zatrudnionych pracowników.

CREATE TABLE big_departments AS
SELECT *
FROM departments
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) > 3
);
