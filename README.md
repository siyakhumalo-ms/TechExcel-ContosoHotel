# ![Contoso Hotel Icon](contoso_hotel/static/favicons/favicon-32x32.png) Contoso Hotel Demo in Python

This is a Demo Application for internal Booking Management of Contoso Hotel. It supports MSSQL and PostgreSQL databases and offers Rest APIs for integration into the larger IT landscape of Contoso Hotel.
https://microsoft.github.io/TechExcel-Modernize-applications-to-be-AI-ready/Docs/Ex02/0201.html
https://rise.articulate.com/share/vqdL_-ZoLawZN1DAZMkHIoBqhbp1mC6y#/
![Contoso Hotel Screenshot](contoso_hotel.jpg)

## Setting up a test database
In case you do not have a test database, you can use the following steps to create a test database in Azure.
 1. Install the test database:
    1. For MSSQL run:
       ```pwsh
       .\iac\manageIac.ps1 -iacAction create -passwd "myLittleSecret111!!!" -deploy "mssql"
       # for more options use: Get-Help .\iac\manageIac.ps1
       ```
    1. For PostgreSQL run:
       ```pwsh
       .\iac\manageIac.ps1 -iacAction create -passwd "myLittleSecret111!!!" -deploy "postgresql"
       # for more options use: Get-Help .\iac\manageIac.ps1
       ```
 1. You can use the connection string from the output (in green color) to connect to the test database as described below.

## General setup guidance

 1. Configure Environment Variable:
    1. For MSSQL: set ``MSSQL_CONNECTION_STRING`` environment variable or supply a file named ``./secrets-store/MSSQL_CONNECTION_STRING``
       * Uses pyodbc, format is: ``DRIVER={ODBC Driver 18 for SQL Server};SERVER=MSSQLINSTANCENAME.database.windows.net;DATABASE=MSSQLDBNAME;UID=MSSQLUSERNAME;PWD=*******;Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;``
    1. For PostgreSQL: set ``POSTGRES_CONNECTION_STRING`` environment variable or supply a file named ``./secrets-store/POSTGRES_CONNECTION_STRING``
       * Uses psycopg2, format is: ``user=PGUSERNAME;password=*******;host=PGINSTANCENAME.postgres.database.azure.com;port=5432;database=PGDBNAME;``
 1. Run the app:
    ```bash
    gunicorn --bind=0.0.0.0 --workers=4 startup:app
    ```
 1. Populate Data:
    1.  **Either** go to: http://localhost:8000/setup
    1.  **Or** invoke the Rest API:
        ```pwsh
        Invoke-RestMethod -Uri 'http://localhost:8000/api/setup' -Method Post -Body '{ "drop_schema" : true, "create_schema": true, "populate_data" : true }' -ContentType 'application/json'
        ```


## Docker based setup

 1. Build the Docker Image:
    ```bash
    docker build -t pycontosohotel:latest .
    ```
 1. Choose one of the following methods to provide the connection string to the container:
    1. For MSSQL (uses pyodbc):
       1. Set environment variable when running the container:
          ```bash
          docker run -p 8000:8000 -e MSSQL_CONNECTION_STRING='DRIVER={ODBC Driver 18 for SQL Server};SERVER=MSSQLINSTANCENAME.database.windows.net;DATABASE=MSSQLDBNAME;UID=MSSQLUSERNAME;PWD=*******;Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;' pycontosohotel:latest
          ```
       1. Use a volume mount
          1. Create a file ``MSSQL_CONNECTION_STRING`` with the connection string in the ``/path/to/secrets-store`` directory
          1. Mount the directory when running the container:
            ```bash
            docker run -p 8000:8000 -v '/path/to/secrets-store:/app/secrets-store' pycontosohotel:latest
            ```
    1. For PostgreSQL (uses psycopg2):
         1. Set environment variable when running the container:
            ```bash
            docker run -p 8000:8000 -e POSTGRES_CONNECTION_STRING='user=PGUSERNAME;password=*******;host=PGINSTANCENAME.postgres.database.azure.com;port=5432;database=PGDBNAME;' pycontosohotel:latest
            ```
         1. Use a volume mount
            1. Create a file ``POSTGRES_CONNECTION_STRING`` with the connection string in the ``/path/to/secrets-store`` directory
            1. Mount the directory when running the container:
               ```bash
               docker run -p 8000:8000 -v '/path/to/secrets-store:/app/secrets-store' pycontosohotel:latest
               ```
 1. Populate Data:
    1.  **Either** go to: http://localhost:8000/setup
    1.  **Or** invoke the Rest API:
        ```pwsh
        Invoke-RestMethod -Uri 'http://localhost:8000/api/setup' -Method Post -Body '{ "drop_schema" : true, "create_schema": true, "populate_data" : true }' -ContentType 'application/json'
        ```

# Supported Environment Variables
You can either set the environemnt variables or provide a file with the connection string in the ``./secrets-store`` directory. (e.g. ``./secrets-store/MSSQL_CONNECTION_STRING``)

In the docker container, the path is ``/app/secrets-store``.

All variables are optional, but at least one of the database connection strings must be provided.

| Variable Name |  Description | Example |
| --- | --- | --- |
| ``MSSQL_CONNECTION_STRING`` | Connection string for MSSQL (uses pyodbc) | ``DRIVER={ODBC Driver 18 for SQL Server};SERVER=MSSQLINSTANCENAME.database.windows.net;DATABASE=MSSQLDBNAME;UID=MSSQLUSERNAME;PWD=*******;Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;`` |
| ``POSTGRES_CONNECTION_STRING`` | Connection string for PostgreSQL (uses psycopg2) | ``user=PGUSERNAME;password=*******;host=PGINSTANCENAME.postgres.database.azure.com;port=5432;database=PGDBNAME;`` |
| ``API_BASEURL`` | Base URL for the API | ``http://localhost:8000`` |
| ``CHATBOT_BASEURL`` | Base URL for the Chatbot  (use ``/`` to activate chatbot demo interface) | ``http://localhost:8001`` |
| ``CHATBOT_KEY`` | The chatbot authorization key if any was set (usually for deployment through AI Studio) | ``1234567890`` |
| ``CHATBOT_FRONTEND_USE_CHATBOT_BASEURL`` | If set to ``true`` the chatbot JS frontend will directly send requests to the ``CHATBOT_BASEURL``. (**default is** ``false``, that sends everything to the backend and the backend will then send it to the CHATBOT_BASEURL) | ``true`` |


# API documentation

## Get Hotels

**Endpoint:** ``GET /api/hotels``

| Get Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| ``hotelname``  | string | *empty* | Optional Hotel Name to filter |
| ``exactMatch`` | bool | false | Optional exactMatch (``false`` uses ``like '%search%'`` ) |

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
[
  {
    "hotelId": 6,
    "hotelname": "Contoso Hotel Los Angeles",
    "pricePerNight": 350.0,
    "totalRooms": 100,
    "country": "United States"
  }
]
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/hotels'
```

#### Bash Curl
```bash
curl -X GET 'http://localhost:8000/api/hotels'
```
</details>


## Get Visitors

**Endpoint:** ``GET /api/visitors``

| Get Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| ``name``  | string | *empty* | Optional Name to filter (first or last name) |
| ``exactMatch`` | bool | false | Optional exactMatch (``false`` uses ``like '%search%'`` ) |

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
[
  {
    "firstname": "Frank",
    "lastname": "Green",
    "visitorId": 6
  }
]
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/visitors'
```

#### Bash Curl
```bash
curl -X GET 'http://localhost:8000/api/visitors'
```
</details>


## Get Bookings

**Endpoint:** ``GET /api/bookings``

| Get Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| ``visitorId``  | int | *empty* | Optional visitorId to filter |
| ``hotelId``    | int | *empty* | Optional hotelId to filter   |
| ``fromdate``   | datetime (YYYY-MM-DD) | *empty* | Optionally filter for bookings that are after this date   |
| ``untildate``  | datetime (YYYY-MM-DD) | *empty* | Optionally filter for bookings that are before this date  |

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
[
  {
    "bookingId": 2,
    "checkin": "2024-07-05",
    "checkout": "2024-07-10",
    "hotelId": 2,
    "hotelname": "Contoso Hotel Paris",
    "visitorId": 2,
    "firstname": "Bob",
    "lastname": "Jones",
    "adults": 2,
    "kids": 0,
    "babies": 0,
    "rooms": 1,
    "price": 1000.0
  }
]
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/bookings'
```

#### Bash Curl
```bash
curl -X GET 'http://localhost:8000/api/bookings'
```
</details>


## Get a single Hotel

**Endpoint:** ``GET /api/hotel?hotelId=<int>``

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "hotelId": 6,
   "hotelname": "Contoso Hotel Los Angeles",
   "pricePerNight": 350.0,
   "totalRooms": 100,
   "country": "United States",
   "skiing" : true,
   "suites" : true,
   "inRoomEntertainment" : true,
   "conciergeServices" : true,
   "housekeeping" : true,
   "petFriendlyOptions" : true,
   "laundryServices" : true,
   "roomService" : true,
   "indoorPool" : true,
   "outdoorPool" : true,
   "fitnessCenter" : true,
   "complimentaryBreakfast" : true,
   "businessCenter" : true,
   "freeGuestParking" : true,
   "complimentaryCoffeaAndTea" : true,
   "climateControl" : true,
   "bathroomEssentials" : true
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/hotel?hotelId=2'
```

#### Bash Curl
```bash
curl -X GET 'http://localhost:8000/api/hotel?hotelId=2'
```
</details>


## Get a single Visitor

**Endpoint:** ``GET /api/visitor?visitorId=<int>``

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "firstname": "Frank",
   "lastname": "Green",
   "visitorId": 6
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/visitor?visitorId=2'
```

#### Bash Curl
```bash
curl -X GET 'http://localhost:8000/api/visitor?visitorId=2'
```
</details>


## Get a single Booking

**Endpoint:** ``GET /api/booking?bookingId=<int>``

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "bookingId": 2,
   "visitorId": 6,
   "hotelId": 2,
   "checkin": "2024-07-05",
   "checkout": "2024-07-10",
   "rooms": 1,
   "adults": 2,
   "kids": 0,
   "babies": 0,
   "price": 1000.0
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/booking?bookingId=2'
```

#### Bash Curl
```bash
curl -X GET 'http://localhost:8000/api/booking?bookingId=2'
```
</details>


## Create Hotel

**Endpoint:** ``PUT /api/hotel``

**Request Body:**
```json
{
   "hotelname": "Contoso Hotel Los Angeles",
   "pricePerNight": 350.0,
   "totalRooms": 100,
   "country": "United States",
   "skiing" : true,
   "suites" : true,
   "inRoomEntertainment" : true,
   "conciergeServices" : true,
   "housekeeping" : true,
   "petFriendlyOptions" : true,
   "laundryServices" : true,
   "roomService" : true,
   "indoorPool" : true,
   "outdoorPool" : true,
   "fitnessCenter" : true,
   "complimentaryBreakfast" : true,
   "businessCenter" : true,
   "freeGuestParking" : true,
   "complimentaryCoffeaAndTea" : true,
   "climateControl" : true,
   "bathroomEssentials" : true
}
```

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "hotelId": 6,
   "hotelname": "Contoso Hotel Los Angeles",
   "pricePerNight": 350.0,
   "totalRooms": 100,
   "country": "United States",
   "skiing" : true,
   "suites" : true,
   "inRoomEntertainment" : true,
   "conciergeServices" : true,
   "housekeeping" : true,
   "petFriendlyOptions" : true,
   "laundryServices" : true,
   "roomService" : true,
   "indoorPool" : true,
   "outdoorPool" : true,
   "fitnessCenter" : true,
   "complimentaryBreakfast" : true,
   "businessCenter" : true,
   "freeGuestParking" : true,
   "complimentaryCoffeaAndTea" : true,
   "climateControl" : true,
   "bathroomEssentials" : true
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```


### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/hotel' -Method Put -ContentType 'application/json' -Body (@{
    hotelname = 'Contoso Hotel Los Angeles II'
    pricePerNight = 350.0
    totalRooms = 100
    country = "United States"
    skiing =  $true
    suites =  $true
    inRoomEntertainment =  $true
    conciergeServices =  $true
    housekeeping =  $true
    petFriendlyOptions =  $true
    laundryServices =  $true
    roomService =  $true
    indoorPool =  $true
    outdoorPool =  $true
    fitnessCenter =  $true
    complimentaryBreakfast =  $true
    businessCenter =  $true
    freeGuestParking =  $true
    complimentaryCoffeaAndTea =  $true
    climateControl =  $true
    bathroomEssentials =  $true
} | ConvertTo-Json)
```

#### Bash Curl
```bash
curl -X PUT 'http://localhost:8000/api/hotel' -H 'Content-Type: application/json' -d '{
    "hotelname": "Contoso Hotel Los Angeles II",
    "pricePerNight": 350.0,
    "totalRooms": 100,
    "country": "United States",
    "skiing" : true,
    "suites" : true,
    "inRoomEntertainment" : true,
    "conciergeServices" : true,
    "housekeeping" : true,
    "petFriendlyOptions" : true,
    "laundryServices" : true,
    "roomService" : true,
    "indoorPool" : true,
    "outdoorPool" : true,
    "fitnessCenter" : true,
    "complimentaryBreakfast" : true,
    "businessCenter" : true,
    "freeGuestParking" : true,
    "complimentaryCoffeaAndTea" : true,
    "climateControl" : true,
    "bathroomEssentials" : true
}'
```
</details>

## Create Visitor

**Endpoint:** ``PUT /api/visitor``

**Request Body:**
```json
{
   "firstname": "Frank",
   "lastname": "Green"
}
```

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "visitorId": 6,
   "firstname": "Frank",
   "lastname": "Green"
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/visitor' -Method Put -ContentType 'application/json' -Body (@{
    firstname = 'Frank'
    lastname = 'Blue'
} | ConvertTo-Json)
```

#### Bash Curl
```bash
curl -X PUT 'http://localhost:8000/api/visitor' -H 'Content-Type: application/json' -d '{
    "firstname": "Frank",
    "lastname": "Blue"
}'
```
</details>

## Create Booking

**Endpoint:** ``PUT /api/booking``

**Request Body:**
```json
{
   "visitorId": 6,
   "hotelId": 2,
   "checkin": "2024-07-05",
   "checkout": "2024-07-10",
   "adults": 2,
   "kids": 0,       // optional
   "babies": 0,     // optional
   "rooms": 1,      // optional
   "price": 1000.0  // optional
}
```

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "bookingId": 2,
   "visitorId": 6,
   "hotelId": 2,
   "checkin": "2024-07-05",
   "checkout": "2024-07-10",
   "rooms": 1,
   "adults": 2,
   "kids": 0,
   "babies": 0,
   "price": 1000.0
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/booking' -Method Put -ContentType 'application/json' -Body (@{
    visitorId = 6
    hotelId = 2
    checkin = '2025-07-05'
    checkout = '2025-07-10'
    adults = 2
    kids = 0
    babies = 0
    rooms = 1
    price = 1000.0
} | ConvertTo-Json)
```

#### Bash Curl
```bash
curl -X PUT 'http://localhost:8000/api/booking' -H 'Content-Type: application/json' -d '{
    "visitorId": 6,
    "hotelId": 2,
    "checkin": "2025-07-05",
    "checkout": "2025-07-10",
    "adults": 2,
    "kids": 0,
    "babies": 0,
    "rooms": 1,
    "price": 1000.0
}'
```
</details>

## Update Hotel

**Endpoint:** ``POST /api/hotel``

**Request Body:**
```json
{
   "hotelId": 6,
   "hotelname": "Contoso Hotel Los Angeles",
   "pricePerNight": 350.0,
   "totalRooms": 100,
   "country": "United States",
   "skiing" : true,
   "suites" : true,
   "inRoomEntertainment" : true,
   "conciergeServices" : true,
   "housekeeping" : true,
   "petFriendlyOptions" : true,
   "laundryServices" : true,
   "roomService" : true,
   "indoorPool" : true,
   "outdoorPool" : true,
   "fitnessCenter" : true,
   "complimentaryBreakfast" : true,
   "businessCenter" : true,
   "freeGuestParking" : true,
   "complimentaryCoffeaAndTea" : true,
   "climateControl" : true,
   "bathroomEssentials" : true
}
```

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "hotelId": 6,
   "hotelname": "Contoso Hotel Los Angeles",
   "pricePerNight": 350.0,
   "totalRooms": 100,
   "country": "United States",
   "skiing" : true,
   "suites" : true,
   "inRoomEntertainment" : true,
   "conciergeServices" : true,
   "housekeeping" : true,
   "petFriendlyOptions" : true,
   "laundryServices" : true,
   "roomService" : true,
   "indoorPool" : true,
   "outdoorPool" : true,
   "fitnessCenter" : true,
   "complimentaryBreakfast" : true,
   "businessCenter" : true,
   "freeGuestParking" : true,
   "complimentaryCoffeaAndTea" : true,
   "climateControl" : true,
   "bathroomEssentials" : true
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/hotel' -Method Post -ContentType 'application/json' -Body (@{
    hotelId = 6
    hotelname = 'Contoso Hotel Los Angeles'
    pricePerNight = 350.0
    totalRooms" = 100
    country = "United States"
    skiing = $true
    suites = $true
    inRoomEntertainment = $true
    conciergeServices = $true
    housekeeping = $true
    petFriendlyOptions = $true
    laundryServices = $true
    roomService = $true
    indoorPool = $true
    outdoorPool = $true
    fitnessCenter = $true
    complimentaryBreakfast = $true
    businessCenter = $true
    freeGuestParking = $true
    complimentaryCoffeaAndTea = $true
    climateControl = $true
    bathroomEssentials = $true
} | ConvertTo-Json)
```

#### Bash Curl
```bash
curl -X POST 'http://localhost:8000/api/hotel' -H 'Content-Type: application/json' -d '{
    "hotelId": 6,
    "hotelname": "Contoso Hotel Los Angeles",
    "pricePerNight": 350.0,
    "totalRooms": 100,
    "country": "United States",
    "skiing" : true,
    "suites" : true,
    "inRoomEntertainment" : true,
    "conciergeServices" : true,
    "housekeeping" : true,
    "petFriendlyOptions" : true,
    "laundryServices" : true,
    "roomService" : true,
    "indoorPool" : true,
    "outdoorPool" : true,
    "fitnessCenter" : true,
    "complimentaryBreakfast" : true,
    "businessCenter" : true,
    "freeGuestParking" : true,
    "complimentaryCoffeaAndTea" : true,
    "climateControl" : true,
    "bathroomEssentials" : true
}'
```
</details>


## Update Visitor

**Endpoint:** ``POST /api/visitor``

**Request Body:**
```json
{
   "visitorId": 6,
   "firstname": "Frank",
   "lastname": "Green"
}
```

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "visitorId": 6,
   "firstname": "Frank",
   "lastname": "Green"
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/visitor' -Method Post -ContentType 'application/json' -Body (@{
    visitorId = 6
    firstname = 'Frank'
    lastname = 'Green'
} | ConvertTo-Json)
```

#### Bash Curl
```bash
curl -X POST 'http://localhost:8000/api/visitor' -H 'Content-Type: application/json' -d '{
    "visitorId": 6,
    "firstname": "Frank",
    "lastname": "Green"
}'
```
</details>


## Delete Hotel

**Endpoint:** ``DELETE /api/hotel?hotelId=<int>``

| Url Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| ``hotelId``  | int | *empty* | Required id of the hotel |

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "success": true,
   "deleted": true,  // indicates if deletion was necessary
   "hotelId": 6
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/hotel?hotelId=2' -Method Delete
```

#### Bash Curl
```bash
curl -X DELETE 'http://localhost:8000/api/hotel?hotelId=2'
```
</details>

## Delete Visitor

**Endpoint:** ``DELETE /api/visitor?visitorId=<int>``

| Url Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| ``visitorId``  | int | *empty* | Required id of the visitor |


**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "success": true,
   "deleted": true,  // indicates if deletion was necessary
   "visitorId": 6
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/visitor?visitorId=2' -Method Delete
```

#### Bash Curl
```bash
curl -X DELETE 'http://localhost:8000/api/visitor?visitorId=2'
```
</details>

## Delete Booking

**Endpoint:** ``DELETE /api/booking?bookingId=<int>``

| Url Parameter | Type | Default Value | Description |
| --- | --- | --- | --- |
| ``bookingId``  | int | *empty* | Required id of the booking |


**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "success": true,
   "deleted": true,  // indicates if deletion was necessary
   "bookingId": 2
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/booking?bookingId=2' -Method Delete
```

#### Bash Curl
```bash
curl -X DELETE 'http://localhost:8000/api/booking?bookingId=2'
```
</details>


## Get the amenities

**Endpoint:** ``GET /api/amenities``

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "skiing" : "Skiiing available nearby",
   "suites" : "Has suites available",
   "inRoomEntertainment" : "Flat-screen TVs, streaming services, high-speed Wi-Fi, and Bluetooth speakers",
   "conciergeServices" : "Concierge Services: Assistance with booking tours, restaurant reservations, and other activities",
   "housekeeping" : "Regular cleaning services, often with eco-friendly options",
   "petFriendlyOptions" : "Amenities and services for guests traveling with pets",
   "laundryServices" : "On-site laundry and dry-cleaning services",
   "roomService" : "In-room dining options (available 24/7 in some hotels)",
   "indoorPool" : "Indoor or outdoor pools, hot tubs, saunas, and spa services",
   "outdoorPool" : "Outdoor pools and hot tubs",
   "fitnessCenter" : "Equipped with modern exercise machines and sometimes offering fitness classes",
   "complimentaryBreakfast" : "Often includes a variety of hot and cold options",
   "businessCenter" : "Facilities with computers, printers, and meeting rooms for business travelers",
   "freeGuestParking" : "On-site parking facilities for guests",
   "complimentaryCoffeaAndTea" : "Complimentary coffee and tea kits, along with stocked mini-bars",
   "climateControl" : "Adjustable air conditioning and heating systems",
   "bathroomEssentials" : "Premium toiletries, hair dryers, bathrobes, and slippers"
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/amenities'
```

#### Bash Curl
```bash
curl 'http://localhost:8000/api/amenities'
```
</details>


## Chat with the Chatbot

**Endpoint:** ``GET /api/chat``

**Request Body:**
```json
{
  "chat_history": [
    {
      "inputs": {
        "content": "Hello, how are you?"
      },
      "outputs": {
        "answer": "I am fine, thank you. How can I help you today?"
      }
    },
  ],
  "question": "Does the Contoso Hotel Los Angeles have a pool?"
}
```
**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |
| 502 | Bad Gateway (Issue with prompflow) |

**Example Response Body (Success - 200):**
```json
{
   "answer": "Yes, the Contoso Hotel Los Angeles has an indoor and outdoor pool."
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```


## Setup the Database

**Endpoint:** ``POST /api/setup``

**Request Body:**
```json
{
   "drop_schema"  : false,
   "create_schema": true,
   "populate_data": true,
   "number_of_visitors": 100,       // any number 2 - 10000, default is 100
   "min_bookings_per_visitor": 2,   // any number 0 - 10, default is 2
   "max_bookings_per_visitor" : 5   // any number 1 - 20, default is 5
}
```

**Response Codes:**
| Code | Description |
| --- | --- |
| 200 | Success |
| 400 | Bad Request (Invalid input data) |
| 500 | Internal Server Error (Server side processing error) |

**Example Response Body (Success - 200):**
```json
{
   "success": true,
   "drop_schema": false,
   "create_schema": { 
      "hotels": false,
      "visitors": false,
      "bookings": true 
   },
   "populate_data": { 
      "hotels": false,
      "visitors": true,
      "bookings": true
   },
   "number_of_visitors": 100,
   "min_bookings_per_visitor": 2,
   "max_bookings_per_visitor" : 5
}
```

**Example Response Body (Failure - 400 or 500):**
```json
{ 
   "success" : false,
   "error" : "Some error message here"
}
```

### Example Code
<details>
<summary>Click to expand</summary>

#### PowerShell

```powershell
Invoke-RestMethod -Uri 'http://localhost:8000/api/setup' -Method Post -ContentType 'application/json' -Body (@{
    drop_schema = $true
    create_schema = $true
    populate_data = $true
} | ConvertTo-Json)
```

#### Bash Curl
```bash
curl -X POST 'http://localhost:8000/api/setup' -H 'Content-Type: application/json' -d '{
    "drop_schema": true,
    "create_schema": true,
    "populate_data": true
}'
```
</details>
