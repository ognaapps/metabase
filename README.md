# Metabase Docker Compose for Ogna Stack

A Docker Compose configuration for running Metabase (business intelligence and analytics platform) as part of the Ogna stack.

## About Metabase

Metabase is an open-source business intelligence tool that allows you to create dashboards, visualizations, and queries without SQL knowledge. It's perfect for data exploration and reporting.

## Prerequisites

- Docker (version 20.10 or higher)
- Docker Compose (version 2.0 or higher)
- At least 2GB of available RAM
- Database to connect to (PostgreSQL, MySQL, MongoDB, etc.)

## Quick Start

1. Clone this repository:

```bash
git clone https://github.com/ognaapps/metabase.git
cd metabase
```

2. Create environment file:

```bash
cp .env.example .env
# Edit .env with your configuration
```

3. Start Metabase:

```bash
docker-compose up -d
```

4. Access Metabase:
   - Open your browser and navigate to `http://localhost:3000`
   - Complete the initial setup wizard
   - Add your database connections

## Configuration

### Default Settings

- **Metabase Version**: Latest stable
- **Port**: 3000
- **Application Database**: H2 (embedded) or PostgreSQL (recommended for production)
- **Data Persistence**: Metabase data stored in Docker volumes

### Environment Variables

Edit your `.env` file with the following configurations:

```bash
# Basic Configuration
MB_PORT=3000
MB_JETTY_HOST=0.0.0.0

# Database Configuration (Metabase's own database)
# Option 1: H2 (Development/Testing)
MB_DB_TYPE=h2
MB_DB_FILE=/metabase-data/metabase.db

# Option 2: PostgreSQL (Production - Recommended)
MB_DB_TYPE=postgres
MB_DB_DBNAME=metabase
MB_DB_PORT=5432
MB_DB_USER=metabase
MB_DB_PASS=changeme
MB_DB_HOST=postgres

# MySQL Option (Alternative)
# MB_DB_TYPE=mysql
# MB_DB_DBNAME=metabase
# MB_DB_PORT=3306
# MB_DB_USER=metabase
# MB_DB_PASS=changeme
# MB_DB_HOST=mysql

# Java Options (Memory Configuration)
JAVA_OPTS=-Xmx2g -Xms512m

# Site Configuration
MB_SITE_NAME=Ogna Analytics
MB_SITE_LOCALE=en
MB_SITE_URL=http://localhost:3000

# Email Configuration (Optional)
MB_EMAIL_SMTP_HOST=smtp.gmail.com
MB_EMAIL_SMTP_PORT=587
MB_EMAIL_SMTP_USERNAME=your-email@example.com
MB_EMAIL_SMTP_PASSWORD=your-app-password
MB_EMAIL_FROM_ADDRESS=metabase@ogna.com

# Security
MB_ENCRYPTION_SECRET_KEY=your-secret-key-here
MB_PASSWORD_COMPLEXITY=strong
MB_PASSWORD_LENGTH=8

# Advanced
MB_EMOJI_IN_LOGS=false
MB_COLORIZE_LOGS=false
```

## Accessing Metabase

### Local Development

```
http://localhost:3000
```

### Production (with domain)

```
https://analytics.your-domain.com
```

## Initial Setup

### First Time Configuration

1. **Create Admin Account**

   - Navigate to `http://localhost:3000`
   - Fill in admin details
   - Set organization name

2. **Add Database Connection**

   - Click "Add Database"
   - Select database type (MongoDB, PostgreSQL, MySQL, etc.)
   - Enter connection details

3. **Example MongoDB Connection** (Ogna Stack)
   - Name: `Ogna MongoDB`
   - Host: `mongo` (or `host.docker.internal` if on same machine)
   - Port: `27017`
   - Database: `your_database_name`
   - Username: `admin`
   - Password: `your_password`
   - Authentication Database: `admin`

## Integration with Ogna Stack

Metabase can connect to various Ogna stack components:

### MongoDB Connection

```
Host: mongo
Port: 27017
Database: ogna_db
Authentication: Yes
```

### PostgreSQL Connection

```
Host: postgres
Port: 5432
Database: ogna_db
```

### MySQL Connection

```
Host: mysql
Port: 3306
Database: ogna_db
```

## Common Use Cases

### Creating Dashboards

1. Navigate to "New" → "Dashboard"
2. Add questions/queries
3. Arrange visualizations
4. Share with team

### SQL Queries

1. Click "New" → "Question"
2. Select "Native Query"
3. Write SQL:

```sql
SELECT
  DATE(created_at) as date,
  COUNT(*) as count
FROM users
WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY DATE(created_at)
ORDER BY date;
```

### Automated Reports

1. Create a dashboard
2. Click "Sharing" → "Email it"
3. Set schedule (daily, weekly, monthly)
4. Add recipients

## Management

### Start Services

```bash
docker-compose up -d
```

### Stop Services

```bash
docker-compose down
```

### View Logs

```bash
docker-compose logs -f metabase
```

### Restart Service

```bash
docker-compose restart metabase
```

### Update Metabase

```bash
docker-compose pull metabase
docker-compose up -d
```

## Data Persistence

Data is persisted in Docker volumes:

- `metabase_data`: Application database (H2 file or PostgreSQL data)
- Questions, dashboards, and configurations are stored here

### Backup Metabase Data

#### H2 Database Backup

```bash
# Stop Metabase
docker-compose stop metabase

# Backup the data directory
docker run --rm -v metabase_metabase_data:/data -v $(pwd)/backup:/backup \
  alpine tar czf /backup/metabase-backup-$(date +%Y%m%d).tar.gz -C /data .

# Start Metabase
docker-compose start metabase
```

#### PostgreSQL Database Backup

```bash
docker-compose exec postgres pg_dump -U metabase metabase > metabase-backup-$(date +%Y%m%d).sql
```

### Restore Metabase Data

#### H2 Database Restore

```bash
# Stop Metabase
docker-compose stop metabase

# Restore
docker run --rm -v metabase_metabase_data:/data -v $(pwd)/backup:/backup \
  alpine sh -c "cd /data && tar xzf /backup/metabase-backup-YYYYMMDD.tar.gz"

# Start Metabase
docker-compose start metabase
```

#### PostgreSQL Database Restore

```bash
docker-compose exec -T postgres psql -U metabase metabase < metabase-backup-YYYYMMDD.sql
```

## Security Best Practices

⚠️ **Critical Security Steps**:

1. **Change Default Passwords**

   - Set strong admin password during setup
   - Update database passwords in `.env`

2. **Use HTTPS in Production**

   - Configure SSL/TLS via reverse proxy
   - Set `MB_SITE_URL` to https URL

3. **Secure Database Connections**

   - Use SSL for database connections
   - Restrict database user permissions (read-only recommended)
   - Use separate credentials for Metabase

4. **Set Encryption Key**

   - Generate strong key: `openssl rand -base64 32`
   - Set `MB_ENCRYPTION_SECRET_KEY`
   - Keep this key secure and backed up!

5. **Configure Authentication**

   - Enable SSO/LDAP for enterprise use
   - Set password complexity requirements
   - Enable session timeout

6. **Network Security**
   - Use Docker networks to isolate services
   - Implement firewall rules
   - Use reverse proxy (nginx/Traefik)

## Reverse Proxy Setup (nginx)

Example nginx configuration:

```nginx
server {
    listen 80;
    server_name analytics.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name analytics.your-domain.com;

    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Troubleshooting

### Metabase Won't Start

- Check logs: `docker-compose logs metabase`
- Verify port 3000 is available: `lsof -i :3000`
- Check memory allocation (increase if needed)
- Verify Java heap size in `JAVA_OPTS`

### Database Connection Failed

- Verify database is running
- Check connection credentials
- Ensure database is accessible from Metabase container
- Test connection from container:
  ```bash
  docker-compose exec metabase ping mongo
  ```

### Out of Memory Errors

- Increase Java heap size in `.env`:
  ```bash
  JAVA_OPTS=-Xmx4g -Xms1g
  ```
- Increase Docker memory limits

### Slow Query Performance

- Add database indexes
- Optimize SQL queries
- Enable query caching in Metabase
- Consider read replicas for large databases

### Can't Access After Restart

- Check if port mapping changed
- Verify `MB_SITE_URL` matches access URL
- Clear browser cache
- Check firewall rules

### Migration Issues

- When upgrading, backup data first
- Check Metabase release notes for breaking changes
- Verify database compatibility

## Performance Optimization

### Database Connection Pooling

```bash
MB_DB_CONNECTION_TIMEOUT_MS=10000
MB_JDBC_DATA_WAREHOUSE_MAX_CONNECTION_POOL_SIZE=15
```

### Query Caching

- Enable in Admin → Settings → Caching
- Set cache TTL based on data freshness needs
- Use dashboard filters for dynamic filtering

### Memory Tuning

```bash
# For larger deployments
JAVA_OPTS=-Xmx6g -Xms2g -XX:+UseG1GC
```

## Useful Features

- **SQL Editor**: Write custom queries
- **Query Builder**: Visual query interface
- **Dashboards**: Interactive visualizations
- **Alerts**: Get notified on data changes
- **Embedding**: Embed charts in other applications
- **API Access**: Programmatic access to data
- **Collections**: Organize questions and dashboards
- **Permissions**: Granular access control

## Best Practices

1. **Use Read-Only Database Users**

   - Prevents accidental data modification
   - Enhances security

2. **Organize with Collections**

   - Group related dashboards
   - Set collection permissions

3. **Document Questions**

   - Add descriptions to queries
   - Use meaningful names

4. **Schedule Regular Backups**

   - Automate backup process
   - Test restore procedures

5. **Monitor Performance**
   - Track slow queries
   - Optimize database indexes
   - Use query caching

## Resources

- [Metabase Documentation](https://www.metabase.com/docs/latest/)
- [Metabase Community Forum](https://discourse.metabase.com/)
- [SQL Tutorial](https://www.metabase.com/learn/sql-questions/)
- [Dashboard Best Practices](https://www.metabase.com/learn/dashboards/)

## License

[Specify your license]

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## Support

For issues and questions:

- Open an issue on [GitHub](https://github.com/ognaapps/metabase/issues)
- Contact the Ogna team
- Check [Metabase documentation](https://www.metabase.com/docs/)

---

Made with ❤️ by the Ogna team
