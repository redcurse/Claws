# Claws
The server-side scraper software for ApolloTV.

## Getting Started

### Installation
Install node^10.14.0 and npm^6.4.1.
- Clone the repository and navigate to the directory.
- Run `npm install`.
- Copy the contents of `.env.dist` to a file called `.env`.
- Fill in the values for `SECRET_SERVER_ID` and `SECRET_CLIENT_ID`.
    - You can use `./generate-env-values.sh` to help.

### Security
In order to authenticate with the server, the client must make a login
request with the hashed (using the `bcrypt` library) `SECRET_CLIENT_ID` and the
current time. It will look like this before it's hashed:

`${current time in seconds}|${SECRET_CLIENT_ID}`

The server then checks the resulting hash with it's own version starting the
second the request arrives to the server. It will check up to 5 seconds back in
time just in case the client has a slow connection.

If the hash is valid within the time frame of 5 seconds, it is authorized and
the server sends a token down to the client that will last 1 hour. After the
hour is up, the client will request another token.

**Calling the authentication API:**

*Logging in*

- URL: `/api/v1/login`
- Method: `POST`
- Body:
```json
{
  "clientId": "{...}"
}
```

*Checking authentication status*
- URL: `/api/v1/authenticated`
- Method: `GET` or `POST`
- Parameters:
    - `token`: JWT token to validate.
- Body (`POST`):
```json
{
  "token": "{...}"
}
```

### Testing the server

### Running in Development Mode
- Set the `SECRET_CLIENT_ID` inside `public/index.html` to match the same value inside `.env`
- Run `npm run dev`
- Open your browser to `http://127.0.0.1:3000`
- To debug: open Chrome and navigate to [chrome://inspect](chrome://inspect) and click `Open dedicated DevTools for Node`. Or install [Node.js V8 --inspector Manager (NiM)](https://chrome.google.com/webstore/detail/nodejs-v8-inspector-manag/gnhhdgbaldcilmgcpfddgdbkhjohddkj)

### Running in Production Mode
- Run `npm start`
- Set up an nginx `proxy_pass` to `http://127.0.0.1:3000`.


### API

#### Movies
- Search for the exact name of a movie, like `The Avengers`.
- Open the developer console and watch the links arrive.

**Calling the Movie API:**
- Endpoint: `/api/v1/search/movies`
- Method: `GET`
- Parameters
    - `title`: movie title (exact name) <br>
    - `token`: valid JWT token


#### TV
- Search for a TV show by filling in name, season, episode. Eg: `Suits 4 1` (in the respective text boxes)
- Open the developer console and watch the links arrive.

**Calling the TV API:**
- Endpoint: `/api/v1/search/tv`
- Method: `GET`
- Parameters
    - `show`: name of show
    - `season`: season
    - `episode`: episode
    - `token`: valid JWT token


#### Event Structure
```javascript
{
    // The event type
    "event": "result",

    // The link to the source.
    "file": {
        "data": "",
        // Mime type or `file` for a base64 encoded m3u8 file
        "kind": "",
    },

    // Pairing (but a last resort)
    "pairing": {
        url: '',
        videoId: ''
        target: ''
    },

    // Metadata
    "metadata": {
        // The link quality
        "quality": "",

        // The provider's name (human readable)
        "provider": "",

        // The source that uploaded the content to the provider (human readable)
        "source": ""
    },

    // Required headers (last resort - may not even be possible)
    "headers": {
        "Referrer": ""
    }
}
```

```javascript
{
    // The event type
    "event": "scrape",

    // The provider URL that Claws needs the HTML from
    "target": "",

    // The headers required to receive a valid response from the provider
    "headers": {},

    // The resolver to send the HTML to
    "resolver": "",
}
```

## Running with Docker
### Build container
Go to the directory that has the Dockerfile and run the following command to build the Docker image. The -t flag lets you tag your image so it's easier to find later using the docker images command
```docker build -t apollotv-server .```
Your image will now be listed by Docker

### Run container
Running your image with -d runs the container in detached mode, leaving the container running in the background. The -p flag redirects a public port to a private port inside the container. Run the image you previously built The --env-file flag points to an env file containing environment variables you want your app to be aware of
```docker run -p 80:3000 --env-file ./.env apollotv-server```

### Other commands

#### List containers
```docker container ls --all```

#### List images
```docker image ls --all```

#### Print app output
```docker logs <container id>```

#### Stop all containers
```docker stop $(docker ps -a -q)```

#### Delete all containers
```docker rm $(docker ps -a -q)```

#### Delete all images
```docker rmi -f $(docker images -q)```

### PM2 commands

#### Monitoring CPU/Usage of each process
```docker exec -it <container-id> pm2 monit```

#### Listing managed processes
```docker exec -it <container-id> pm2 list```

#### Get more information about a process
```docker exec -it <container-id> pm2 show```

#### 0sec downtime reload all applications
```docker exec -it <container-id> pm2 reload all```
