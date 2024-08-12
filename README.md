# Deploying a Docker Container with a Laravel Project

This project provides a Docker environment to deploy a Laravel application using the backend from [Open Admin](https://github.com/open-admin-org/open-admin).

## Prerequisites

Before you begin, make sure you have the following installed on your machine:

- **Docker**: [Installation Instructions](https://docs.docker.com/get-docker/)
- **Docker Compose**: [Installation Instructions](https://docs.docker.com/compose/install/)

## Deployment Steps

### 1. Clone the Repository

Clone the Open Admin repository:

```bash
git clone https://github.com/open-admin-org/open-admin.git
```

Navigate to the project directory:

```bash
cd open-admin
```

### 2. Create a `.env` File

Copy the `.env.example` file and rename it to `.env`:

```bash
cp .env.example .env
```

Then, edit the `.env` file with your preferred configurations. Make sure to adjust database connection variables and other settings as needed.

### 3. Create a Dockerfile

Create a `Dockerfile` in the root of the project with the following content:

```Dockerfile
# Use an official PHP image with necessary extensions for Laravel
FROM php:8.1-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl

# Clean up the cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install necessary PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set the working directory
WORKDIR /var/www

# Copy the current project to the working directory in the container
COPY . .

# Install Laravel dependencies
RUN composer install --no-dev --optimize-autoloader

# Copy the Laravel configuration file
COPY .env.example .env

# Generate the Laravel application key
RUN php artisan key:generate

# Set appropriate permissions
RUN chown -R www-data:www-data /var/www \
    && chmod -R 755 /var/www/storage

# Expose port 9000 and define the default command
EXPOSE 9000
CMD ["php-fpm"]
```

### 4. Configure `docker-compose.yml`

Create a `docker-compose.yml` file in the root of the project with the following content:

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: open-admin-laravel
    container_name: open-admin-laravel-app
    restart: unless-stopped
    working_dir: /var/www
    volumes:
      - .:/var/www
      - ./vendor:/var/www/vendor
      - ./storage:/var/www/storage
    networks:
      - app-network
    env_file:
      - .env

  db:
    image: mysql:8.0
    container_name: open-admin-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: your_mysql_root_password
      MYSQL_DATABASE: open_admin_db
      MYSQL_USER: open_admin_user
      MYSQL_PASSWORD: your_mysql_password
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db_data:
```

### 5. Build and Start the Containers

Build and start the containers using Docker Compose:

```bash
docker-compose up --build -d
```

### 6. Migrate the Database

Run the database migrations inside the application container:

```bash
docker-compose exec app php artisan migrate
```

### 7. Access the Application

Once the containers are up and running, you can access the application in your web browser at `http://localhost`.

## Useful Commands

- **Stop the containers**:

  ```bash
  docker-compose down
  ```
- **View the container logs**:

  ```bash
  docker-compose logs -f
  ```
- **Restart the containers**:

  ```bash
  docker-compose restart
  ```

## Troubleshooting

### File Permissions

If you encounter issues related to file permissions, make sure that the `storage` and `bootstrap/cache` directories have the correct permissions:

```bash
chmod -R 775 storage bootstrap/cache
```

### Environment Variables

If changes are made to the `.env` file, make sure to rebuild the container:

```bash
docker-compose up --build -d
```

## Contributions

If you would like to contribute to this project, please open an issue or submit a pull request.

## License

This project is licensed under the [MIT License](LICENSE).

## Support Us

If you'd like to support this project, you can make a donation via PayPal. Your contribution is greatly appreciated!

<a href="https://www.paypal.com/donate/?hosted_button_id=8QR4FCJ8LJWGA" target="_blank">
    <img src="https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif" alt="Donate with PayPal" />
</a>


