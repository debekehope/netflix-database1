## netflix-database1

## 1 Provision the EC2 instance for MongoDB, backend and frontend
   
   *N/B: For best practice use three different SGs for each instance*
   
<img width="1275" height="429" alt="image" src="https://github.com/user-attachments/assets/c23ae7df-fe85-459f-836f-2e8576cde268" />

## 2 Install MongoDB compass
Search for MongoDB database installation on ubuntu for a correct guide
Go to install Mongo db community edition on ubuntu


**Import the public key.**
From a terminal, install gnupg and curl if they are not already available:

```
sudo apt-get install gnupg curl
```

<img width="1043" height="564" alt="image" src="https://github.com/user-attachments/assets/8db38cbc-98bc-4c90-9d41-a66eb52b05e6" />

**To import the MongoDB public GPG key, run the following command:**
```
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```

**Create the list file.**

Create the list file /etc/apt/sources.list.d/mongodb-org-7.0.list for your version of Ubuntu.

Create the list file for Ubuntu 22.04 (Jammy):

```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
<img width="1366" height="66" alt="image" src="https://github.com/user-attachments/assets/5d3b237a-d370-4c68-8528-59e685860a0b" />

**Reload the package database.**
Issue the following command to reload the local package database:

```
sudo apt-get update
```

<img width="906" height="240" alt="image" src="https://github.com/user-attachments/assets/95ca6c11-88d5-4dc9-8f9e-92f8c5577708" />

**Install MongoDB Community Server.**
You can install either the latest stable version of MongoDB or a specific version of MongoDB.

**To install the latest stable version, issue the following**

```
sudo apt-get install -y mongodb-org
```
<img width="1352" height="519" alt="image" src="https://github.com/user-attachments/assets/a7d68b7e-a783-4441-976c-74a7a441be6f" />


**Start MongoDB.**
You can start the mongod process by issuing the following command:

```
sudo systemctl start mongod
```

**Verify that MongoDB has started successfully.**

```
sudo systemctl status mongod
```
<img width="1366" height="405" alt="image" src="https://github.com/user-attachments/assets/4633678a-16a1-4355-a5d6-51fac4deeeea" />


## 3 Enable authentication (critical — this is what usually gets skipped)
Turn on authorization: enabled in mongod.conf.

**To move to MongoDB config**
```
cd /etc/ 
ls -la | grep mongo 
```
We can see a configuration file

<img width="741" height="242" alt="image" src="https://github.com/user-attachments/assets/219edbd1-c35d-471d-81f2-94c9e4b9e642" />

**To go into it, run:**

```
sudo nano mongod.conf
```
So lets use Mobarxterm for the configuration. We log in to our database server with ssh key

**Open the configuration file and change the bind ip and the port number**

```
sudo nano /etc/ mongod.conf
```
<img width="560" height="409" alt="image" src="https://github.com/user-attachments/assets/ef4042a6-12a5-4f0b-98d8-fb781ba02fe4" />

<img width="772" height="282" alt="image" src="https://github.com/user-attachments/assets/f2bfa2af-635b-4aae-b6bb-14dc1a60f34e" />


*N/B: For best practice, set the bindIp in /etc/mongod.conf to the private IP of the database (not 0.0.0.0) so that it only listens on the internal network interface. That is: bindIp: 127.0.0.1,<private-IP-of-the-DB-server>*

127.0.0.1 — lets you connect locally on the server itself (useful for admin tasks, SSH tunneling, mongosh from the box).
172.31.16.20 (or whatever the DB server's own private IP is) — lets it accept connections coming in over the private VPC network, which is how your backend server will reach it.

**Open the new port in the security group**

<img width="1354" height="315" alt="image" src="https://github.com/user-attachments/assets/e3b577ba-80cf-4b46-875b-6ee95cb25539" />

*N/B: For best practice, do not open MongoDB's port (default 27017, or your custom port) to 0.0.0.0/0.*

**Restart your MongoDB**

Restart MongoDB.
You can restart the mongod process by issuing the following command:

```
sudo systemctl restart mongod
```
**Now lets connect to our Mongo DB compass Using the IP address and port number of your database**

<img width="1352" height="713" alt="image" src="https://github.com/user-attachments/assets/71e304ba-a092-4b1f-b94e-970dc3143905" />

We can see the default databases and can create ours from there

<img width="1352" height="713" alt="image" src="https://github.com/user-attachments/assets/98eb3d5a-2fc4-41eb-94b2-8387682d3aaf" />

We can also access the mongosh from here

<img width="1352" height="745" alt="image" src="https://github.com/user-attachments/assets/52f80790-65c1-437f-9ff9-bd91e2a9783f" />


**Now we enable authentication fully so that each time we connect to MongoDB compass we will need a username and password**

<img width="848" height="335" alt="image" src="https://github.com/user-attachments/assets/e4af1754-5547-4ba9-b063-898a37c1716f" />

**Lets try to connect now. We successfully connected but cannot see the databases again because authentication has been enabled**

<img width="1352" height="713" alt="image" src="https://github.com/user-attachments/assets/c376d3e2-609f-459b-a365-b5f81d7b2186" />


**Turn off auth temporarily**

bash

```bash
sudonano /etc/mongod.conf
```

Comment out or remove the security block:

yaml

```yaml
#security:
#  authorization: enabled
```

Save and exit.

**Restart mongod**

bash

```bash
sudo systemctl restart mongod 
sudo systemctl status mongod
```

Confirm it's `active (running)`.

**Connect without credentials and setup the user cleanly**

bash

```bash
mongosh --port 27050
```

Then, one line at a time:

js

```jsx
use admin
```

js

```jsx
db.dropUser("hopeuser")
```

js

```jsx
db.createUser({user:"hopeuser",pwd:"TempPass2026",roles:[{role:"root",db:"admin"}]})
```
<img width="1366" height="721" alt="image" src="https://github.com/user-attachments/assets/1e96717c-b24c-4f89-bbce-e2b291837cf1" />

**Re-enable auth**

bash

```bash
sudo nano /etc/mongod.conf
```

Uncomment/restore:

yaml

```yaml
security:
authorization: enabled
```

Save, then:

bash

```bash
sudo systemctl restart mongod
```

**Test auth immediately**

```
mongosh "mongodb://hopeuser:TempPass2026@98.93.48.186:27050/admin?authSource=admin”
````
<img width="1366" height="326" alt="image" src="https://github.com/user-attachments/assets/e7e09ac4-115d-42b6-8b36-1ee06444100f" />

**Verify you're actually authenticated (not just connected)**

js
```db.getUsers()
```

<img width="1292" height="275" alt="image" src="https://github.com/user-attachments/assets/6e3905e1-cf4d-463f-94b1-0fd29e9329ba" />

## 4 Now log in to mongoDB compass with your username and password credentials. Now you can see the databases and other information

<img width="1352" height="713" alt="image" src="https://github.com/user-attachments/assets/257a09c2-3cba-471a-81b5-bee1768ef32b" />

## 5 Create your own database

*Correction: create a collection name called "movies" because that is what we have set up in our backend code*

<img width="1352" height="713" alt="image" src="https://github.com/user-attachments/assets/ce639c92-0fed-4c18-8fd7-04b639169002" />

Go find your database repo. Copy the code inside the data file and patse it into your database collection. By clicking on add data and pasting the code

<img width="1352" height="713" alt="image" src="https://github.com/user-attachments/assets/74aaf057-b45d-42b2-b761-04620cb63634" />

