## Docker Db(mongodb) + Backend + Frontend

### 1. Create Shared Network
```bash
docker network create netmain
```

### 2. Run MongoDB Container
```bash
docker run --name cntmongodb --rm -d --network netmain mongo
```

### 3. Build & Run Backend Container
```bash
docker build -t imgbackend .
docker run --name cntbackend --rm -d -p 8080:80 --network netmain imgbackend
#before in the code replace 'host.docker.internal' w <namecontainertarget> (row 87)+ rebuild image, still need port 8080 x the frontend(bc is a pure front-side no server)
```

### 4. Build & Run Frontend Container
```bash
docker build -t imgfrontend .
docker run --name cntfrontend --rm -d -p 3000:3000 -it imgfrontend
#not necessary '--network' bc this is a pure front-side
```

### 5. Test Application
Open your browser and go to:
👉 http://localhost:3000/
to verify that all three Docker containers are working correctly.

### 6. Stop All Containers
```bash
docker stop cntmongodb
docker stop cntbackend
docker stop cntfrontend
```