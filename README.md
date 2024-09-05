# This is a complex query with more than 20 SQL elements in it. 

--This section of the query selects the top five records from the property table
-- inner joined with the owner table. This section also excludes owner BI109
-- and any record that does NOT have a zip code that begins with 9 or 1.
SELECT TOP 5 PROPERTY_ID, P.OWNER_NUM, P.ADDRESS, ZIP_CODE, (MONTHLY_RENT * 12) AS YEARLY_RENT
  FROM PROPERTY P INNER JOIN OWNER O ON P.OWNER_NUM = O.OWNER_NUM
      WHERE O.OWNER_NUM != 'BI109'
        AND O.ADDRESS IN(SELECT ADDRESS
          FROM OWNER
            WHERE ZIP_CODE LIKE '9%'
              OR ZIP_CODE LIKE '1%')
-- This section of the query selects the top five records from the Property table
-- which is left joined with the service request table and left joined with the 
-- owner table. The query also only selects records that have a yearly rent between
-- 10k and 25k, and records that have a service category of painting.
UNION
SELECT 5 PROPERTY_ID, P.OWNER_NUM, P.ADDRESS, ZIP_CODE, (MONTHLY_RENT * 12) AS YEARLY_RENT
  FROM PROPERTY P LEFT JOIN SERVICE_REQUEST SR ON P.PROPERTY_ID = SR.PROPERTY_ID
    LEFT JOIN OWNER O ON P.OWNER_NUM = O.OWNER_NUM
      WHERE (MONTHLY_RENT * 12) BETWEEN 10000 AND 25000
AND EXISTS (SELECT *
    FROM SERVICE_CATEGORY SG 
      WHERE CATEGORY_DESCRIPTION = 'PAINTING'
        AND SR.CATEGORY_NUMBER = SG.CATEGORY_NUM)
--This section of the query intersects with the previous sections. The query
--Uses the property table which is fully joined with the service request table
--which full joined with the owner table. This section selects the top five records
--where monthly rent is greater than the average monthly rent and where the
--is not a null entry for the next service date. Finally this section orders
--the selection by property id in desc order. 
INTERSECT
SELECT 5 PROPERTY_ID, P.OWNER_NUM, P.ADDRESS, ZIP_CODE, (MONTHLY_RENT * 12) AS YEARLY_RENT
  FROM PROPERTY P FULL JOIN SERVICE_REQUEST SR ON P.PROPERTY_ID = SR.PROPERTY_ID
  FULL JOIN OWNER O ON P.OWNER_NUM = O.OWNER_NUM
    WHERE MONTHLY_RENT > (SELECT AVG(MONTHLY_RENT)
      FROM PROPERTY)
        AND NEXT_SERVICE_DATE IS NOT NULL
--This section creates a table with a cross join to get all possible outcomes
--of the table
UNION
SELECT 5 PROPERTY_ID, P.OWNER_NUM, P.ADDRESS, ZIP_CODE, (MONTHLY_RENT * 12) AS YEARLY_RENT
  FROM PROPERTY P CROSS JOIN SERVICE_REQUEST LEFT JOIN OWNER O ON P.OWNER_NUM = O.OWNER_NUM
--This section selects entries where established hours are greater than the max
--established hours and groups them by having a category number greater than 3
--or where owners live in the same state. 
UNION
SELECT 5 PROPERTY_ID, P.OWNER_NUM, P.ADDRESS, ZIP_CODE, (MONTHLY_RENT * 12) AS YEARLY_RENT
  FROM PROPERTY P LEFT JOIN SERVICE_REQUEST SR ON P.PROPERTY_ID = SR.PROPERTY_ID
    LEFT JOIN OWNER O ON P.OWNER_NUM = O.OWNER_NUM
      WHERE EST_HOURS > ANY (SELECT MAX(EST_HOURS)
                  FROM SERVICE_REQUEST
                    GROUP BY CATEGORY_NUMBER
                      HAVING COUNT(CATEGORY_NUMBER) > 3)
      OR P.OWNER_NUM IN (SELECT A.OWNER_NUM 
      FROM OWNER A INNER JOIN OWNER B ON A.STATE = B.STATE
        WHERE A.OWNER_NUM > B.OWNER_NUM)
      ORDER BY P.PROPERTY_ID DESC;

