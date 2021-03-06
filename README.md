# campsite-booking

[![Build
Status](https://travis-ci.org/igor-baiborodine/campsite-booking.svg?branch=master)](https://travis-ci.org/igor-baiborodine/campsite-booking)

A RESTful web service that manages campsite bookings. 

### Technical Task
#### Booking Constraints
* The campsite can be reserved for max 3 days.
* The campsite can be reserved minimum 1 day(s) ahead of arrival and up to 1 month in advance.
* Reservations can be cancelled anytime.
* For sake of simplicity assume the check-in & check-out time is 12:00 AM.
#### System Requirements
* The users will need to find out when the campsite is available. So the system should expose an API to provide information of the
availability of the campsite for a given date range with the default being 1 month.
* Provide an end point for reserving the campsite. The user will provide his/her email & full name at the time of reserving the campsite
along with intended arrival date and departure date. Return a unique booking identifier back to the caller if the reservation is successful.
* The unique booking identifier can be used to modify or cancel the reservation later on. Provide appropriate end point(s) to allow
modification/cancellation of an existing reservation
* Due to the popularity of the campsite, there is a high likelihood of multiple users attempting to reserve the campsite for the same/overlapping
date(s). Demonstrate with appropriate test cases that the system can gracefully handle concurrent requests to reserve the campsite.
* Provide appropriate error messages to the caller to indicate the error cases.
* The system should be able to handle large volume of requests for getting the campsite availability.
* There are no restrictions on how reservations are stored as as long as system constraints are not violated.

### Running Project
* Default active profile: **h2**
* URL to access Campsite Booking service: **http://localhost:8090/campsite/api/bookings/**
#### With Maven
```bash
$ git clone https://github.com/igor-baiborodine/campsite-booking.git
$ cd campsite-booking
$ mvn spring-boot:run
```
#### With Executable JAR
```bash
git clone https://github.com/igor-baiborodine/campsite-booking.git
cd campsite-booking
mvn package -DskipTests
java -jar target/campsite-booking-0.0.1-SNAPSHOT.jar
```

### Accessing Data in H2 Database
#### H2 Console
URL to access H2 console: **http://localhost:8090/campsite/h2-console**

Fill the login form as follows and click on Connect:
* Saved Settings: **Generic H2 (Embedded)**
* Setting Name: **Generic H2 (Embedded)**
* Driver class: **org.h2.Driver**
* JDBC URL: **jdbc:h2:mem:campsite;MODE=MySQL**
* User Name: **sa**
* Password:

![H2 Console Login](/images/h2-console-login.bmp)
![H2 Console Main View](/images/h2-console-main-view.bmp)

### Exploring API
#### Swagger UI
URL to access Swagger UI: **http://localhost:8090/campsite/swagger-ui.html**

### Testing API
#### With Maven
* Run only unit tests:
```bash
$ mvn clean test
```
* Run unit and integration tests:
```bash
$ mvn clean integration-test
```
* Run only integration tests:
```bash
$ mvn clean failsafe:integration-test
```
* Run any checks on results of integration tests to ensure quality criteria are met:
```bash
$ mvn clean verify
```
#### Unit & Integration Tests Coverage
Results of running of all tests with coverage in IntelliJ IDEA:
![Test Coverage Results](/images/test-coverage-results.bmp)

#### Concurrent Bookings Creation Test
Note: should be executed with **mysql** active profile

To simulate concurrent bookings creation for the same booking dates, create three JSON files with booking data as follows:
```bash
$ cat booking-john-smith-1.json
{
  "email": "john.smith.1@email.com",
  "fullName": "John Smith 1",
  "startDate": "2018-11-11",
  "endDate": "2018-11-13"
}
```
```Bash
$ cat booking-john-smith-2.json
{
  "email": "john.smith.2@email.com",
  "fullName": "John Smith 2",
  "startDate": "2018-11-11",
  "endDate": "2018-11-13"
}
```
```Bash
$ cat booking-john-smith-3.json
{
  "email": "john.smith.3@email.com",
  "fullName": "John Smith 3",
  "startDate": "2018-11-11",
  "endDate": "2018-11-13"
}
```
Then execute the following command to send three concurrent HTTP POST requests:
```Bash
$ curl -H "Content-Type: application/json" -d @booking-john-smith-1.json http://localhost:8090/campsite/api/bookings & curl -H "Content-Type: application/json" -d @booking-john-smith-2.json http://localhost:8090/campsite/api/bookings & curl -H "Content-Type: application/json" -d @booking-john-smith-3.json http://localhost:8090/campsite/api/bookings
```
The response should be as follows after formatting, i.e., only one booking was created:
```json
{  
   "id":2,
   "version":0,
   "email":"john.smith.1@email.com",
   "fullName":"John Smith 1",
   "startDate":"2018-11-11",
   "endDate":"2018-11-13",
   "active":true,
   "_links":{  
      "self":{  
         "href":"http://localhost:8090/campsite/api/bookings/2"
      }
   }
}
{  
   "status":"BAD_REQUEST",
   "timestamp":"2018-11-07T23:02:57.592462",
   "message":"No vacant dates available from 2018-11-11 to 2018-11-13"
}
{  
   "status":"BAD_REQUEST",
   "timestamp":"2018-11-07T23:02:57.618991",
   "message":"No vacant dates available from 2018-11-11 to 2018-11-13"
}
```

#### Basic Load Testing 
Basic load testing for retrieving vacant dates can be performed with the ApacheBench by executing the following command:
```Bash
$ ab -n 10000 -c 100 -k http://localhost:8090/campsite/api/bookings/vacant-dates
```
* **-n 10000** is the number of requests to make
* **-c 100** is the number of concurrent requests to make at a time
* **-k** sends the **KeepAlive** header, which asks the web server to not shut down the connection after each request is done, but to instead keep reusing it

Result:
```
Benchmarking localhost (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        
Server Hostname:        localhost
Server Port:            8090

Document Path:          /campsite/api/bookings/vacant-dates
Document Length:        404 bytes

Concurrency Level:      100
Time taken for tests:   5.760 seconds
Complete requests:      10000
Failed requests:        0
Keep-Alive requests:    0
Total transferred:      5230000 bytes
HTML transferred:       4040000 bytes
Requests per second:    1736.24 [#/sec] (mean)
Time per request:       57.596 [ms] (mean)
Time per request:       0.576 [ms] (mean, across all concurrent requests)
Transfer rate:          886.77 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       3
Processing:     4   57  42.2     47     373
Waiting:        3   53  40.5     43     368
Total:          4   57  42.2     47     373

Percentage of the requests served within a certain time (ms)
  50%     47
  66%     62
  75%     76
  80%     85
  90%    113
  95%    142
  98%    180
  99%    201
 100%    373 (longest request)
```

