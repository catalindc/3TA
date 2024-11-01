# Three-Tier Application Setup

## Overview

This document outlines the steps to set up and run the three-tier application locally, including the backend service, PostgreSQL database, and frontend application.

## Prerequisites

- Node.js installed
- PostgreSQL installed
- Docker (optional for containerized setup)
- `npm` installed
- Basic knowledge of terminal commands

## Project Structure

~/3tierapp/ ├── app/ │ ├── db/ │ ├── service/ │ └── app/ ├── docker-compose.yml └── readme.md


## Step 1: Set Up PostgreSQL Database

1. **Create a Database:**
   ```bash
   sudo -u postgres psql
   CREATE DATABASE demo_db;
   CREATE USER demo_user WITH PASSWORD 'your_password';
   GRANT ALL PRIVILEGES ON DATABASE demo_db TO demo_user;
   \q

Update pg_hba.conf: Ensure the authentication method for the user demo_user is set to md5 in /etc/postgresql/{version}/main/pg_hba.conf:

local   all             demo_user                               md5


Restart PostgreSQL: sudo systemctl restart postgresql 

Step 2: Set Up Backend Service 
Navigate to the service directory
Create an Environment File: Create a file named .env with the following content:CONNECTION_STRING=postgres://demo_user:your_password@localhost:5433/demo_db

Install Dependencies: npm install 

Create the Server File: Create a file named server.js with the following content:

const express = require('express');
const { Pool } = require('pg');
require('dotenv').config();
const app = express();
const port = 3001;

const pool = new Pool({
    connectionString: process.env.CONNECTION_STRING
});

app.get('/data', function(req, res) {
    pool.query('SELECT county, city FROM county_and_city', [], (err, result) => {
        if (err) {
            return res.status(405).jsonp({ error: err });
        }
        return res.status(200).jsonp({ data: result.rows });
    });
});

app.listen(port, () => console.log(`Backend REST API listening on port ${port}!`));

Start the Backend Service: npm start 

Step 3: Set Up Frontend Application
Navigate to the app directory:
Create an Environment File: Create a file named variable.env with the following content: API_URL=http://localhost:3001/data

Create the Index File: Create a file named index.js with the following content: 
const express = require('express');
const request = require('request');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
    request(process.env.API_URL, (error, response, body) => {
        if (error) {
            return res.status(500).send('Error connecting to backend API');
        }
        res.send(body);
    });
});

app.listen(port, () => console.log(`Frontend app listening on port ${port}!`));

Install Frontend Dependencies:npm install
Start the Frontend Application: npm start 

Step 4: Testing the Application
Test the Backend API: In a new terminal window, run:curl http://localhost:3001/data

You should see data retrieved from the county_and_city table.

Test the Frontend Application: Open your web browser and navigate to http://localhost:3000. The frontend should display data retrieved from the backend.
