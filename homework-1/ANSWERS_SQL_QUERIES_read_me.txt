Question 1. Understanding docker first run

Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

What's the version of pip in the image?

    24.3.1
    24.2.1
    23.3.1
    23.2.1

Terminal query:

> docker run -it --entrypoint bash python:3.12.8
> pip list

Answer: 24.3.1

Question 2. Understanding Docker networking and docker-compose

Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?

services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data

    postgres:5433
    localhost:5432
    db:5433
    postgres:5432
    db:5432

If there are more than one answers, select only one of them.

Answer: db:5432


-----------------------
Run Postgres and load data as shown in the videos We'll use the green taxi trips from September 2019:

wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz

You will also need the dataset with zones:

wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline)

----------------------
For this step, I manually downloaded the second link since the link is dead. Putting them on the same folder as my docker-compose.yaml file, I then used

>docker-compose up

and logged in in localhost:8080 and created my local server as instructed. The .yaml file is found where this .txt is located on.
----------------------


Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:

    Up to 1 mile
    In between 1 (exclusive) and 3 miles (inclusive),
    In between 3 (exclusive) and 7 miles (inclusive),
    In between 7 (exclusive) and 10 miles (inclusive),
    Over 10 miles

Answers:

    104,802; 197,670; 110,612; 27,831; 35,281
    104,802; 198,924; 109,603; 27,678; 35,189
    104,793; 201,407; 110,612; 27,831; 35,281
    104,793; 202,661; 109,603; 27,678; 35,189
    104,838; 199,013; 109,645; 27,688; 35,202

SELECT 
  COUNT(CASE WHEN trip_distance <= 1 THEN 1 END) AS "x <= 1",
  COUNT(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 END) AS "1 < x <= 3",
  COUNT(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 END) AS "3 < x <= 7",
  COUNT(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 END) AS "7 < x <= 10",
  COUNT(CASE WHEN trip_distance > 10 THEN 1 END) AS "x > 10"
FROM green_taxi_data
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00' 
  AND lpep_pickup_datetime < '2019-11-01 00:00:00'
  
Answer: 104,849; 199,040; 109,670; 27,693; 35,202




Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance.

    2019-10-11
    2019-10-24
    2019-10-26
    2019-10-31
    
Answer:

SELECT
  CAST(lpep_pickup_datetime as DATE) as "day",
  trip_distance
FROM 
  green_taxi_data t
ORDER BY trip_distance DESC;

Answer is 2019-10-31.


Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in total_amount (across all trips) for 2019-10-18?

Consider only lpep_pickup_datetime when filtering by date.

    East Harlem North, East Harlem South, Morningside Heights
    East Harlem North, Morningside Heights
    Morningside Heights, Astoria Park, East Harlem South
    Bedford, East Harlem North, Astoria Park

Answer:

SELECT
  zpu."Zone" AS pick_up_zone,
  SUM(total_amount)
FROM 
  green_taxi_data t
  FULL OUTER JOIN zones zpu
    ON t."PULocationID" = zpu."LocationID"
WHERE 
  CAST(lpep_pickup_datetime AS DATE) = '2019-10-18'
GROUP BY 
  zpu."Zone"
HAVING 
  SUM(total_amount) > 13000
ORDER BY 
  SUM(total_amount) DESC
  
Answer is East Harlem North, East Harlem South, Morningside Heights.


Question 6. Largest tip

For the passengers picked up in October 2019 in the zone name "East Harlem North" which was the drop off zone that had the largest tip?

Note: it's tip , not trip

We need the name of the zone, not the ID.

    Yorkville West
    JFK Airport
    East Harlem North
    East Harlem South

Answer:

SELECT
  zpu."Zone" AS astoria_pickup_zone,
  zdo."Zone" AS drop_off_zone,
  t."tip_amount" AS tip_amount
FROM 
  green_taxi_data t
  FULL OUTER JOIN zones zpu
    ON t."PULocationID" = zpu."LocationID"
  FULL OUTER JOIN zones zdo
  	ON t."DOLocationID" = zdo."LocationID"
WHERE
  EXTRACT(YEAR FROM t.lpep_pickup_datetime) = 2019
  AND EXTRACT(MONTH FROM t.lpep_pickup_datetime) = 10
  AND zpu."Zone" = 'East Harlem North'
ORDER BY 
  tip_amount DESC
  
Answer is JFK Airport.

Question 7. Terraform Workflow

Which of the following sequences, respectively, describes the workflow for:

    Downloading the provider plugins and setting up backend,
    Generating proposed changes and auto-executing the plan
    Remove all resources managed by terraform`

Answers:

    terraform import, terraform apply -y, terraform destroy
    teraform init, terraform plan -auto-apply, terraform rm
    terraform init, terraform run -auto-approve, terraform destroy
    terraform init, terraform apply -auto-approve, terraform destroy
    terraform import, terraform apply -y, terraform rm



Answer is terraform init, terraform apply -auto-approve, terraform destroy.
