## Docker Db(mongodb) + Backend + Frontend

### 1. Create Shared Network
```bash
docker network create netmain
```

### 2. Run MongoDB Container
```bash
#docker run --name cntmongodb --rm -d --network netmain mongo
docker run --name cntmongodb -v mongopersistdata:/data/db --rm -d --network netmain -e MONGO_INITDB_ROOT_USERNAME=myusername -e MONGO_INITDB_ROOT_PASSWORD=mypassword mongo
```

### 3. Build & Run Backend Container
```bash
docker build -t imgbackend .
#docker run --name cntbackend --rm -d -p 8080:80 --network netmain imgbackend
#before in the code (row 87) replace 'host.docker.internal' w <namecontainertarget> + rebuild image, still need port 8080 x the frontend(bc is a pure front-side no server)
#(row 89) use 'mongodb://myusername:mypassword@cntmongodb:27017/course-goals?authSource=admin' to connect w auth(se prima ti eri connesso senza auth, mongodb ha creato un volume settato a 'noauth'; per accedere a mongodb con auth elimina quel volume + run cntmongodb)

docker run --name cntbackend --rm -d -p 8080:80 -v "${PWD}:/app:ro" -v logs:/app/logs -v /app/node_modules -e MONGODB_USERNAME=myusername -e MONGODB_PASSWORD=mypassword --network netmain imgbackend
#console in position within dir 'backend' x absolute path given by PWD(powershell, %cd% x cmd.exe)
#abilitate live-changes(on Windows, linux no need '--legacy-watch' in file package.json start mode) sourcecode->cnt thanks to bind mounts + use env vars x pass mongo credentials to cntmongodb
```

### 4. Build & Run Frontend Container
```bash
docker build -t imgfrontend .
#docker run --name cntfrontend --rm -d -p 3000:3000 -it imgfrontend
#not necessary '--network' bc this is a pure front-side
#il cnt frontend(also when using --network) serve Reactjs all'esterno su porta 3000, ma se hai settato nel front 'const API_URL = 'http://cntbackend';' (row 12) il browser(su localhost:3000) lo prova a fetchare come se fosse un dominio pubblico, dando errore!! quindi: o crei un mini-server(usando nodejs/express) sul frontend cosi da poter usare 'cntbackend', altrimenti ESPONI PORT SUL BACKEND WHEN CREATE CNT BACKEND e raggiungila dal frontend con 'const API_URL = 'http://localhost:8080';'.

docker run --name cntfrontend --rm -d -p 3000:3000 -it -e CHOKIDAR_USEPOLLING=true -e WATCHPACK_POLLING=true -v "${PWD}/src:/app/src:ro" imgfrontend
#console in position within dir 'frontend' x absolute path given by PWD(powershell, %cd% x cmd.exe)
#abilitate live-changes (on Windows,) sourcecode->cnt using bind mounts(only x folder /src not entire frontend)
```

### 5. Test Application
- **Access the App** 
    - Open your browser and visit: 👉 http://localhost:3000/ to verify that all three Docker containers (MongoDB, Backend, Frontend) are working correctly.
- **Test MongoDB Data Persistence** 
    - On the webpage, create some tasks/goals.
    - Stop the backend container
    - Refresh the page in the browser — you’ll see an error (as expected, the backend is down).
    - Restart the backend by running a new container.
    - Refresh the page again: you will see the tasks you created earlier! 
- **Test Live Changes Backend sourcecode->cnt (docker bind mounts + plugin nodemon live-server)** 
    - Run the backend container.
    - `docker logs cntbackend` check message 'CONNECTED TO MONGODB'.
    - file app.js row 102, change the log message 'CONNECTED TO MONGODB' to 'CONNECTED TO MONGODB!!' and save.
    - `docker logs cntbackend` check the updated message of the container.
- **Test Live Changes Frontend sourcecode->cnt (docker bind mounts)**
    - Run the frontend container.
    - Check rendered button with text 'Add Goal' on the web page.
    - Open file `frontend/src/components/goals/GoalInput.js` and at line 37 change the text 'Add Goal' to 'Add Goal mytest'.
    - Check rendered button text updated on the web page.

### 6. Stop All Containers
```bash
docker stop cntmongodb
docker stop cntbackend
docker stop cntfrontend
```

![Reference1](./readmefiles/dbbackfront1.png)