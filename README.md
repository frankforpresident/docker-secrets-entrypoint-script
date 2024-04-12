# Docker Secrets Loader

This script is designed to address the issue of securely loading sensitive information, such as passwords, API keys, or certificates, into Docker containers as environment variables using Docker secrets.

## Why this script?

Docker provides a mechanism called Docker secrets to manage sensitive data securely. However, using Docker secrets directly in environment variables can be challenging. This script simplifies the process of loading Docker secrets into environment variables and also supports populating variables from files.

## How it works

1. **Expanding Variables from Docker Secrets**: The script searches for environment variables containing a specific prefix (`DOCKER-SECRET->`) that indicates the presence of a Docker secret. It then retrieves the secret value from the specified location (`ENV_SECRETS_DIR`) and replaces the variable with its value.

2. **Populating Variables from Files**: Additionally, the script supports populating variables from files. It looks for environment variables ending with `_FILE` suffix, indicating that the value should be read from a file. It reads the contents of the file and assigns it to the corresponding variable.

3. **Execution**: Finally, the script executes the command provided as arguments, ensuring that the environment variables are properly populated with Docker secrets and file contents.

## Usage Examples

### Using Docker Secrets

Suppose you have a Docker secret named `db_password` containing the password for your database. You can use it in your Docker container by setting an environment variable as follows:

```bash
export DB_PASSWORD=DOCKER-SECRET->db_password
```

### Populating Variables from Files

You might have a file containing the SSL certificate for your application named `ssl_cert.pem`. You can populate an environment variable with its contents like this:

```bash
export SSL_CERT_FILE=/path/to/ssl_cert.pem
```

### Running the Script

Assuming the script is named `docker-entrypoint.sh`, you can use it in your Docker container's entry point like this:

```Dockerfile
ENTRYPOINT ["./docker-entrypoint.sh", "your_command_here"]
```

Replace `"your_command_here"` with the command you want to execute inside the container.

### Docker Compose Example
```yaml
version: '3.8'
services:
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
      # using the _FILE suffix to indicate that the value is a file path
      - POSTGRES_USER_FILE=/run/secrets/db_username
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - db_data:/var/lib/postgresql/data
    secrets:
      - db_username
      - db_password
    restart: always

secrets:
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
 
This script is licensed under the MIT License. See the [LICENSE](LICENSE) file for more information.

##### Acknowledgement

This script is inspired by and based on the work of the original repository DevilaN/docker-entrypoint-example. I extend our gratitude to the contributors of that repository for providing a solid foundation for managing Docker entry points.