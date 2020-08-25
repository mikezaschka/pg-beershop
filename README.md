# CAP Beershop using PostgreSQL for persistence

## Local execution

### Prerequisites

To get started quickly you need docker and docker-compose.

### Setup

To run the example with a local PostgreSQL DB in docker create a `default-env.json` file with the following content:

```JSON
{
  "VCAP_SERVICES": {
    "postgres": [
      {
        "name": "postgres",
        "label": "postgres",
        "tags": [
          "PostgreSQL"
        ],
        "credentials": {
          "host": "localhost",
          "port": "5432",
          "database": "beershop",
          "user": "postgres",
          "password": "postgres"
        }
      }
    ]
  }
}
```

Start the PostgreSQL database and [Adminer](https://www.adminer.org/) using:

`npm run start:docker`

Then open [http://localhost:8080/](http://localhost:8080/) and login by selecting System *PostgreSQL*, Username *postgres* and Password *postgres*. Create a new database *beershop* using the *Create database* link. Then execute the SQL commands you find in *beershop.sql*.

Now you can start the CAP application by using:

`cds run`

Then open <http://localhost:4004/beershop/Beers> in the browser and you should see:

```JSON
{
  "@odata.context": "$metadata#Beers",
  "value": [
    {
      "ID": "b8c3fc14-22e2-4f42-837a-e6134775a186",
      "name": "Lagerbier Hell",
      "abv": 5.2,
      "ibu": 12,
      "brewery_ID": "9c937100-d459-491f-a72d-81b2929af10f"
    },
    {
      "ID": "9e1704e3-6fd0-4a5d-bfb1-13ac47f7976b",
      "name": "Schönramer Hell",
      "abv": 5,
      "ibu": 20,
      "brewery_ID": "fa6b959e-3a01-40ef-872e-6030ee4de4e5"
    }
  ]
}
```

## Execution on SAP Cloud Platform

Until [SAP will provide a fully managed PostgreSQL DB](https://blogs.sap.com/2020/02/11/consuming-hyper-scaler-backing-services-on-sap-cloud-platform-an-update/) you need to provide your on PostgreSQL DB. One way is to install a [Open Service Broker](https://www.openservicebrokerapi.org/). The page [Compliant Service Brokers](https://www.openservicebrokerapi.org/compliant-service-brokers) lists brokers supporting AWS, Azure and GCP. The SAP Developers Tutorial Mission [Use Microsoft Azure Services in SAP Cloud Platform](https://developers.sap.com/mission.cp-azure-services.html) describes in great detail how to setup the Service Broker for Azure. When you finished this setup you can run:

`npm run create-service:pg:dbms`

to instanciate a PostgreSQL DBMS. Then run:

`npm run create-service:pg:db`

to create a the beershop database in the DBMS. With that opreperation done you can build the MTA by running:

`npm run build:mta`

That MTA can be deployed using:

`npm run deploy:cf`

The created database is empty. As currently no deploy script is available the needed tables and views for the CAP application need to be created before you can run the application. The easiest way to create the tables and views is to use Adminer as for the local deployment. You can get the credentials by opening the pg-beershop-srv application via the SAP Cloud Platform Cockpit. Navigate to the Service Bindings and click on "Show sensitive data". Enter the data in the corresponsing fields of the Adminer login screen. Execute the SQL commands you find in *beershop.sql*. To fill the database with data also execute the ones in *beershop-data.sql*. Now try out the URL you find in the Overview of the pg-beershop-srv application.

## Features

### Convert SQL generated from cds compile to PostgreSQL

When you run:

`npm run compile:tosql`

the CDS model will be compiled to the *beershop-cds.sql* file. Using the script *cdssql2pgsql.js* this SQL is converted to support PostgreSQL. Currently only the datatype NVARCHAR must be replaced with VARCHAR.

### Import CDS files from db/data into the beershop database

The path db/data is mounted to the docker container at /tmp/data. That allows to run the COPY commands generated at the end of *beershop.sql*.

## Limitations

### jest tests not working yet

Running the jest test with `npm test` currently fails with:

```bash
    Failed: Object {
      "code": -20005,
      "message": "Failed to load DBCAPI.",
      "sqlState": "HY000",
      "stack": "Error: Failed to load DBCAPI.
```

when running the standalone script `node test/test-db.js` that uses the same way to connect everything is OK.

## Ideas

### Schema Migrations

- <https://flywaydb.org/>
- <https://www.liquibase.org/>
