# Cadre Query Interface

## Stack

- Backend: Python 3.6.4 and Flask
- Frontend: Vue.js 2.5.21 (Vue CLI 3.3.0) using Node 10.12.0 and 6.4.1

Dependencies can be found in `/backend/requirements.txt` and `/frontend/package.json`.

### Tips

Install Node using NVM:  https://github.com/nvm-sh/nvm/blob/master/README.md

Use Python Virtual Environments:  https://docs.python.org/3/library/venv.html

## Setup and Deployment

### Hosting

The main hosting option used is [AWS Elastic Beanstalk](https://docs.aws.amazon.com/elastic-beanstalk/index.html).  This can be compiled and hosted elsewhere if desired.

### Edit Config Files

```
cd conf
cp example.frontend.config.json frontend.config.json
cp example.backend.config backend.config
```

Update settings as appropriate.  

### Install Frontend

Frontend is built using Vue.js and can be compiled and updated using `npm`

```
cd frontend
npm install
npm run build
```

### Install Backend

The backend is written in Python and uses the Flask microframework.

```
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Start Microserver

When running locally, the flask microservice can be started using the following command.

```
./start.sh
```

In production, the query builder is deployed using AWS's Elastic Beanstalk hosting and does not rely on this start script.  It runs the application directly.

### Bundle for AWS Elastic Beanstalk


After you've got everything running properly through flask, you can bundle the app for deployment on AWS elastic beanstalk by running one of the below commands:

```
./bundle_for_beanstalk.sh
```

or

```
./deploy_to_beanstalk.sh
```

Both of these shell scripts will compile the frontend using `npm run build` and then bundle all the files up into a file called `eb_bundle.zip`.  `bundle_for_beanstalk` will only bundle the files and append a timestamp to the filename.  This file can then be uploaded manually to EBS.  

If configured properly, the `deploy_to_beanstalk` script will also deploy the bundle directly to EBS.  The EBS CLI must be installed and configured in order to use this.

**Note:**  This requires a `/conf/backend.config` file, which is not included in the git repo by default.  All the settings in backend.config will be used for the elastic beanstalk installation.


## File Structure

### Folders

`.ebextensions` contains one file that contains requirements that need to be included on the Beanstalk EC2 instance.

`.elasticbeanstalk` contains a config file that defines settings for the Elastic Beanstalk Environment

`backend` contains the Flask App.  It contains two symbolic links `frontend` and `conf`.  `conf` points directly to the root `/conf` directory.  `frontend` points to `/frontend/dist` where the frontend gets built to

`conf` contains two configuration files that define settings for the frontend and the backend

`frontend` contains the Vue.js application source.  Using the command `npm run build` at the command line will build the application and put it in the `dist` folder.  Many of the files are boilerplate created by the Vue CLI.

## Relation to the rest of CADRE

The query builder relies on the Data API and the Federated Login system.  It connects to the login system via a generated cadre token. 

It also connects to the Meta Database to fetch job data and status.  It uses the psycopg2 library from within the backend.

The interface can send preview queries directly to the database through the data api, or it can submit full queries through the async message queue.

The jobs status page includes a link to the RAC jupyter notebook page which allows users to create a notebook server if not running or go directly to their notebook server.