/* This code create the 
	necessary tables, index and triggers for the CallCenter Database */
	
--------------------------
-- Database  and Schema creation ⬇️⬇️
--------------------------

-- Create Database
DROP DATABASE IF EXISTS callcenter WITH (FORCE);
CREATE DATABASE callcenter;
--------------------------

-- Create Schema
DROP SCHEMA IF EXISTS public;
DROP SCHEMA IF EXISTS viz1;
CREATE SCHEMA IF NOT EXISTS viz1;
--------------------------

--------------------------
-- Main tables creations ⬇️⬇️
--------------------------

	-- Dimensions tables
-- Create calendar table
DROP TABLE IF EXISTS viz1.calendar;
CREATE TABLE viz1.calendar(
	idx SERIAL PRIMARY KEY NOT NULL UNIQUE,
	dates DATE NOT NULL UNIQUE,
	years INT NOT NULL,
	months INT NOT NULL,
	months_name VARCHAR(3) NOT NULL,
	days INT NOT NULL,
	days_week INT NOT NULL,
	days_name VARCHAR(3) NOT NULL,
	months_ends DATE NOT NULL,
	quarters INT NOT NULL,
	week_start DATE NOT NULL,
	week_end DATE NOT NULL,
	weeks VARCHAR(7) NOT NULL,
	week_year INT NOT NULL,
	month_year VARCHAR(6) NOT NULL,
	ini_month VARCHAR(1) NOT NULL,
  year_offset INT NOT NULL,
  month_offset INT NOT NULL,
  week_offset INT NOT NULL
);
-- Create Index on calendar table
DROP INDEX IF EXISTS ix_calendar;
CREATE INDEX ix_calendar
	ON viz1.calendar (idx,dates,years,months,name_months);
--------------------------...

-- Create categories table
DROP TABLE IF EXISTS viz1.category;
CREATE TABLE viz1.category(
    idx SERIAL PRIMARY KEY NOT NULL,
    category_name VARCHAR(15) NOT NULL UNIQUE
);
-- Index for category table
DROP INDEX IF EXISTS ix_categories;
CREATE INDEX ix_categories
	ON viz1.category (category_name);
--------------------------------------------

--Create Status table
DROP TABLE IF EXISTS viz1.status;
CREATE TABLE viz1.status(
    idx SERIAL PRIMARY KEY NOT NULL,
    status_type VARCHAR(11) NOT NULL UNIQUE
);
-- Index for Status table
DROP INDEX IF EXISTS ix_status;
CREATE INDEX ix_status
	ON viz1.status (status_type);
------------------------------------------

-- Create Agents table
DROP TABLE IF EXISTS viz1.agents;
CREATE TABLE viz1.agents(
    idx SERIAL NOT NULL UNIQUE,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    emp_code VARCHAR(25) NOT NULL PRIMARY KEY,
    lob VARCHAR(5) NOT NULL,
    hire_date BIGINT NOT NULL,
    acct_status BIGINT NOT NULL DEFAULT 1,
    FOREIGN KEY (hire_date) REFERENCES viz1.calendar(idx)
    ON UPDATE CASCADE ON DELETE RESTRICT,
    FOREIGN KEY (acct_status) REFERENCES viz1.status(idx)
    ON UPDATE CASCADE ON DELETE RESTRICT
);
-- Index for Agents table
DROP INDEX IF EXISTS ix_agents;
CREATE INDEX ix_agents
	ON viz1.agents (lob,emp_code,hire_date);
---------------------------------------------------

	-- Fact tables
-- CREATE absence/attendance TABLE
DROP TABLE IF EXISTS viz1.attendances;
CREATE TABLE viz1.attendances(
    date_idx BIGINT NOT NULL,
    emp_code VARCHAR(25) NOT NULL,
    lob VARCHAR(5) NOT NULL,
	attendance_status VARCHAR(25) NOT NULL,
    FOREIGN KEY (emp_code) REFERENCES viz1.agents(emp_code)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (date_idx) REFERENCES viz1.calendar(idx)
    ON UPDATE CASCADE ON DELETE RESTRICT
);
ALTER TABLE viz1.attendances DROP COLUMN lob;
-- Index for absence table
DROP INDEX IF EXISTS ix_absence;
CREATE INDEX ix_absence
	ON viz1.attendances (emp_code,date_idx);
-------------------------------------------

-- Create QA table
DROP TABLE IF EXISTS viz1.qa_score;
CREATE TABLE viz1.qa_score(
    date_idx BIGINT NOT NULL,
    emp_code VARCHAR(25) NOT NULL,
    score1 BIGINT NOT NULL CHECK(score1 >= 50 AND score1 <= 100),
    score2 BIGINT NOT NULL CHECK(score2 >= 50 AND score2 <= 100),
    FOREIGN KEY (emp_code) REFERENCES viz1.agents(emp_code)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (date_idx) REFERENCES viz1.calendar(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT
);
-- Index for QA table
DROP INDEX IF EXISTS ix_qa;
CREATE INDEX ix_qa
	ON viz1.qa_score (emp_code,date_idx);
----------------------------------------

-- CREATE Chat Table
DROP TABLE IF EXISTS viz1.wk_chats;
CREATE TABLE viz1.wk_chats (
    date_idx BIGINT NOT NULL,
    emp_code VARCHAR(25) NOT NULL,
	catg_id BIGINT NOT NULL,
    aht time without time zone NOT NULL,
    rating INT,
    FOREIGN KEY (emp_code) REFERENCES viz1.agents(emp_code)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (date_idx) REFERENCES viz1.calendar(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (catg_id) REFERENCES viz1.category(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT
);
-- Index for Chat table
DROP INDEX IF EXISTS ix_chat;
CREATE INDEX ix_chat
	ON viz1.wk_chats (emp_code,date_idx,catg_id);
---------------------------------

-- CREATE Phone Table
DROP TABLE IF EXISTS viz1.wk_phone;
CREATE TABLE viz1.wk_phone (
    date_idx BIGINT NOT NULL,
    emp_code VARCHAR(25) NOT NULL,
    catg_id BIGINT NOT NULL,
    aht time without time zone NOT NULL,
    rating INT,
    FOREIGN KEY (emp_code) REFERENCES viz1.agents(emp_code)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (date_idx) REFERENCES viz1.calendar(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (catg_id) REFERENCES viz1.category(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT
);
-- Index for Phone table
DROP INDEX IF EXISTS ix_phone;
CREATE INDEX ix_phone
	ON viz1.wk_phone (emp_code,date_idx,catg_id);
----------------------------------------

-- CREATE Email Table
DROP TABLE IF EXISTS viz1.wk_email;
CREATE TABLE viz1.wk_email (
    date_idx BIGINT NOT NULL,
    emp_code VARCHAR(25) NOT NULL,
    catg_id BIGINT NOT NULL,
    rating INT,
    FOREIGN KEY (emp_code) REFERENCES viz1.agents(emp_code)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (date_idx) REFERENCES viz1.calendar(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT,
    FOREIGN KEY (catg_id) REFERENCES viz1.category(idx)
    ON UPDATE RESTRICT ON DELETE RESTRICT
);
-- Index for Emails table
DROP INDEX IF EXISTS ix_email;
CREATE INDEX ix_email
	ON viz1.wk_email (emp_code,date_idx,catg_id,rating);
----------------------------------------

--------------------------
-- Functions and Triggers creations ⬇️⬇️
--------------------------

-- Create Functions to not mix agents to another department
DROP FUNCTION IF EXISTS viz1.fc_check_email_agents; --Check/change function name
CREATE OR REPLACE FUNCTION viz1.fc_check_email_agents() --Check/change function name
RETURNS TRIGGER 
AS $$
BEGIN
    IF NOT EXISTS (
        SELECT emp_code
        FROM viz1.agents
        WHERE emp_code = NEW.emp_code AND lob = 'Email' ---change department/lob
    ) THEN
        RAISE EXCEPTION 'Agent is not in the "Email" department'; --- change department message
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create Triggers to not mix agents to another department to each department
DROP TRIGGER IF EXISTS tg_email_agents --change name
ON viz1.wk_email; --change table name
CREATE OR REPLACE TRIGGER tg_email_agents --change name
    BEFORE INSERT OR UPDATE 
    ON viz1.wk_email --change table
    FOR EACH ROW
    EXECUTE FUNCTION viz1.fc_check_email_agents(); --change function

----------------------------------------
--Create Functions for deactivated agents
CREATE OR REPLACE FUNCTION viz1.fc_check_status_agents() 
RETURNS TRIGGER 
AS $$
BEGIN
    -- Check agent availability
    IF EXISTS (
        SELECT 1
        FROM viz1.agents AS ag
        WHERE ag.emp_code = NEW.emp_code AND ag.acct_status > 2
    ) THEN
        RAISE EXCEPTION 'Agent % is deactivated and cannot be used for production', NEW.emp_code;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create Triggers for deactivated agents on each department
DROP TRIGGER IF EXISTS chats_agents_status --change name
ON viz1.wk_chats; --change table name
CREATE OR REPLACE TRIGGER chats_agents_status --change name
    BEFORE INSERT OR UPDATE 
    ON viz1.wk_chats --change table
    FOR EACH ROW
    EXECUTE FUNCTION viz1.fc_check_status_agents();

--------------------------
-- Views Tables ⬇️⬇️
--------------------------
-- Daily Summary
DROP VIEW IF EXISTS viz1.daily_summary CASCADE;
CREATE OR REPLACE VIEW viz1.daily_summary AS
SELECT 
    'chats' AS department,
    chats.date_idx,
    chats.catg_id,
    count(*) FILTER (WHERE chats.rating = 0) AS yes_rate,
    count(*) FILTER (WHERE chats.rating = 1) AS no_rate,
    count(chats.rating) AS rates
FROM 
    viz1.wk_chats AS chats
GROUP BY 
    chats.date_idx, chats.catg_id
UNION ALL
SELECT 
    'calls' AS department,
    calls.date_idx,
    calls.catg_id,
    count(*) FILTER (WHERE calls.rating = 0) AS yes_rate,
    count(*) FILTER (WHERE calls.rating = 1) AS no_rate,
    count(calls.rating) AS rates
FROM 
    viz1.wk_phone AS calls
GROUP BY 
    calls.date_idx, calls.catg_id
UNION ALL
SELECT 
    'emails' AS department,
    emails.date_idx,
    emails.catg_id,
    count(*) FILTER (WHERE emails.rating = 0) AS yes_rate,
    count(*) FILTER (WHERE emails.rating = 1) AS no_rate,
    count(emails.rating) AS rates
FROM 
    viz1.wk_email AS emails
GROUP BY 
    emails.date_idx, emails.catg_id;
----------------------------------------
-- Week calendar helper
DROP VIEW IF EXISTS viz1.calendar_week_help CASCADE;
CREATE OR REPLACE VIEW viz1.calendar_week_help
AS
SELECT 
    idx,
    week_start,
    max(idx) OVER (PARTITION BY week_start) AS max_idx_week
FROM 
    viz1.calendar
WHERE 
    day_week < 6
ORDER BY 
idx;
----------------------------------------
-- Week Attendance
DROP VIEW IF EXISTS viz1.week_attendance CASCADE;
CREATE OR REPLACE VIEW viz1.week_attendance
AS(
SELECT cal.max_idx_week AS date_week,
att.emp_code AS employee_code,
att.attendance_status AS attendance_type,
count(*) AS total_attendance
FROM viz1.attendances AS att
    JOIN viz1.calendar_week_help AS cal ON att.date_idx = cal.idx
    JOIN viz1.agents AS ag ON ag.emp_code = att.emp_code
GROUP BY att.emp_code, att.attendance_status, cal.max_idx_week
ORDER BY cal.max_idx_week, att.emp_code);
----------------------------------------
-- Week Calls
DROP VIEW IF EXISTS viz1.week_calls CASCADE;
CREATE OR REPLACE VIEW viz1.week_calls
AS
SELECT cal.max_idx_week AS date_week,
phone.emp_code AS employee_code,
phone.catg_id AS category_type,
date_trunc('second'::text, avg(phone.aht::interval)) AS handling_time,
count(
    CASE
        WHEN phone.rating = 0 THEN 1
        ELSE NULL::integer
    END) AS csat,
count(
    CASE
        WHEN phone.rating = 1 THEN 1
        ELSE NULL::integer
    END) AS dsat
FROM viz1.wk_phone AS phone
    JOIN viz1.calendar_week_help AS cal ON phone.date_idx = cal.idx
GROUP BY cal.max_idx_week, phone.emp_code, phone.catg_id
ORDER BY cal.max_idx_week, phone.emp_code, phone.catg_id;
----------------------------------------
-- Week Chats
DROP VIEW IF EXISTS viz1.week_chats CASCADE;
CREATE OR REPLACE VIEW viz1.week_chats
AS
SELECT cal.max_idx_week AS date_week,
chats.emp_code AS employee_code,
chats.catg_id AS category_type,
date_trunc('second'::text, avg(chats.aht::interval)) AS handling_time,
count(
    CASE
        WHEN chats.rating = 0 THEN 1
        ELSE NULL::integer
    END) AS csat,
count(
    CASE
        WHEN chats.rating = 1 THEN 1
        ELSE NULL::integer
    END) AS dsat
FROM viz1.wk_chats AS chats
    JOIN viz1.calendar_week_help AS cal ON chats.date_idx = cal.idx
GROUP BY cal.max_idx_week, chats.emp_code, chats.catg_id
ORDER BY cal.max_idx_week, chats.emp_code, chats.catg_id;
----------------------------------------
-- Week Emails
DROP VIEW IF EXISTS viz1.week_emails CASCADE;
CREATE OR REPLACE VIEW viz1.week_emails
AS
SELECT cal.max_idx_week AS date_week,
email.emp_code AS employee_code,
email.catg_id AS category_type,
count(
    CASE
        WHEN email.rating = 0 THEN 1
        ELSE NULL::integer
    END) AS csat,
count(
    CASE
        WHEN email.rating = 1 THEN 1
        ELSE NULL::integer
    END) AS dsat
FROM viz1.wk_email email
    JOIN viz1.calendar_week_help AS cal ON email.date_idx = cal.idx
GROUP BY cal.max_idx_week, email.emp_code, email.catg_id
ORDER BY cal.max_idx_week, email.emp_code, email.catg_id;
----------------------------------------
-- Week QA Score
DROP VIEW IF EXISTS viz1.week_qa_score CASCADE;
CREATE OR REPLACE VIEW viz1.week_qa_score
AS
SELECT date_idx AS date_week,
emp_code,
round(sum(score1 + score2) / 2, 0) AS total_score
FROM viz1.qa_score qa
GROUP BY date_idx, emp_code
ORDER BY date_idx, emp_code;
----------------------------------------
-- Work Calendar
DROP VIEW IF EXISTS viz1.work_calendar CASCADE;
CREATE OR REPLACE VIEW viz1.work_calendar
AS
WITH first_last_id AS(
SELECT 
min(daily_summary.date_idx) AS min_idx,
max(daily_summary.date_idx) AS max_idx
FROM viz1.daily_summary)
SELECT idx,
dates,
months,
months_name,
days,
days_week,
days_name,
week_start
FROM viz1.calendar AS cal
INNER JOIN first_last_id AS help
ON cal.idx >= help.min_idx AND cal.idx <= help.max_idx
-- --------------------------------------------
--Bulk Insert
COPY --tablename
FROM --file/folder path
WITH (FORMAT csv, HEADER, DELIMITER ',');
------------------------------------------
