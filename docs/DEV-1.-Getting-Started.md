# Overview

The FMTM codebase consists of:

- An API backend in FastAPI (code in: `src/backend`)
- A frontend website (soon to be a PWA) in react (code in: `src/frontend`)

![FMTM Dataflow Diagram](https://github.com/hotosm/fmtm/blob/main/docs/dataflow.dia)

# To Start working on this project

## Get project code

1. Clone the project `git clone https://github.com/hotosm/fmtm.git`

## Setup your local environment

### Setup OSM OAUTH 2.0

The FMTM uses OAUTH2 with OSM to authenticate users. To properly configure your FMTM project, you will need to create keys for OSM.

1. Login to OSM, Click on My Settings and register your local fmtm backend app to Oauth2applications

2. Put your login redirect url as `http://127.0.0.1:8000/auth/callback/`, For Production replace the URL as production API Url

<img width="716" alt="image" src="https://user-images.githubusercontent.com/36752999/216319298-1444a62f-ba6b-4439-bb4f-2075fdf03291.png">

3. Right now read user preferences permission is enough later on fmtm may need permission to modify the map option which should be updated on OSM_SCOPE variable on .env , Keep read_prefs for now

4. Now Copy your Client ID , Client Secret and put it to `.env`

### Create an `.env` File

Environmental variables are used throughout this project. To get started, create `.env` file in the top level dir, Sample is `.env.example`

    cp .env.example .env

Your env should look like this

    ODK_CENTRAL_URL=https://central-proxy
    ODK_CENTRAL_USER=`<any_valid_email_address>`
    ODK_CENTRAL_PASSWD=`<password_of_central_user>`
    API_URL=http://127.0.0.1:8000
    FRONTEND_MAIN_URL=http://localhost:8080
    FRONTEND_MAP_URL=http://localhost:8081
    OSM_CLIENT_ID=`<OSM_CLIENT_ID_FROM_ABOVE>`
    OSM_CLIENT_SECRET=`<OSM_CLIENT_SECRET_FROM_ABOVE>`
    OSM_URL=https://www.openstreetmap.org
    OSM_SCOPE=read_prefs
    OSM_LOGIN_REDIRECT_URI=http://127.0.0.1:8000/auth/callback/
    OSM_SECRET_KEY=`<random_key_for_development>`

## Verify Setup

### Check Deployment

For details on how to run this project locally for development, please look at: [DEV 2. Backend](https://github.com/hotosm/fmtm/wiki/DEV-2.-Backend)

### Check Authentication

Once you have deployed, you will need to check that you can properly authenticate.

1. Navigate to `API_URL/docs`

    Three endpoints are responsible for oauth
    <img width="698" alt="image" src="https://user-images.githubusercontent.com/36752999/216319601-949c4262-782f-4da4-ae26-dac81c141403.png">

2. Hit `/auth/osm_login/` : This will give you the Login URL where you can supply your osm username/password

    Response should be like this :

        {"login_url": "https://www.openstreetmap.org/oauth2/authorize/?response_type=code&client_id=xxxx"}

    Now Copy your login_url and hit it in new tab, and you will be redirected to OSM for your LOGIN. Give FMTM the necessary permission

    After successful login, you will get your `access_token` for FMTM Copy it and now you can use it for rest of the endpoints that need authorizations

3. Check your access token: Hit `/auth/me/` and pass your `access_token` You should get your osm id, username and profile picture id

# Start Developing

Don't forget to review [Contribution](https://github.com/hotosm/fmtm/wiki/Contribution) guidelines and our [Code of Conduct](https://github.com/hotosm/fmtm/wiki/Code-of-Conduct) before contributing!

## Backend Tips

### Implement authorization on an endpoints

To add authentication to an endpoint, import `login_required` from `auth` module, you can use it as a decorator or use fastapi `Depends(login_required)` on endpoints.

## Backend Debugging

1. Uncomment in docker-compose.yml:

```yaml
services:
  api:
    target: debug
    ports:
      - "5678:5678"
```

2. Re-build the docker image `docker compose build api`
3. Start the docker container `docker compose up -d api` (the api startup will be halted until you connect a debugger)
4. Set a debugger config in your IDE (e.g. VSCode) and start the debugger
5. The API server will start up & any set breakpoints will trigger

Example launch.json config for vscode:

```
{
  "configurations": [
    {
      "name": "Remote - Server Debug",
      "type": "python",
      "request": "attach",
      "host": "localhost",
      "port": 5678,
      "pathMappings": [
        {
          "localRoot": "${workspaceFolder}/src/backend",
          "remoteRoot": "/app/backend"
        }
      ],
      "justMyCode": false
    }
  ]
}
```

> Note: Note: either port 5678 needs to be bound to your localhost, or the `host` parameter can be set to the container IP address.
