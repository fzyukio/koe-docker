## Setup the first time

### Install docker and docker-compose
Docker can be any version, but docker-compose MUST be 1.20.1

#### Install Docker
Docker can be installed on Windows, Mac and many variety of Linux.
However, at the moment of writing this document, Mac with Apple Silicon (M1) has not been
supported. However as Docker images are platform independent, when Docker does support
Apple Silicon, feel free to try this instruction on that architecture. Currently, the
only way to run Koe on a Mac M1 is to run it as a python project (See https://github.com/fzyukio/koe for details)

To install Docker, follow the instruction here: https://docs.docker.com/engine/install/

#### Install docker-compose:
docker-compose 1.20.1 is available to download here: https://github.com/docker/compose/releases/tag/1.20.1
You MUST install docker-compose after Docker, otherwise, Docker will overwrite your version of docker-compose

### Setup docker network
Koe attaches itself to a bridge network, meaning that if you have multiple websites hosting on the same machine, you can
host koe together with them as well, given that all other websites are also docker.

Typically a network should be configured to run at startup and left alone.
If you need to stop Koe or deploy a new version of Koe, you don't shutdown the network.

If you don't have a bridge network already, you can run one as following:
```bash
# Navigate to nginx-server
cd nginx-server

# Start the network
docker-compose up -d
```

The default bridge network will serve at port 9080 for HTTP and 9443 for HTTPS. To change these ports, modify
`nginx-server/docker-compose.yml` as following:

```yml
ports:
  # Change 9080 and 9443 to any publicly available port you have
  - "9080:80"
  - "9443:443"
```

The network's name is `nginxserver_web`. If you alrady have a bridge network with a different name, you need to change koe's settings accordingly. Next section will talk more about that.

### Change Koe Docker's settings

Make a copy of .env.default:
```
cp .env.default .env
```

You might want to change:
- `DOMAIN=localhost` to something else, localhost or a public domain
- `EXTERNAL_NETWORK=nginxserver_web` if you have an existing bridge network with a different name, put its name here

### Change Koe's Django settings

Make a copy of `settings/settings.default.yaml`:
```
cp settings/settings.default.yaml settings/settings.yaml
```

You might want to change:
```yaml
allowed_hosts:
  # If you wish to make koe available publicly (to the internet, or to your local LAN network)
  # you need to add the domain name here, for example, koe.io.ac.nz, or MACHINE-1234
  - 'localhost'
  - '0.0.0.0'
  - '127.0.0.1'

csrf_trusted_origin:
  # Do the same as with allowed_hosts

secret_key: # Generate your own secret key here

# Change it to the public URL of your Koe image
site_url: 'http://localhost:9080/'

# Change it to your own STMP server
email_config: 'smtp.your-domain.com:your-name:your-password:465'

# Change it to your admin's email
from_email: 'your-name@your-domain.com'
contact_emails:
    - 'your-name@your-domain.com'

```

### Run Koe

#### IMPORTANT: Run KOE the first time
> IMPORTANT: When running KOE the first time, you must turn on a flag inside docker-compose.yml to create a new database:
> Find the following snippet:

```yaml
#     Uncomment below to DELETE and re-create the database from fixtures
# - RESET_DB=true
```

Remove the two characters `# ` (hash and then space) from the second line

Now run Koe:
```
docker-compose up -d
```

##### Check that Koe is running
Use your favourite browser, navigate to localhost:9080 - if everything was done correctly, you should see the welcome screen

##### Reset Koe's admin password
Run the following command and follow the prompt to change the password of `superuser` - This is the admin of Koe

```bash
docker exec -it koe_docker_app python manage.py changepassword superuser
```

##### IMPORTANT: Disable database reset
> IMPORTANT: If you don't do this, everytime you run Koe your database will be reset, and you'll lose everything, including your admin password

First, shutdown Koe:
```
docker-compose down
```

Now, open docker-compose.yml and locate the line `- RESET_DB=true`, put `# ` at the beginning to comment it out

#### Run Koe from the second time onward:
Navigate to `koe-docker` directory and run:

```bash
docker-compose up -d
```
