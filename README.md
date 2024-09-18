# SQL-Project
Analyzed data by solving complex SQL queries and optimizing query performance for efficient data retrieval.

I write queries to retrieve, update, and manipulate data from databases to support decision-making processes.

### SQL Concepts used: Aggregation functions, Joins, Window functions, Conditional functions, etc.

1.	Represent the “book_date” column in “yyyy-mmm-dd” format using Bookings table 
Expected output: book_ref, book_date (in “yyyy-mmm-dd” format) , total amount 
Answer:  
select 
 book_ref,
 to_char(book_date, 'yyyy-mmm-dd') as book_date, 
 total_amount 
from bookings
 
2.	Get the following columns in the exact same sequence.
Expected columns in the output: ticket_no, boarding_no, seat_number, passenger_id, passenger_name.

Answer: 
select 
 bp.ticket_no,
 bp.boarding_no,
 bp.seat_no as seat_number,
 t.passenger_id,
 t.passenger_name
from tickets as t 
join boarding_passes as bp 
on bp.ticket_no = t.ticket_no


3.	Write a query to find the seat number which is least allocated among all the seats?

Answer: 
with cte as (select 
 seat_no,
 count(ticket_no) as count 
 from boarding_passes
 group by 1),

least_allocation as (
    select *,
rank() over (order by count asc) as least_allocated_seats
from cte)

select 
seat_no
from least_allocation
where least_allocated_seats = 1


4.	In the database, identify the month wise highest paying passenger name and passenger id.
Expected output: Month_name(“mmm-yy” format), passenger_id, passenger_name and total amount

Answer: 
with cte as (select 
to_char(b.book_date, 'mmm-yy') as Month_name,
t.passenger_id,
t.passenger_name,
b.total_amount 
from bookings as b 
left join tickets as t 
on b.book_ref = t.book_ref),
Highest_paying_passenger as (
    select *, 
    rank() over(partition by Month_name order by total_amount desc) as rnk 
    from cte 
)
select 
Month_name,
passenger_id,
passenger_name,
total_amount
from Highest_paying_passenger
where rnk = 1



5.	In the database, identify the month wise least paying passenger name and passenger id?
Expected output: Month_name(“mmm-yy” format), passenger_id, passenger_name and total amount

Answer: 
with cte as (select 
to_char(b.book_date, 'mmm-yy') as Month_name,
t.passenger_id,
t.passenger_name,
b.total_amount 
from bookings as b 
left join tickets as t 
on b.book_ref = t.book_ref),
Highest_paying_passenger as (
    select *, 
    rank() over(partition by Month_name order by total_amount asc) as rnk 
    from cte 
)
select 
Month_name,
passenger_id,
passenger_name,
total_amount
from Highest_paying_passenger
where rnk = 1


6.	Identify the travel details of non stop journeys  or return journeys (having more than 1 flight).
Expected Output: Passenger_id, passenger_name, ticket_number and flight count.

Answer: 
select
    t.passenger_id,
    t.passenger_name,
    t.ticket_no,
    count(tf.flight_id) as flight_count
from tickets as t 
join ticket_flights as tf 
on t.ticket_no = tf.ticket_no
group by 1,2,3
having count(tf.flight_id) > 1


7.	How many tickets are there without boarding passes?
Expected Output: just one number is required.

Answer: 
select 
count(t.ticket_no) as tickets_without_boarding_passes
from tickets t 
left join boarding_passes bp 
on t.ticket_no = bp.ticket_no
where boarding_no is null



8.	Identify details of the longest flight (using flights table)?
Expected Output: Flight number, departure airport, arrival airport, aircraft code and durations.

Answer: 
select 
flight_no,
departure_airport,
arrival_airport,
aircraft_code,
(actual_arrival - actual_departure) duration
from flights
where actual_arrival is not null 
and actual_departure is not null 
order by 5 desc
limit 1 



9.	Identify details of all the morning flights (morning means between 6AM to 11 AM, using flights table)?
Expected output: flight_id, flight_number, scheduled_departure, scheduled_arrival and timings.

Answer: 
select 
flight_id,
flight_no,
scheduled_departure,
scheduled_arrival,
to_char(scheduled_departure, 'HH24:MI:SS') timings
from flights
where 
    extract(hour from scheduled_departure) >= 6
    and extract(hour from scheduled_departure) < 11


10.	Identify the earliest morning flight available from every airport.
Expected output: flight_id, flight_number, scheduled_departure, scheduled_arrival, departure airport and timings.
Answer: 
with cte as(
    select 
    flight_id,
    flight_no,
    scheduled_departure,
    scheduled_arrival,
    departure_airport,
    to_char(scheduled_departure, 'HH24:MI:SS') timings,
    rank() over(partition by departure_airport order by scheduled_departure asc ) rnk 
    from flights)
select 
    flight_id,
    flight_no,
    scheduled_departure,
    scheduled_arrival,
    departure_airport,
    to_char(scheduled_departure, 'HH24:MI:SS') timings
from cte
where 
    extract(hour from scheduled_departure) >= 6
    and extract(hour from scheduled_departure) < 11 
    and rnk = 1


11.	Questions: Find list of airport codes in Europe/Moscow timezone
 Expected Output:  Airport_code. 

Answer: 
select 
airport_code
from airports
where timezone = ('Europe/Moscow')



12.	Write a query to get the count of seats in various fare condition for every aircraft code?
 Expected Outputs: Aircraft_code, fare_conditions ,seat count

Answer: 
select 
aircraft_code,
fare_conditions,
count(seat_no) seat_count
from seats
group by 1,2



13.	How many aircrafts codes have at least one Business class seats?
 Expected Output : Count of aircraft codes

Answer: 
select
count(distinct aircraft_code) 
from seats 
where fare_conditions = 'Business'
having count(seat_no) >= 1



14.	Find out the name of the airport having maximum number of departure flight
 Expected Output: Airport_name 


Answer:  
 select 
 airport_name
 from flights as f 
 join airports as a 
 on f.departure_airport = a.airport_code
 group by 1
 order by count(flight_no) desc
 limit 1 



15.	Find out the name of the airport having least number of scheduled departure flights
 Expected Output : Airport_name 


Answer: 
  select 
 airport_name
 from flights as f 
 join airports as a 
 on f.departure_airport = a.airport_code
 group by 1
 order by count(scheduled_departure) asc
 limit 1 



16.	How many flights from ‘DME’ airport don’t have actual departure?
 Expected Output : Flight Count 

Answer: 
select 
count(flight_no) as Flight_count
from flights
where departure_airport = 'DME' and actual_departure is null


17.	Identify flight ids having range between 3000 to 6000
 Expected Output : Flight_Number , aircraft_code, ranges 

Answer: 
select 
f.flight_no as Flight_Number,
a.aircraft_code,
a.range
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where range between 3000 and 6000



18.	Write a query to get the count of flights flying between URS and KUF?
 Expected Output : Flight_count

Answer: 
select 
count(flight_no) as Flight_count
from flights
where departure_airport = 'URS' and arrival_airport = 'KUF'



19.	Write a query to get the count of flights flying from either from NOZ or KRR?
 Expected Output : Flight count 

Answer: 
select 
count(Flight_no) as Flight_count
from flights
where departure_airport in ('NOZ','KRR') 




20.	Write a query to get the count of flights flying from KZN,DME,NBC,NJC,GDX,SGC,VKO,ROV
Expected Output : Departure airport ,count of flights flying from these   airports.

Answer: 
select 
departure_airport,
count(Flight_no) as Flight_count
from flights
where departure_airport in ('KZN','DME','NBC','NJC','GDX','SGC','VKO','ROV')
group by 1



21.	Write a query to extract flight details having range between 3000 and 6000 and flying from DME
Expected Output :Flight_no,aircraft_code,range,departure_airport

Answer: 
select 
f.flight_no,
a.aircraft_code,
a.range,
f.departure_airport
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where (range between 3000 and 6000) and (departure_airport = 'DME')



22.	Find the list of flight ids which are using aircrafts from “Airbus” company and got cancelled or delayed
 Expected Output : Flight_id,aircraft_model
Answer: 
select 
f.flight_id,
a.model as aircraft_model
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where (model like ('%Airbus%'))
  and (status in ('Cancelled','Delayed'))


23.	Find the list of flight ids which are using aircrafts from “Boeing” company and got cancelled or delayed
Expected Output : Flight_id,aircraft_model

Answer: 
select 
f.flight_id,
a.model as aircraft_model
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where (model like ('%Boeing%'))
  and (status in ('Cancelled','Delayed'))


24.	Which airport(name) has most cancelled flights (arriving)?
Expected Output : Airport_name 

Answer: 
 select 
 airport_name
 from flights as f 
 join airports as a 
 on f.departure_airport = a.airport_code
 where status in ('Cancelled')
 group by 1
 order by count(flight_no) desc
 limit 1 

25.	Identify flight ids which are using “Airbus aircrafts”
Expected Output : Flight_id,aircraft_model

Answer: 
select 
f.flight_id,
a.model as aircraft_model
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where model like ('%Airbus%')


26.	Identify date-wise last flight id flying from every airport?
Expected Output: Flight_id,flight_number,schedule_departure,departure_airport

Answer: 
with cte as (
select 
flight_id,
flight_no,
scheduled_departure,
departure_airport,
date (scheduled_departure) as date
from flights ), 

Final as (
    select 
    *,
    row_number() over(partition by departure_airport order by date desc) as rnk 
    from cte
)

select 
flight_id,
flight_no,
scheduled_departure,
departure_airport
from final 
where rnk = 1 


27.	Identify list of customers who will get the refund due to cancellation of the flights and how much amount they will get?
Expected Output : Passenger_name,total_refund.


Answer: 
select
    t.passenger_name,
    sum(b.total_amount) as total_refund
from tickets as t 
join bookings as b 
on t.book_ref = b.book_ref
join flights as f 
on f.flight_id = f.flight_id
where f.status = 'Cancelled'
group by t.passenger_name



28.	Identify date wise first cancelled flight id flying for every airport?
Expected Output : Flight_id,flight_number,schedule_departure,departure_airport


Answer: 
with cte as (
select 
flight_id,
flight_no,
scheduled_departure,
departure_airport,
status,
date (scheduled_departure) as date
from flights
where status = 'Cancelled'), 

Final as (
    select 
    *,
    row_number() over(partition by departure_airport order by date asc) as rnk 
    from cte
)

select 
flight_id,
flight_no,
scheduled_departure,
departure_airport
from final 
where rnk = 1 and status = 'Cancelled'






29.	Identify list of Airbus flight ids which got cancelled.
Expected Output : Flight_id

Answer: 
select 
f.flight_id
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where f.status = 'Cancelled' and a.model like ('%Airbus%')

30.	Identify list of flight ids having highest range.
 Expected Output : Flight_no, range

Answer: 
with max_range as (
select 
max(range) as maxrange 
from aircrafts )

select 
f.flight_no,
a.range
from flights as f 
join aircrafts as a 
on f.aircraft_code = a.aircraft_code
where range = (select maxrange from max_range)
group by 1,2
