# The theft of the Mona Lisa
---

The exercises are adapted to Postgresql from the sql-tutorial by [opentechschool](http://opentechschool.github.io/sql-tutorial/)

How you will find the thief of the Mona Lisa?

Follow the tutorial, step-by-step:

-> Find out, who was in Paris at that time?

-> The thief was probably not working alone. Is there any suspicious communication during the time in question?

-> If you find the people responsible, who was the actual wire-puller and who was "only" the henchman?

In DBeaver set the monalisa as the default schema.

## Exercises:

Work in pairs to solve the following tasks

![](https://static.wikia.nocookie.net/s__/images/7/79/Jessica_fletcher.jpeg/revision/latest?cb=20130409163539&path-prefix=sherlockpedia%2Fde)

### 1. Investigate the tables

Take a look at the table names and columns. What table could have information about travelling?
the table flight

### 2. Have a look at the data

What kind of information is stored in the table with travelling information?

the passenger id, name, origin, destination and the date of travel

### 3. Who all live in Paris?

SELECT name
FROM person
WHERE residence = 'Paris'

### 4. Get the names 

Who travelled to Paris before 23.10.2014.

SELECT name FROM flight 
WHERE dest_city = 'Paris' 
AND date < '2014-10-23';

### 5. Get more names 

SELECT name FROM flight 
WHERE start_city = 'Paris' 
AND date > '2014-10-23';

### 6. Get even more the names 

Who travelled to Paris before 23.10.2014, and whose names also appear in entries for journeys departing from Paris after 23.10.2014. 

SELECT name FROM flight 
WHERE dest_city = 'Paris' 
AND date < '2014-10-23'
INTERSECT 
SELECT name FROM flight 
WHERE start_city = 'Paris' 
AND date > '2014-10-23';


### 7. Get all the names 

SELECT name FROM flight 
WHERE dest_city = 'Paris' 
AND date < '2014-10-23'
INTERSECT 
SELECT name FROM flight 
WHERE start_city = 'Paris' 
AND date > '2014-10-23'
UNION
SELECT name
FROM person
WHERE residence = 'Paris';

### What is the pool of suspects you have left?

The Local police will visit individuals on the list to inquire about their alibi. Proceed with the previous query, this time we require a list of names and their respective residences. 

First perform a selection on the flight table where the name is one of those who do not live in Paris. Second, check for every person-residence pair if there is a corresponding flight. For one there is no corresponding flight. Who is this person? Why is this person turning up on that list?

Now, check the result. Is there something wrong! Do you have the correct list of people who travelled to Paris before 23.10.2014 and who travelled from Paris after 23.10.2014?

select * from flight where name in ('Philipp', 'Kesia', 'Sarah');

/*
AND*

SELECT name FROM flight 
WHERE dest_city = 'Paris' 
AND date < '2014-10-23'
INTERSECT 
SELECT name FROM flight 
WHERE start_city = 'Paris' 
AND date > '2014-10-23'
UNION
SELECT name
FROM person
WHERE residence = 'Paris';

---

select distinct person.name, residence from person, flight 
where residence = 'Paris' or 
(flight.person_id = person.id and dest_city = 'Paris' 
and date < '2014-10-23' and flight.name in 
(select flight.name from flight where start_city = 'Paris'
and date > '2014-10-23'));


## Primary Key and Foreign Key

Now, look at your where statement in the last query you performed. Are the tables person and flight joined by the field name whose entries are not unique? <br>

To prevent mistakes like this we use primary key (PK) and foreign key (FK). People who create a table, usually define a column (or the combination of several columns) which is unique for every entry. This column is called the primary key. Other tables which are referencing entries of this table have a corresponding column, containing the same value as the primary key column of the referenced table. This corresponding column is called a foreign key. 
For example, in our person table, the field id is the primary key. This id is used in the phone_contract table as a foreign key person_id.

person (id) = phone_contract (person_id)

### 8. Foreign Keys

What foreign keys are used in the table containing the messages and which is the referenced table?

**hint:** the primary key is unique and the name often contains the suffix "id" and the foreign key columns often contain the name of the referenced table. (this is though only a convention, not always implemented in practice) 

SELECT * 
FROM messages 
WHERE sent > '2014-10-20' OR sent < '2014-10-25';

### 9. Try the previous query 

use the primary key/foreign key connection instead of the name

select id from phone_contract where phone_contract.person_id
in (select distinct person.id from person, flight
where residence = 'Paris' or 
(flight.person_id = person.id
and dest_city = 'Paris' and date < '2014-10-23'
and flight.name in (select flight.name from flight
where start_city = 'Paris' and date > '2014-10-23')));

## WHO is the thief?? - order by and group by
---

![](https://64.media.tumblr.com/bd06d5721a022f2ba59abf60a4537758/tumblr_o8ofi9NGiJ1udb1f6o1_500.jpg)

To find out who the thief is, check the text messages stored by the mobile phone providers.

### 10. What are the names 

Of the table containing the text messages and the ones containing phone contracts?

### 11. Get all the text messages 

Sent between 2014-10-20 and 2014-10-25.

SELECT * 
FROM messages 
WHERE sent > '2014-10-20'and sent < '2014-10-25';

### 12. Get all the contract ids 

Where the contract.person_id is equal to one of the persons from the results of question 7.

select * from messages where sent > '2014-10-20'
and sent < '2014-10-25' and contract_sender_id in
(select id from phone_contract
where phone_contract.person_id in
(select distinct person.id from person, flight
where residence = 'Paris' or (flight.person_id = person.id
and dest_city = 'Paris' and date < '2014-10-23'
and flight.name in (select flight.name from flight
where start_city = 'Paris' and date > '2014-10-23'))));

### 13. Get all text messages 

with the sent date between 2014-10-20 and 2014-10-25 and the contract_sender_id is equal to the contract ids. And where the contract.person_id is equal to one of the persons from the results of question 7.

- You see that you got all the required information but the output looks kind of chaotic. You can order a result set according to a column with an order by phrase. The query to get all text messages from 21.10.2014 ordered by time reads

- select a message from messages where sent like '2014-10-21'
order by sent;


SELECT * FROM messages
WHERE sent > '2014-10-20' and sent < '2014-10-25'
AND contract_sender_id IN (SELECT id FROM phone_contract
WHERE phone_contract.person_id IN
(SELECT DISTINCT person.id FROM person, flight
WHERE residence = 'Paris' OR (flight.person_id = person.id
AND dest_city = 'Paris' AND date < '2014-10-23'
AND flight.name IN (select flight.name FROM flight
WHERE start_city = 'Paris' AND date > '2014-10-23'))))
ORDER BY sent;

### 14. Get a list of messages 

From our suspects from the given time period and sort the messages.

### 15.  Who are the thieves? 

SELECT * 
FROM person
WHERE id = 100 OR id = 106;

Hint: Read the conversations as they give a clear trace. Check the ids of the sender and receiver of the message and look it up in the person tables!

If you have found the thieves you deserve some break! To practice more SQL, you can continue the investigation of this incredible crime and find out [who was the string puller](http://opentechschool.github.io/sql-tutorial/chapter4.html) when you have some free time.
