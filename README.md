# Drive Lah Assessment

# Problem Statement
Project -

 As a data analyst you are given a task to add two charts on the weekly KPI tracker dashboard.
You will need to connect to the Database and write SQL query for both these charts and provide them to us.
Mentioned below are the credentials for the development database. It has the same structure as that for production, only the data is dummy.

TASK 1:
 You are to create a time series chart which compares successful trips days booked in last 3 months as compared to the same period the previous year grouped weekly.
So basically let's say if I am opening that chart today, it will show me data for the last three months (24 Oct '23 to 24 Jan'24) and compare it to 24 Oct '22 to 24 Jan'23).
 Each week in the weekly grouping has to start with Monday.
 You will need to use a transaction table for this.
 Some help for you to identify fields and nomenclatures:
 BookingStart and BookingEnd has the duration for you to get Trip days from.
 createdAt is the timestamp to be used for trip creation.
 Output needs to be a table which has the following columns
 Week starting date
 Trip days this year (cumulative for the week)
 Trip days last year (cumulative for the week)
 
TASK 2:
 We use Metabse for our SQL based analytics.
 One of the requirements your stakeholders have is to be able to change the parameters via filters.
 For example in the chart above apply filters based on platform.
 Do your research and provide a solution on how you will do this.
 The same query above will have to be modified with this.
 Fields on transaction level for platform
 In transaction table > protectedData
 If booking is created from iOS: isMobile = true
 If booking is created from Android: isAndroid = true
 If both of these fields are not present/null then its created from Web.

 # SQL Scripts
 # Task 1:
 SELECT WeekNumber,
 SUM(CASE WHEN Year = 2024 THEN TotalTripDays ELSE 0 END) AS TripDaysThisYear,
    SUM(CASE WHEN Year = 2023 THEN TotalTripDays ELSE 0 END) AS TripDaysLastYear
 FROM (
 SELECT DATE_FORMAT(createdAt, '%Y-%m-%d') AS WeekStartingDate,
       YEAR(createdAt) AS Year,
       WEEK(createdAt, 1) AS WeekNumber,
       SUM(DATEDIFF(bookingEnd,bookingStart)) AS TotalTripDays
 FROM transactions
 WHERE (createdAt >= DATE_SUB(CURDATE() , INTERVAL 3 MONTH) and 
 createdAt <= CURDATE()-INTERVAL 1 day) or
 (createdAt >= DATE_SUB(DATE_SUB(CURDATE() , INTERVAL 3 MONTH),INTERVAL 1 YEAR) and 
 createdAt <= DATE_SUB((CURDATE()-INTERVAL 1 day),INTERVAL 1 year))
 GROUP BY Year, WeekNumber
 order by YEAR, WeekNumber
 ) as subquery
 GROUP BY WeekNumber;

 ![1](https://github.com/Sivakumar1707/Drive-Lah-Assessment/assets/156114789/5ac74960-74b4-44a2-b87f-893fc14c77a9)

 ![4](https://github.com/Sivakumar1707/Drive-Lah-Assessment/assets/156114789/149bdf15-509f-4215-8e0d-0934a49a8e25)


# Task 2: Type 1:
SELECT WeekNumber, device,
SUM(CASE WHEN Year = 2024 THEN TotalTripDays ELSE 0 END) AS TripDaysThisYear,
SUM(CASE WHEN Year = 2023 THEN TotalTripDays ELSE 0 END) AS TripDaysLastYear
FROM (
SELECT DATE_FORMAT(createdAt, '%Y-%m-%d') AS WeekStartingDate,
       YEAR(createdAt) AS Year,
       WEEK(createdAt, 1) AS WeekNumber,
       SUM(DATEDIFF(bookingEnd,bookingStart)) AS TotalTripDays,
       CASE 
WHEN JSON_CONTAINS(protectedData, '{"fromMobile": true}') THEN 'ios'
when JSON_CONTAINS(protectedData, '{"fromAndroid": true}') THEN 'android' else 'web' end as device
FROM transactions
WHERE (createdAt >= DATE_SUB(CURDATE() , INTERVAL 3 MONTH) and 
createdAt <= CURDATE()-INTERVAL 1 day) or
(createdAt >= DATE_SUB(DATE_SUB(CURDATE() , INTERVAL 3 MONTH),INTERVAL 1 YEAR) and 
createdAt <= DATE_SUB((CURDATE()-INTERVAL 1 day),INTERVAL 1 year))
GROUP BY Year, WeekNumber, device
order by YEAR, WeekNumber
) as subquery
GROUP BY WeekNumber, device;

![2](https://github.com/Sivakumar1707/Drive-Lah-Assessment/assets/156114789/4494e90e-d252-4be7-957f-2c1f242ad0b1)

![3](https://github.com/Sivakumar1707/Drive-Lah-Assessment/assets/156114789/dda9ebcb-311a-41c7-affb-744ba8cc1791)


# Type 2:
SELECT 
    WeekNumber,
    SUM(CASE WHEN device = 'ios' and year = 2024 THEN TotalTripDays ELSE 0 END) AS TripDaysThisYear_ios_2024,
    SUM(CASE WHEN device = 'android' and year = 2024 THEN TotalTripDays ELSE 0 END) AS TripDaysThisYear_android_2024,
    SUM(CASE WHEN device = 'web' and year = 2024 THEN TotalTripDays ELSE 0 END) AS TripDaysThisYear_web_2024,
    SUM(CASE WHEN device = 'ios' and year = 2023 THEN TotalTripDays ELSE 0 END) AS TripDaysLastYear_ios_2023,
    SUM(CASE WHEN device = 'android' and year = 2023 THEN TotalTripDays ELSE 0 END) AS TripDaysLastYear_android_2023,
    SUM(CASE WHEN device = 'web' and year = 2023 THEN TotalTripDays ELSE 0 END) AS TripDaysLastYear_web_2023
FROM (
    SELECT 
    DATE_FORMAT(createdAt, '%Y-%m-%d') AS WeekStartingDate,
    YEAR(createdAt) AS Year,
    WEEK(createdAt, 1) AS WeekNumber,
    SUM(DATEDIFF(bookingEnd, bookingStart)) AS TotalTripDays,
    CASE 
    WHEN JSON_CONTAINS(protectedData, '{"fromMobile": true}') THEN 'ios'
    WHEN JSON_CONTAINS(protectedData, '{"fromAndroid": true}') THEN 'android'
    ELSE 'web' 
    END AS device
    FROM 
    transactions
    WHERE 
    (createdAt >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH) AND createdAt <= CURDATE() - INTERVAL 1 DAY)
    OR
    (createdAt >= DATE_SUB(DATE_SUB(CURDATE(), INTERVAL 3 MONTH), INTERVAL 1 YEAR) AND createdAt <= DATE_SUB((CURDATE() - INTERVAL 1 DAY), INTERVAL 1 YEAR))
    GROUP BY Year, WeekNumber, device
) AS subquery
GROUP BY WeekNumber
ORDER BY WeekNumber;

![5](https://github.com/Sivakumar1707/Drive-Lah-Assessment/assets/156114789/4358ef10-8f0d-4f6f-b5ba-770dc8be7e14)

![6](https://github.com/Sivakumar1707/Drive-Lah-Assessment/assets/156114789/b81294e3-5736-41f0-b01a-c7d71d2e3c50)

 


