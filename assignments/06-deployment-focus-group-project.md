# Assignment 6: Deployment Focus (Individual)

**THIS IS NOT THE FINAL VERSION FOR SPRING 2020** W

**WORK AHEAD AT YOUR OWN RISK**

## Phase One

1. Create an amazon ec-2 instance (micro) using the Amazon AMI version of linux
2. Login to server with pem key you create (and will need to `chmod 400`) using this command, for example, `ssh -i otf.pem ec2-user@ec2-54-209-155-106.compute-1.amazonaws.com`\
3. Install python 3: `sudo yum install python3 -y`
4. Install virtualenv `sudo pip3 install virtualenv`
5. `mkdir github`
6. `cd github`
7. `mkdir virtualenvs`
8. `virtualenv --python=python3 virtualenvs/myenv`
9. `source virtualenvs/myenv/bin/activate`
10. `sudo yum install git`
11. `git clone https://github.com/chaoss/augur`
12. `cd augur`
13. `sudo yum install postgresql postgresql-server postgresql-devel postgresql-contrib postgresql-docs`
14. `sudo postgresql-setup initdb`
15. `sudo vim /var/lib/pgsql/data/pg_hba.conf`


Update the bottom of the file, which will read something like this, by default:
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     ident
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```

To read this:
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
# local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5 
# IPv6 local connections:
# host    all             all             ::1/128                 ident

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     trust
local   all     all                 md5
# IPv4 local connections:
host    all             power_user      0.0.0.0/0               md5
host    all             other_user      0.0.0.0/0               md5
host    all             storageLoader   0.0.0.0/0               md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            ident
#host    replication     postgres        ::1/128                 ident
```


Now that we've updated the authorization settings, we need to update PostgreSQL to enable remote connections to the database. At the command line enter:

16. `sudo vim /var/lib/pgsql/data/postgresql.conf`
17. Uncomment line 59:
```
#listen_addresses = 'localhost'          # what IP address(es) to listen on;
```
18. And update the line to enable connections from any IP address:
```
listen_addresses='*'
```
19. And uncomment line 63:
```
#port = 5432
```
So it reads
```
port = 5432
```
20. Now start the server: `sudo service postgresql start`
21. Create your Augur database
```
    sudo su - postgres 
    psql
    postgres=# create database augur;
    postgres=# create user augur with encrypted password 'mypass';
    postgres=# grant all privileges on database augur to augur;

```
22. Exit the Postgres interface and return to be your user. 
```
postgres-# \q
postgres@js-104-101:~$ exit
logout
(project4) sgoggins@js-104-101:~/augur$ 

```
23. Install NodeJS: (Notice the "." at the start of line 2. Important)
``` 
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
. ~/.nvm/nvm.sh
nvm install node

## Test that Node.js is installed and running correctly by typing the following at the command line.

node -e "console.log('Running Node.js ' + process.version)"
```
24. `git checkout dev` to switch to Augur's dev branch. 
24. Install and compile augur by doing a `make install`
```
**********************************
Setting up the database configuration...
**********************************

If you need to install Postgres, the downloads can be found here: https://www.postgresql.org/download/


1) Would you like create the Augur database, user and schema LOCALLY?
2) Would you like to add the Augur schema to an already created Postgres 10 or 11 database (local or remote)?
3) Would you like to connect to a database already configured with Augur's schema? 

Please type the number corresponding to your selection and then press the Enter/Return key.
Your choice: 2
Please set the credentials for your database.
Database: augur
Host: localhost
Port: 5432
User: augur
Password: mypass

Additionally, you'll need to provide a valid GitHub API key.
** This is required for Augur to gather data ***
GitHub API Key: (get yours here: https://github.com/settings/tokens)


The Facade data collection worker needs to clone repositories to run its analysis.
Would you like to use an existing directory, or create a new one?
1) Create a new directory
2) Use an existing directory

Please type the number corresponding to your selection and then press the Enter/Return key.
Your choice: 1
** You MUST use an absolute path. **
Desired directory name: /home/ec2-user/repos/
That directory already exists. Continuing...

**********************************
Generating configuration file...
**********************************

2020-02-21 18:08:22 ip-172-31-87-9.ec2.internal augur[6361] INFO Couldn't open augur.config.json, attempting to create.
augur.config.json successfully created
-- ----------------------------
CREATE SCHEMA augur_data;
CREATE SCHEMA
CREATE SCHEMA augur_operations;
CREATE SCHEMA
CREATE SCHEMA spdx;
CREATE SCHEMA
-- create the schemas

```

**/home/ec2-user/repos/**

```
Schema created
Would you like to load your database with some sample data provided by Augur?
1) Yes
2) No

Please type the number corresponding to your selection and then press the Enter/Return key.
Your choice: 1
```

```
Would you like to install Augur's frontend dependencies?

1) y
2) n

Please type the number corresponding to your selection and then press the Enter/Return key.
Your choice: 1

```

## Front end will FAIL on a micro. THAT IS OK

```
<--- JS stacktrace --->

==== JS stack trace =========================================

    0: ExitFrame [pc: 0x13cd419]
Security context: 0x0df910cc0921 <JSObject>
    1: _(aka _) [0x211c8a53ce29] [/home/ec2-user/github/augur/frontend/node_modules/terser/dist/bundle.min.js:~1] [pc=0x1cd66e5d34e0](this=0x22535b5404b9 <undefined>,0x08c6257bc939 <String[#4]: punc>,0x07505ce8e5a9 <String[#1]: :>,0x22535b5404b9 <undefined>)
    2: ee [0x211c8a53a8f9] [/home/ec2-user/github/augur/frontend/node_modules/terser/dist/bundle.min.js:~1...

FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory

Writing Node.js report to file: report.20200221.181408.6698.0.001.json
Node.js report completed
 1: 0xa06760 node::Abort() [node]
 2: 0xa06ba1 node::OnFatalError(char const*, char const*) [node]
 3: 0xb74cee v8::Utils::ReportOOMFailure(v8::internal::Isolate*, char const*, bool) [node]
 4: 0xb75069 v8::internal::V8::FatalProcessOutOfMemory(v8::internal::Isolate*, char const*, bool) [node]
 5: 0xd23575  [node]
 6: 0xd23c06 v8::internal::Heap::RecomputeLimits(v8::internal::GarbageCollector) [node]
 7: 0xd32479 v8::internal::Heap::PerformGarbageCollection(v8::internal::GarbageCollector, v8::GCCallbackFlags) [node]
 8: 0xd332b5 v8::internal::Heap::CollectGarbage(v8::internal::AllocationSpace, v8::internal::GarbageCollectionReason, v8::GCCallbackFlags) [node]
 9: 0xd35d8c v8::internal::Heap::AllocateRawWithRetryOrFailSlowPath(int, v8::internal::AllocationType, v8::internal::AllocationOrigin, v8::internal::AllocationAlignment) [node]
10: 0xcfc964 v8::internal::Factory::NewFillerObject(int, bool, v8::internal::AllocationType, v8::internal::AllocationOrigin) [node]
11: 0x1049c8e v8::internal::Runtime_AllocateInYoungGeneration(int, unsigned long*, v8::internal::Isolate*) [node]
12: 0x13cd419  [node]
util/scripts/install/frontend.sh: line 20:  6687 Aborted                 npm run build
**********************************
*** INSTALLATION COMPLETE ***

```

## Run the back end of augur
```
(myenv) [ec2-user@ip-172-31-87-9 augur]$ nohup augur run >augur.log 2>augur.err &

```

### Output you should see
```
(myenv) [ec2-user@ip-172-31-87-9 augur]$ cat augur.log
(myenv) [ec2-user@ip-172-31-87-9 augur]$ cat augur.err
nohup: ignoring input
2020-02-21 18:18:01 ip-172-31-87-9.ec2.internal augur[6922] INFO Booting broker and its manager...
2020-02-21 18:18:02 ip-172-31-87-9.ec2.internal augur[6922] INFO Booting housekeeper...
/home/ec2-user/github/augur/augur/housekeeper/housekeeper.py:327: UserWarning: DataFrame columns are not unique, some columns will be omitted.
  reorganized_repos = reorganized_repos.to_dict('records')
2020-02-21 18:18:02 ip-172-31-87-9.ec2.internal root[6922] INFO Starting update processes...
2020-02-21 18:18:02 ip-172-31-87-9.ec2.internal augur[6922] INFO Starting server...
[2020-02-21 18:18:02 +0000] [6922] [INFO] Starting gunicorn 19.9.0
[2020-02-21 18:18:02 +0000] [6922] [INFO] Listening at: http://0.0.0.0:5000 (6922)
[2020-02-21 18:18:02 +0000] [6922] [INFO] Using worker: sync
[2020-02-21 18:18:02 +0000] [6985] [INFO] Booting worker with pid: 6985
[2020-02-21 18:18:02 +0000] [6986] [INFO] Booting worker with pid: 6986
[2020-02-21 18:18:03 +0000] [6987] [INFO] Booting worker with pid: 6987
[2020-02-21 18:18:03 +0000] [6988] [INFO] Booting worker with pid: 6988
(myenv) [ec2-user@ip-172-31-87-9 augur]$ 
```

========

## Submission
1. Screenshot of SSH inside EC2 Instance
2. Screenshot inside virtualenv
3. Screenshot of python and node version
4. Screenshot after: 
<br> `sudo service postgresql start`
<br> `sudo su - postgres `
<br> `psql`
5. Navigate to augur folder. Screenshot after `cat augur.err` (Only the top)






