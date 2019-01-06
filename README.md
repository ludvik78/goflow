# **This is pre-release, Alpha software. Use at your own risk!**
# Description
<p align="center">
    <img src="https://i.imgur.com/HdIxEOB.png">
</p>
This is a very early release of Goflow, a golang-based netflow collector with a flexible backend.

A list of upcoming features can be found under the issue tracker for this project.

Currently supported frontend/backend combinations are 

Frontend | Backend
------ | ------ |
Netflow | Mysql 
Netflow | Timescaledb
Netflow | Apache Kafka


# Prereqs

You need a running backend and associated connection information;

- Server fqdn 
- Username
- Password
- Database name

For Mysql, you could use the free tier of Amazon RDS to get started.

See the SETUP.md instructions within the backend directories in this repo for help
specific to each backend. (i.e ./backends/timescale/SETUP.MD)

# Installation
Goflow requires two files to run;
 - goflow, the binary itself
 - config.yml, the configuration file

The tar releases contain both these.

```bash
# Extract and set perms
tar -xzvf goflow.Linux.AMD64.tar.gz
chmod +x goflow
mv config_example.yml config.yml
# Edit the config.yml file to make specific to your environment
vi config.yml
# Export the required environment variables
export SQL_PASSWORD=your_sql_pw_here
# Run
./goflow
```

In the future, an installation script will be packaged for most systems but for now, you will need to create your own systemd or init scripts to start it.

# Monitoring and utilities
The goflow binary doubles as a client interface, and a JSON API is started at the same time as the daemon.

The API listens on localhost by default, but this can be tuned (see the configuration example).

The Goflow API is not for retrieving flow data, but performing ongoing maintenence and ops on the collector itself.

Goflow help displays a list of options.
```bash
./goflow help
```

# Integrations
## Grafana

Goflow integrates natively (i.e - no plugins required) with Grafana when using the Timescale backend type.
 
Grafana transforms the underlying Postgres database into a set of  pretty graphs!

_Note: dummy data shown_

<a href="https://imgur.com/2VTol8w"><img width="300px" src="https://i.imgur.com/2VTol8w.png" title="source: imgur.com" /></a>
<a href="https://imgur.com/DdIhG6k"><img width="300px" src="https://i.imgur.com/DdIhG6k.png" title="source: imgur.com" /></a>
<a href="https://imgur.com/WWpPKch"><img width="300px" src="https://i.imgur.com/WWpPKch.png" title="source: imgur.com" /></a>

For convenience, Goflow provides the Dashboards (/grafana_db/*.json) and some code to setup Grafana correctly.

You need:
* A timescale backend, already configured in config.yml
* A running grafana instance
* <a href="http://docs.grafana.org/http_api/auth/">An API key</a> 

With these requirements met, the dashboards/datasources can be setup as below.
```bash
# Make sure the required env-var for timescale is exported
export SQL_PASSWORD=your-sql-password
./goflow configure-grafana http://[ your-grafana-server ] [ your-api-key ] [ dashboard-directory ]
```

# Performance
Each release of Goflow is benchmarked in a test environment. 

Currently the most efficient backend is **timescaledb**.

The environment setup is;
* Type: AWS T2.Micro
* CPU: 1vCPU
* Memory: 1GB
* Storage: EBS SSD (Non-provisioned)


<img src="https://docs.google.com/spreadsheets/d/e/2PACX-1vRzmIcecD3Q-bhAaPSu46EDgxb680ejwWB06Gr9OmabVUFR-GtkVm3PCvUoI6o4Fw0YBW1KTQQjarwn/pubchart?oid=341468645&format=image">

Both network latency and storage have a large impact on performance. The benchmarks above are running with Goflow on the same server as the backends.
