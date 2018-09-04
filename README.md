# hoverfly-demo-microservices

Using hoverfly to simulate network latency for microservice applications

User Guide: Hoverfly Demo utilizing "Cinema" microservices

Setup and Installation:

1. Download the "Cinema" microservices project from GitHub

`git clone https://github.com/mmorejon/microservices-docker-go-mongodb.git`

2. Setup and Install "Cinema"

`mkdir microservices-docker-go-mongodb/`
`cd microservices-docker-go-mongodb/`

3. Add this line to "hosts" database file:

`127.0.0.1   movies.local bookings.local users.local showtimes.local`

4. Ensure Docker is up and running and execute the compose command

`docker-compose up -d`
`docker container ls`

4. Docker should return a similar list:

CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                      NAMES
ce2acde9c7c0        cinema/users          "/bin/sh -c /go/bin/…"   3 weeks ago         Up 11 seconds       8080/tcp                   cinema-users
132a9dc70dae        cinema/showtimes      "/bin/sh -c /go/bin/…"   3 weeks ago         Up 11 seconds       8080/tcp                   cinema-showtimes
216f4b43c958        cinema/bookings       "/bin/sh -c /go/bin/…"   3 weeks ago         Up 11 seconds       8080/tcp                   cinema-bookings
70c83dba73f1        cinema/movies         "/bin/sh -c /go/bin/…"   3 weeks ago         Up 11 seconds       8080/tcp                   cinema-movies
d6fa443af7f4        mongo:3.3             "/entrypoint.sh mong…"   3 weeks ago         Up 12 seconds       0.0.0.0:27017->27017/tcp   cinema-db
ac4bc47c0e3f        jwilder/nginx-proxy   "/app/docker-entrypo…"   3 weeks ago         Up 12 seconds       0.0.0.0:80->80/tcp         cinema-nginx-proxy

5. Use brew to install Hoverfly

- Hoverfly Key Concepts:

https://hoverfly.readthedocs.io/en/latest/pages/keyconcepts/proxyserver.html

- Installing Hoverfly:

https://hoverfly.readthedocs.io/en/latest/pages/introduction/downloadinstallation.html

`brew install SpectoLabs/tap/hoverfly`

6. Ensure that hoverctl and hoverfly are installed in same directory and can communicate

`hoverctl version`
`hoverfly -version`

7. Note the differences between the two binaries
- Hoverctl and Hoverfly:
	Hoverfly is composed of two binaries: Hoverfly and hoverctl.
	hoverctl is a command line tool that can be used to configure and control Hoverfly. It allows you to run Hoverfly as a daemon.
	Hoverfly is the application that does the bulk of the work. It provides the proxy server or webserver, and the API endpoints.
	Once you have extracted both Hoverfly and hoverctl into a directory on your PATH, you can run hoverctl and Hoverfly.

8. Use hoverctl to capture web-traffic from "Cinema" microservice

`hoverctl start`
`hoverctl mode capture`

- In "capture" mode, Hoverfly will read and record traffic based on Request - Response parings (recording what is sent as well as what is recieved)
- Read more about Request - Response pairs: https://hoverfly.readthedocs.io/en/latest/pages/keyconcepts/simulations/pairs.html#

9. Perform a cURL request and push through the default Hoverfly proxy of 8500

`curl --proxy localhost:8500 http://users.local/users`

- Hoverfly has now lisened to traffic, after routing through it's listening port of 8500, and has recorded the details.

10. Export Hoverfly recorded data for manipulation

`hoverctl export fileName.json`

11. Open exported file and mockup contents to create a delay for the exact request we listening to. In this case, we have recorded the GET request for the cinema/users microservice
- See cinemaUsersMock.json file provided for example of how to add a "Delay" function to the captured file
- How to add delays: https://hoverfly.readthedocs.io/en/latest/pages/keyconcepts/simulations/delays.html
- Add a significant delay for demo purposes (5000 to 10000)
- A python script can be used to randomly create delays, but is out of scope of this demo

12. Load mock delay data and simluate a webserver delay using real Request - Response pairing from step 9

- Delete any previous runs with "stop", "start", "delete"

`hoverctl stop`
`hoverctl start`
`hoverctl delete`

- Ensure Hoverfly is in simluate mode (simulating webservice)

`hoverctl mode simulate`

- Load mock data

`hoverctl import fileName.json`

- Run exact step from step 9

`curl --proxy localhost:8500 http://users.local/users`

- In this case, we route the request through Hoverfly again, but instead of capturing network traffic, Hoverfly uses Request - Response pairing to mimick the webservers response.
- Note the added delay to the response by our mock webserver.