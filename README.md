## Deploy PostgreSQL fully manageable database for Thingsboard

I installed things board community version in `Dell Server` locally and use `PostgreSQL` database.
In working period I generate some question and try to make a note for future understanding.

```bash
ThingsBoard is able to use SQL or hybrid database approach.
```
- After Installing the Thingsboard CE on Ubuntu machine: What is necessary to back up the whole machine or only the database?

- After installation the thingsboard on our `Ubuntu-machine` we can log thingsboard service.
```
$ less /var/log/thingsboard/thingsboard.log
```
- After that press `Ctrl+n` to view the latest input and check it.

## Backup Database
```
Make a backup folder for the database.
```
- Create a backup folder
```
$ mkdir backup
```
- Enter the backup folder.
```
$ cd backup
```
- Now, we can copy everything to the current folder.
```
$ sudo cp /etc/thingsboard/conf/* ./
```
By `*asterisk` we can copy everything from previous folder to the new folder.

- To check the copy file
```
$ ll 
```
<img src= "IMG/ScreenShort01.png" width=800>

- Basically, we need `thingsboard.yml` and `thingsboard.conf` files.

- After the backup procedure, we need to stop the things board service.

## Configuration

- Service management commands:
```
sudo service postgresql stop
sudo service postgresql start
sudo service postgresql restart
```
- To check the status
```
$ sudo service thingsboard status
```

<img src= "IMG/ScreenShort02.png" width=800>


- Once PostgreSQL is installed we may want to create a new user or set the password for the the main user.   
  The instructions below will help to set the password for main postgresql user.
  
## Note: 
`For password setting we must take the thingsboard datasource password.` 
  
```
  sudo su - postgres
  psql
  \password
  \q
  
``` 

- Enter the database to check the free space of database.

- Use `psql-console` to enter the database
```
$ psql -U postgres -h localhost
```  
- Now we will get the details of database.
```
\l+
```

<img src= "IMG/ScreenShort03.png" width=1000>

- check the disk space, logout from database and write this command.
```
$ df -h /
```

<img src= "IMG/ScreenShort12.png" width=800>


- Make sure we have enough space to place a backup of the database
`Check database size`
```
$ sudo -u postgres psql -c "SELECT pg_size_pretty( pg_database_size('thingsboard') );"
```

- Make sure our PostgreSQL database is running
```
$ pg_lsclusters
```
- If there is enough free space - make a backup.
```
$ pg_dump -U postgres -h localhost -d thingsboard > "specify the file"
```
- Enter the password of `postgres` not our machine.

- Finally, we can check the backup file which is created.
```
$ ll -h  
```
- Restore the folder for any trouble
```
$ pg_restore -U thingsboard -h localhost -d thingsboard > "File name"
```

- To copy the config file
```
$ cd /etc/thingsboard/conf
$ ll
$ sudo mv thingsboard.yml thingsboard.yml.invalid
$ sudo cp ~/backup/thingsboard.yml ./
```

- Now we can see the copy folder
```
$ ll
```
<img src= "IMG/ScreenShort4.png" width=800>

- After finishing the backup processing, to start the thingsoard service.
```
$ sudo service thingsboard start
```
## Midnight Commander

- We can use `midnight commander` compare between our things board files and our backup files. 
```
$ sudo mc
```

## File Storage
- Where are located the files with the application, like configuration files, rule chain, widget and dashboard so on?
- ThingsBoard has ability to store binary content (files) in the DataBase. Currently it is used to store report files generated by Generate Report Node. 
  Stored files are accessed by Send Email Node to create email attachments. Another way to send a file is the REST API Call Node. 
  User can access stored files using Files or Reports widgets which is part of Files Widgets Bundle.

- Enter the thingsboard config file
```
$ cd /etc/thingsboard/conf
$ ll
```
- To check the user share file of thingsboard and look what we have here
```
$ cd /usr/share/thingsboard
$ ll
```

- Enter the "bin" folder
```
$ cd bin
$ ll
```

<img src= "IMG/ScreenShort10.png" width=800>


<img src= "IMG/ScreenShort11.png" width=800>


- Basically it's executable and some instance and installed data etc. We have config file where we will see `data` folder and   here we can see the certificate and some third party related data.
 
- Enter the extension folder, which is use for custom developed role node like rule chains.
```
$ cd extensiions/
$ ll
```
<img src= "IMG/ScreenShort5.png" width=800>

## Clean Database

- How to clean a Database? Or How to purge all data older a specific date, like keep only the last six months data?
- To remove previous data from our database, we can use `PSQL console or Swagger`. Both have some [recommendable commands](https://www.postgresqltutorial.com/postgresql-cheat-sheet/).

## `PSQL console`

- First, open psql console and connect to thingsboard.
- Enter the password of the database and connect with thingsboard. 
```
\c thingsboard
```
- To see all tables,
```
\dt+
```
- And describe time-series,
```
\d ts_kv
thingsboard=# select * from ts_kv limit 5;
```
<img src= "IMG/ScreenShort7.png" width=800>

- Basically, Thingsboard stores all the data to ts_kv table. By below command, we can see the details of ts_kv table. 
```
thingsboard=# select * from ts_kv;
```

<img src= "IMG/ScreenShort13.png" width=800>

- If we want to delete the records from ts_kv table using timestamp,  `like Saturday, January 1, 2022 11:14:40 AM to Saturday, February 1, 2022 11:14:40 AM`,  we can do it by,

```
DELETE FROM ts_kv WHERE "ts" >= '1643673600000' AND "ts" < '1643964657892';
```
<img src= "IMG/ScreenShort14.png" width=800>

- With [epoch time converter](https://www.epochconverter.com/) web application, We can easily convert the timestamp to human readable data.

<img src= "IMG/ScreenShort8.png" width=800>

`Key: We should make a backup file for the database and stop the thingsboard service before work.`

## `Swagger-ui`

- Use a brower to login the thingsboard [swagger service](http://localhost/swagger-ui/).
- Automatically it will open a pop-up and ask authorizations.

<img src= "IMG/ScreenShort6.png" width=800>

- Use bearer token: bearer <past your account token>

  `Key: We don't need to stop thingsboard service because swagger try to communicate with thingsboard server.`
 
## `Clean events`
  
- If we want to clean our role chains, we can clean "events" from database table. 
 ``` 
$ psql -U postgres -h localhost
```  
```  
\c thingsboard
\dt+
```  
<img src= "IMG/ScreenShort9.png" width=800> 
  
## `How can we fix error writing /etc/something.conf: permission denied?`

  - We need to give the sdit permisssion by `sudo chmod`
  - Like: sudo vim /etc/something.conf
  
  ## `PgAdmin4`
  
  - Creating Database in PostgreSQL using user name and password.
  ```
  sudo su - postgres
  psql
  postgres=#
  ```
  - Now create a new database and a user using the following commands.
  
  ```
  postgres=# CREATE USER admin WITH PASSWORD 'admin@postgres';
  postgres=# CREATE DATABASE testdb;
  postgres=# GRANT ALL PRIVILEGES ON DATABASE testdb to admin;
  postgres=# \q
  ```

  - Configuring PostgreSQL Client Authentication
  
  PostgreSQL uses client authentication to decide which user accounts can connect to which databases from which hosts and this is 
  controlled by settings in the client   authentication configuration file, which on Ubuntu is located at /etc/postgresql/12/main/pg_hba.conf.
  
  - Open this file using your favorite text editor as shown.
  
  ```
  sudo vim /etc/postgresql/14/main/pg_hba.conf
  ```
  - Restart the PostgreSQL service.
  
  ```
  sudo systemctl restart postgresql
  ```
  
  ## `Installing pgAdmin4 in Ubuntu`
  -  pgAdmin4 APT repository,
  
  ```
   curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add
   sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'
  ```
  - Install pgAdmin4,
  ```
  sudo apt install pgadmin4
   ```
  - Email address and set a strong secure password as well:
  
  ```
  sudo /usr/pgadmin4/bin/setup-web.sh
  ```
  
 - Accessing pgAdmin4 Web Interface
  ```
  http://127.0.0.1/pgadmin4
  ``` 
  
  
🚩 Connect with me on social
- LinkedIn: [LinkedIn](https://www.linkedin.com/in/ariful-islam-arif-2987b51a3/)
- Twitter: [Twitter](https://twitter.com/arifulislam301)
- Instagram: [Instagram](https://www.instagram.com/ariful_mr_islam/)

🔔 Subscribe to my YouTube channel: [YouTube](https://www.youtube.com/channel/UCED68cm6nHaAlAk0h9I3yAQ)

