# Technical Overview

## Architecture
This repository is a monorepo of independent, self-contained Oracle Database samples. Examples are grouped primarily by programming language (for client access patterns) and by Oracle feature area (for database capabilities). Most folders can be studied and run in isolation, with minimal cross-dependencies.

In the “app-like” samples, the structure typically follows layered boundaries:
- Node.js sample apps often resemble: HTTP routes/controllers that validate requests and shape responses, services that implement business logic, and database access modules (often named `db_apis`) that execute SQL/PLSQL against Oracle Database via the driver.
- Java sample apps often resemble: Controller/resource layer (Servlet/JAX-RS/Spring), a service layer for business logic, and a DAO/repository layer that uses JDBC and optionally UCP (Universal Connection Pool) to efficiently manage connections to Oracle Database.

## Technologies used
The repo demonstrates Oracle Database development across multiple languages, tools, and feature stacks:
- Core database technologies: Oracle Database, SQL, and PL/SQL.
- Client languages and drivers: Java (JDBC, UCP), Node.js (node-oracledb), Python (python-oracledb), Ruby, and C/OCI (plus precompiler examples).
- Oracle developer tools and runtimes: Oracle APEX, SQL Developer (including extension examples), and related scripting assets.
- Feature-focused areas covered by samples include (but are not limited to): Spatial, Oracle Text, Oracle Machine Learning (OML), TxEventQ/TEQ (database-resident queues with client APIs), Optimizer and performance tuning, Security (including SQL Firewall), Exadata, and JSON Relational Duality.

## Database interaction patterns
Across languages, the common pattern is: client code uses a database driver to open a connection (often via a connection pool), execute SQL/PLSQL, and map results into application objects or response payloads.
- Connection pooling is commonly demonstrated where it is idiomatic or required for throughput, such as UCP in Java and pooling in node-oracledb and python-oracledb.
- REST-style flows appear in various samples, where HTTP endpoints call service logic that ultimately executes SQL/PLSQL in Oracle Database.
- Messaging examples (TxEventQ/TEQ) use queues that live inside Oracle Database and are accessed via Java/JMS or Kafka-like client patterns, enabling transactional, database-integrated eventing.
- APEX examples run through the APEX engine, interacting with database objects; in typical deployments, APEX applications are served through ORDS and ultimately execute SQL/PLSQL in the database.

## Key components (top-level modules)
The repository is organized into these major entry points:
- Language collections: `C/`, `java/`, `javascript/`, `python/python-oracledb/`, `ruby/`, `sql/`, `plsql/`, `precompilers/`
- Feature areas: `txeventq/`, `sagas/`, `spatial/`, `text/`, `optimizer/`, `machine-learning/`, `security/`, `sql-firewall/`, `exadata/`, `json-relational-duality/`
- Tooling and UI: `apex/`, `sqldeveloper/extension/`
- Reference schemas: the README links to `db-sample-schemas` (Oracle Database Sample Schemas) as a related reference commonly used by examples.
