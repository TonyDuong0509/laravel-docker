# Laravel Docker Environment

A complete Docker containerization setup for Laravel applications with PHP 8.3, MySQL 8.0, Redis, and Nginx.

## Architecture Overview

This setup provides a multi-container environment with the following services:

- **App Container**: PHP 8.3-FPM with Laravel application
- **Database**: MySQL 8.0 with phpMyAdmin
- **Cache**: Redis for caching and sessions
- **Web Server**: Nginx as reverse proxy
- **Queue Workers**: Supervisor-managed Laravel queue workers
- **Cron Jobs**: Automated Laravel task scheduling

## Services Configuration

### Application Container (`app`)
- **Base Image**: PHP 8.3-FPM
- **Extensions**: PDO, MySQL, GD, BCMath, Intl, Zip, Mbstring
- **Tools**: Composer, Node.js/NPM, Supervisor, Cron
- **Port**: 9000 (PHP-FPM)

### Database Container (`database`)
- **Image**: MySQL 8.0
- **Database**: `tasklist`
- **User**: `tasklist` / Password: `root`
- **Port**: 6101 (external) → 3306 (internal)

### Web Server Container (`webserver`)
- **Image**: Nginx Alpine
- **Port**: 6100 (external) → 80 (internal)
- **Configuration**: Custom Nginx config with PHP-FPM integration

### Cache Container (`redis`)
- **Image**: Redis Alpine
- **Port**: 6102 (external) → 6379 (internal)

### Database Management (`phpmyadmin`)
- **Image**: phpMyAdmin
- **Port**: 6103 (external) → 80 (internal)

## Quick Start

### Prerequisites
- Docker
- Docker Compose

### Setup & Installation

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd laravel-docker
   ```

2. **Place your Laravel project**
   ```bash
   # Place your Laravel application in the ./src directory
   # or update the volume mapping in docker-compose.yml
   ```

3. **Start the environment**
   ```bash
   docker-compose up -d
   ```

4. **Access your application**
   - **Laravel App**: http://localhost:6100
   - **phpMyAdmin**: http://localhost:6103
   - **MySQL**: localhost:6101
   - **Redis**: localhost:6102

## Container Features

### Automated Setup ([.docker/start_app.sh](.docker/start_app.sh))
The application container automatically:
- Installs Composer dependencies
- Generates application key
- Creates storage symlink
- Runs database migrations
- Clears and optimizes caches
- Installs NPM dependencies and builds assets
- Starts queue workers and cron jobs

### Process Management ([.docker/supervisord.conf](.docker/supervisord.conf))
Supervisor manages:
- **Laravel Queue Workers**: Processes background jobs
- **Cron Daemon**: Handles scheduled tasks
- **PHP-FPM**: Serves PHP requests

### Scheduled Tasks ([.docker/cronjob](.docker/cronjob))
- Runs Laravel's task scheduler every minute
- Logs output to `/var/log/cron.log`

### Web Server Configuration ([.docker/nginx.conf](.docker/nginx.conf))
- Optimized for Laravel applications
- PHP-FPM integration with extended timeouts
- Static file serving
- URL rewriting for clean URLs

## File Structure

```
.
├── .docker/
│   ├── Dockerfile          # PHP application container
│   ├── nginx.conf          # Nginx configuration
│   ├── supervisord.conf    # Process management
│   ├── start_app.sh        # Container startup script
│   └── cronjob            # Cron schedule
├── .dockerignore          # Docker build exclusions
├── docker-compose.yml     # Multi-container orchestration
└── src/                   # Laravel application code
```

## Development Workflow

### Accessing Containers
```bash
# Access application container
docker exec -it tasklist_backend bash

# Access database container
docker exec -it tasklist_db mysql -u tasklist -p

# Access Redis
docker exec -it tasklist_redis redis-cli
```

### Laravel Artisan Commands
```bash
# Run migrations
docker exec tasklist_backend php artisan migrate

# Clear caches
docker exec tasklist_backend php artisan cache:clear

# Queue workers
docker exec tasklist_backend php artisan queue:work
```

### Logs
```bash
# Application logs
docker logs tasklist_backend

# Nginx logs
docker logs tasklist_webserver

# Database logs
docker logs tasklist_db
```

## Customization

### Environment Variables
Create a `.env` file in your Laravel application with database configuration:
```env
DB_CONNECTION=mysql
DB_HOST=database
DB_PORT=3306
DB_DATABASE=tasklist
DB_USERNAME=tasklist
DB_PASSWORD=root

CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```

### Port Configuration
Modify ports in [docker-compose.yml](docker-compose.yml):
- Laravel App: Change `6100:80`
- MySQL: Change `6101:3306`
- Redis: Change `6102:6379`
- phpMyAdmin: Change `6103:80`

## Troubleshooting

### Common Issues
1. **Permission Issues**: Ensure proper file permissions for Laravel storage directories
2. **Database Connection**: Verify database service is running and environment variables are correct
3. **Queue Jobs**: Check supervisor logs if background jobs aren't processing

### Useful Commands
```bash
# Rebuild containers
docker-compose down && docker-compose up --build -d

# View all logs
docker-compose logs -f

# Reset database
docker-compose down -v && docker-compose up -d
```

## Production Considerations

- Update default passwords and credentials
- Configure proper SSL/TLS certificates
- Set up proper backup strategies for database volumes
- Review and harden security configurations
- Monitor resource usage and scale accordingly
