# [TVCharts](https://tvcharts.ianray.com/)

TVCharts is a simple frontend / backend application with an API and multi-container deployment. It ingests raw data from [IMDb datasets](https://www.imdb.com/interfaces/) into a document store, then allows users to search and display results as charts of episode ratings over the entire run of a TV series.

![](https://raw.githubusercontent.com/ianthekid/tvcharts/master/client/preview.webp)

## Tech Stack

* React / Redux
* NodeJS / Express
* MongoDB
* Nginx / letsencrypt
* Docker
* AWS CDN (S3 + CloudFront)

See a working example at [tvcharts.ianray.com](https://tvcharts.ianray.com/)

## Requirements

Make sure all dependencies have been installed before moving on:

* Docker
* Node
* [Yarn](https://yarnpkg.com/en/docs/install) (or you can use npm)

## Build and Deploy

Steps to follow after cloning this repository

#### 1. Build assets for frontend
```shell
$ cd client
$ cp .env.example .env
$ yarn install
$ yarn build
```

You can customize `.env` or use default key for testing.

#### 2. Build and deploy server

For the `server/.env` you will need to get an [OMDB_API](http://www.omdbapi.com/apikey.aspx) key to display images. It's a free, one-click signup. The application will still work without it, but all the images will be placeholders and it won't look as cool.

```shell
$ cd server
$ cp .env.example .env
$ docker-compose build
$ docker-compose up
```

If you used the defaul settings in `.env`, then you should see the site at: [http://localhost](http://localhost)

You can start testing while data is still loading in, but all of the episode data won't be available until `docker logs data` shows `"all done."`

**! Warning:** the `basics` and `episode` files from IMDb include over 10 million of lines of text. So the initial load might make a laptop fan kick into high gear for a few minutes. The mongoDB indexing after initial load can also take over 5 mins depending on your CPU.

If you can use a VPS like EC2, DigitalOcean, etc. then just choose an option with 3+ vCPUs and at least 2GB memory. I used 4CPUs/2GB and it completely loads up in about 5mins.

#### Optional: Generate SSL Cert

Also included in the `nginx` container is letsencrypt and a config file (`ssl.conf`) for using SSL. To enable this, you need to set the `.env` variables `BUILD=production` and `DOMAIN` (which can be anything **except** `localhost`)

Then run the bash script:

```shell
$ docker exec -it nginx ./certbot.sh
```

If you have letsencrypt certs already generated, copy them to `server/nginx/letsencrypt`. It will check this directory before generating new ones.

#### Optional: CDN - AWS S3 + Cloudfront

Use AWS S3 + Cloudfront to host the static frontend assets. You will need to have an AWS account that can generate access key + secret, an S3 Bucket, and a CloudFront distribution that maps to a Bucket folder. Once you have those created, plug them into your `.env` variables

```
...
AWS_KEY=
AWS_SECRET=
S3BUCKET=
S3DIR=
CLOUDFRONT=
```

Then run the bash script:

```shell
$ docker exec -it nginx ./aws.sh
```

To un-do this, you can change the `BUILD` .env variable to `development` and re-build/deploy using docker. Or simply re-deploy the `nginx` container to use the latest `frontend` build: `docker-compose up -d nginx`

## Updating Database

Since we rely on IMDb datasets as our source of information, we can just drop the entire database and re-create it every time.

```shell
## Re-load database from IMDb datasets
$ docker restart data
```

Alternatively, you can also use `docker-compose up -d data`

## Author

I built TVCharts as a way of displaying my technical abilities to companies looking to hire a frontend or full stack developer.

* [Ian Ray](https://ianray.com/)
* [LinkedIn](https://www.linkedin.com/in/ianrray/)

## Credits

 * An awesome Python version which inspired me was developed by [thepiratex](https://github.com/thepiratex) can be found [here](https://dataiszen.com/app/tvshow) with the full [code here](https://github.com/thepiratex/tvshow)
 * The original concepts come from [/r/dataisbeautiful](https://www.reddit.com/r/dataisbeautiful/), with original done by user [/u/Hbenne](https://www.reddit.com/user/Hbenne)