1. Равзвернем демо базу https://edu.postgrespro.ru/demo-medium.zip. `psql --username demo --host localhost --dbname demodb -f demo-medium-20170815.sql`
2. Прямое соединение - номер билета и дата бронирования
```
SELECT TICKETS.TICKET_NO,
	BOOKINGS.BOOK_DATE
FROM TICKETS
INNER JOIN BOOKINGS ON BOOKINGS.BOOK_REF = TICKETS.BOOK_REF
LIMIT 100;
```
3. Левостороннее соединение - билеты, для которых нет брони
```
SELECT TICKETS.TICKET_NO,
	BOOKINGS.BOOK_DATE
FROM TICKETS
LEFT JOIN BOOKINGS ON BOOKINGS.BOOK_REF = TICKETS.BOOK_REF
WHERE BOOKINGS.BOOK_REF IS NULL
LIMIT 100;
```
4. Кросс соединение - аэропорты и модели самолетов, которые могут летать через эти аэропорты
```
SELECT AIRPORT_CODE,
	AIRCRAFT_CODE,
	MODEL ->> 'ru' as model
FROM BOOKINGS.AIRPORTS_DATA
CROSS JOIN BOOKINGS.AIRCRAFTS_DATA;
```
5. Полное соединение. Получаем все места во всех моделях самолетов
```
SELECT SEAT_NO,
	MODEL - >> 'ru' AS MODEL
FROM SEATS
FULL JOIN AIRCRAFTS_DATA ON AIRCRAFTS_DATA.AIRCRAFT_CODE = SEATS.AIRCRAFT_CODE
```
6. Разный тип соединений. Получаем количество бронирований по аэропортам
```
SELECT A.AIRPORT_NAME ->> 'ru' AS NAME,
	COUNT(*) AS BOOK_COUNT
FROM BOOKINGS.TICKETS T
LEFT JOIN BOOKINGS.BOOKINGS B ON T.BOOK_REF = B.BOOK_REF
INNER JOIN BOOKINGS.BOARDING_PASSES P ON P.TICKET_NO = T.TICKET_NO
INNER JOIN BOOKINGS.FLIGHTS F ON F.FLIGHT_ID = P.FLIGHT_ID
INNER JOIN BOOKINGS.AIRPORTS_DATA A ON A.AIRPORT_CODE = F.ARRIVAL_AIRPORT
GROUP BY A.AIRPORT_NAME
ORDER BY NAME ASC
```

Схема бд:
<img width="886" alt="image" src="https://user-images.githubusercontent.com/9808191/171599316-2d8414f7-b752-4c6e-a3e0-f4102bf3cd11.png">
