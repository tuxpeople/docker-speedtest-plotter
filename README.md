# speedtest-plotter
![Github Workflow Badge](https://github.com/tuxpeople/docker-speedtest-plotter/actions/workflows/release.yml/badge.svg)
![Github Last Commit Badge](https://img.shields.io/github/last-commit/tuxpeople/docker-speedtest-plotter)
![Docker Pull Badge](https://img.shields.io/docker/pulls/tdeutsch/speedtest-plotter)
![Docker Stars Badge](https://img.shields.io/docker/stars/tdeutsch/speedtest-plotter)
![Docker Size Badge](https://img.shields.io/docker/image-size/tdeutsch/speedtest-plotter)

## Quick reference

Originally from Anton Semjonov, I made it multi-arch for me.

This is a collection of scripts, which takes internet speedtest measurements
with [speedtest-cli](https://github.com/sivel/speedtest-cli) and plot them
with [gnuplot](http://gnuplot.sourceforge.net). A crontab schedule can be used
to automate measurements every couple of minutes and save them to a database.
The results can optionally be displayed through a simple Flask webserver.

![example plot of speedtest results](example.png)

* **Originally from:**
  https://github.com/ansemjo/speedtest-plotter
* **Code repository:**
  https://github.com/tuxpeople/docker-speedtest-plotter
* **Where to file issues:**
  https://github.com/tuxpeople/docker-speedtest-plotter/issues
* **Supported architectures:**
  ```amd64```, ```armv7```, ```armv6```, ```ppc64le``` and ```arm64```

## Image tags
- ```latest``` gets automatically built on every push to master and also via a weekly cron job

## USAGE

### CONTAINER
To start the container with default settings run:

    docker run -d -p 8000:8000 tdeutsch/speedtest-plotter

This will take a measurement every 15 minutes, save them to a SQLite database
in `/data/speedtests.db` and run the webserver on port `8000`. Visit http://localhost:8000
to look at the plotted results. (*Note: The smoothed bezier curves require at least two
measurements and the image will stay blank otherwise. So you might have to wait a while first.*)

For data persistence, either mount a volume at `/data` to save the database file
or set the environment variable `DATABASE` to an SQLAlchemy-compatible URI. A PostgreSQL
URI might look like this:

    docker run -d \
      -p 8000:8000 \
      -e DATABASE=postgresql://user:password@hostname:5432/database' \
      tdeutsch/speedtest-plotter

You can modify the measurement schedule with the environment variables `MINUTES` and
`SCHEDULE`. The former takes a measurement every `n` minutes and the latter may define
an entirely custom cron schedule like "four times a day":

    docker run -d -p 8000:8000 -e SCHEDULE="0 3,9,15,21 * * *" tdeutsch/speedtest-plotter

The webserver is a single-threaded Flask application, which may not be suitable
for production usage. To disable the webserver completely set the `PORT` environment
variable to an empty string. This will only take measurements and save them to the
database.

    docker run -d -e PORT= -v speedtests:/data tdeutsch/speedtest-plotter

To dump the results as CSV from a running container use the `dump` command:

    docker exec $containerid speedtest-plotter dump > results.csv

To reimport a previous dump in a fresh container use `import`:

    docker exec $containerid speedtest-plotter import < results.csv

This can also be used to import results obtained manually with `speedtest-cli`.

### PYTHON SCRIPT

You can use the Python script by itself locally, too. First install the requirements:

    pip install -r requirements.txt

Choose a database location and take any number of measurements:

    ./speedtest-plotter -d sqlite:///$PWD/measurements.db measure
    ...

Then start the flask webserver to look at the results:

    ./speedtest-plotter -d sqlite:///$PWD/measurements.db serve

### GNUPLOT SCRIPT

To keep things really simple, you can also take measurements manually with `speedtest-cli` and only
plot an image with `gnuplot`.

The [`plotscript`](plotscript) expects the format that `speedtest-cli` outputs when using the `--csv` flag
and a header line from `--csv-header`. To take some measurements manually with a simple sleep-loop:

    speedtest-cli --csv-header > results.csv
    while true; do speedtest-cli --csv | tee -a results.csv; sleep 600; done
    ^C

Afterwards plot the results to a PNG picture with:

    gnuplot -c plotscript results.csv plot.png

## LICENSE

Copyright (c) 2019 Anton Semjonov
Licensed under the MIT License
