# Docker Secrets Loader
This script is designed to address the issue of securely loading sensitive information, such as passwords, API keys, or certificates, into Docker containers as environment variables using Docker secrets.

# Why Use This Script?
Docker provides a feature called Docker secrets for secure management of sensitive information. However, integrating Docker secrets directly into environment variables can present challenges. This script simplifies the process of loading Docker secrets into environment variables and extends support for importing variables from files.

It becomes particularly valuable when you need to seamlessly integrate sensitive data from Docker secrets into your container's environment variables. Notably, Docker secrets typically require modifications to application code to read secrets from the /run/secrets directory. However, with the Docker Secrets Loader, such adjustments are unnecessary. This tool effortlessly injects the secret into an environment variable, shielding sensitive information from exposure.

## How it works

1. **Populating Variables from Files**: The script supports populating variables from files. It looks for environment variables ending with `_FILE` suffix, indicating that the value should be read from a file. It reads the contents of the file and assigns it to the corresponding variable without the suffix. The suffix `_FILE` is a commonly practiced Docker secret convention for reading sensitive data from files.

2. **Expanding Variables from Docker Secrets**: Alternatively, the script searches for environment variables containing a specific prefix (`DOCKER-SECRET->`) that indicates the presence of a Docker secret. It then retrieves the secret value from the specified location (`ENV_SECRETS_DIR`) and replaces the variable with its value.

3. **Execution**: Finally, the script executes the command provided as arguments, ensuring that the environment variables are properly populated with Docker secrets and file contents. This allows you to use the script as an entry point in your Docker container. 

## Usage Examples

### Using Docker Secrets

Suppose you have a Docker secret named `db_password` containing the password for your database. You can use it in your Docker container by setting an environment variable as follows:

```bash
- DB_PASSWORD=DOCKER-SECRET->db_password
```

Note: The file should be stored in the directory specified by `ENV_SECRETS_DIR` (default: `/run/secrets`).

### Populating Variables from Files

You can populate an environment variable with suffix `_FILE`. By appending `_FILE` as suffix, the image will fetch the content of the file instead of relying on the environment variable. For example variable `DB_PASSWORD_FILE` its contents will be exported as `DB_PASSWORD`.

```bash
- DB_PASSWORD_FILE=/run/secrets/db_password
```

## Installation

### Adding the script to your own docker image

To use this script in your Docker container, you can copy it into your image using a `COPY` command in your Dockerfile:

```Dockerfile
COPY ./docker/entrypoint /
```

Assuming the script is named `docker-entrypoint.sh`, you can use it in your Docker container's entry point like this:

```Dockerfile
ENTRYPOINT ["./docker-entrypoint.sh", "previous_entrypoint_here"]
```

### Add the script to an existing image

If you want to use this script with an existing Docker image, you can mount the script as a volume and execute it as the entry point:

```bash
docker run -d -e API_KEY_FILE='DOCKER-SECRET->api_key' -v /path/to/docker-entrypoint.sh:/docker-entrypoint.sh --entrypoint "/docker-entrypoint.sh previous_entrypoint_here" existing/image:latest
```

Replace `"previous_entrypoint_here"` with the command that was previously set as the entry point in the Dockerfile.

#### retrieving entrypoint from existing image
```bash
docker inspect --format='{{json .Config.Entrypoint}}' <image_name_or_id>
```

### Docker Compose Example
```yaml
version: '3.8'
services:
  web:
    image: existing/image:latest
    entrypoint: /docker-entrypoint.sh previous_entrypoint_here
    volumes:
      - /path/to/docker-entrypoint.sh:/docker-entrypoint.sh
    environment:
      - API_KEY_FILE=DOCKER-SECRET->api_key
    secrets:
      - api_key
    depends_on:
      - app
    restart: always

  app:
    image: your/app:latest
    ports:
      - "5000:5000"
    environment:
      - ENV_SECRETS_DEBUG=true
      - DB_NAME=my_database
      # using the DOCKER-SECRET->secret_key syntax
      - DB_USERNAME_FILE=DOCKER-SECRET->db_username
      - DB_PASSWORD_FILE=DOCKER-SECRET->db_password
    secrets:
      - db_username
      - db_password
    depends_on:
      - db
    restart: always

  db:
    image: postgres:latest
    environment:
      - POSTGRES_DB=my_database
      # _FILE suffix supported natively by postgres image
      - POSTGRES_USER_FILE=/run/secrets/db_username
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - db_data:/var/lib/postgresql/data
    secrets:
      - db_username
      - db_password
    restart: always

secrets:
  api_key:
    file: ./api_key.txt
  db_username:
    external: true
  db_password:
    external: true

volumes:
  db_data:
```

## Notes

- Make sure to set the `ENV_SECRETS_DIR` environment variable to specify the directory where Docker secrets are stored. By default, it is set to `/run/secrets`.
- Enable debug mode by setting the `ENV_SECRETS_DEBUG` environment variable to any value to print debug messages.
- This script assumes that Docker secrets are stored in plain text files within the specified directory (`ENV_SECRETS_DIR`). Ensure proper permissions and encryption if additional security measures are required.

Feel free to modify and integrate this script into your Docker environment to manage secrets securely and efficiently.

## License
 
This script is licensed under the MIT License. See the [LICENSE](https://github.com/frankforpresident/docker-secrets-loader/blob/main/LICENCE) file for more information.

### Acknowledgement

This script is inspired by and based on the work of the original repository [DevilaN/docker-entrypoint-example](https://github.com/DevilaN/docker-entrypoint-example). I extend our gratitude to the contributors of that repository for providing a solid foundation for managing Docker entry points.