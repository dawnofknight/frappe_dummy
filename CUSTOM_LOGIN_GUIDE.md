# Custom Login Guide for Frappe Docker

This guide documents the customizations made to the Frappe login page and provides troubleshooting information for common issues.

## Customizations Made

### 1. Custom Logo Addition
- Added a custom logo to the login page
- Positioned above the login form
- Applied responsive styling for different screen sizes

### 2. Removed "Built on Frappe" Text
- Added CSS to hide the web footer containing the "Built on Frappe" text
- Created a custom CSS file in the `custom_login` app
- Added inline CSS to ensure the footer is hidden

### 3. Pre-filled Administrator Username
- Added JavaScript to automatically fill the Administrator username in the login form

## File Locations

- **Custom Login Template**: `/workspace/development/frappe-bench/apps/custom_login/custom_login/www/login.html`
- **Custom CSS**: `/workspace/development/frappe-bench/apps/custom_login/custom_login/public/css/custom.css`

## Troubleshooting

### JavaScript Errors

If you encounter a `ReferenceError: frappe is not defined` error:
1. Ensure the login template extends the base template: `{% extends "templates/web.html" %}`
2. Check that JavaScript files are properly included in the template
3. Make sure the template includes the head block with necessary meta tags

### CSS Not Loading

If custom styles are not applying:
1. Verify the CSS file exists in the correct location
2. Check that the CSS file is properly linked in the HTML template
3. Try adding inline styles directly in the template as a fallback

### "Built on Frappe" Text Still Visible

If the "Built on Frappe" text is still visible:
1. Add the following CSS to hide the footer:
   ```css
   .web-footer { display: none !important; }
   ```
2. Clear your browser cache and reload the page
3. Check if the text appears in a different element that needs to be targeted

## Docker Commands Cheatsheet

### Basic Docker Commands

```bash
# Start all containers
docker compose up -d

# Stop all containers
docker compose down

# View running containers
docker compose ps

# View container logs
docker compose logs -f [service_name]
```

### Executing Commands in Containers

```bash
# Execute a command in the Frappe container
docker compose exec frappe [command]

# Open a bash shell in the Frappe container
docker compose exec frappe bash

# Run bench commands in the Frappe container
docker compose exec -w /workspace/development/frappe-bench frappe bench [command]
```

### Container Management

```bash
# Rebuild containers
docker compose build

# Restart a specific container
docker compose restart [service_name]

# View container resource usage
docker stats
```

## Bench Commands Cheatsheet

### Site Management

```bash
# Create a new site
bench new-site [site-name]

# Set site as default
bench use [site-name]

# List all sites
bench site

# Backup site
bench backup [site-name]

# Restore site
bench --site [site-name] restore [backup-file]
```

### App Management

```bash
# Install an app
bench --site [site-name] install-app [app-name]

# Uninstall an app
bench --site [site-name] uninstall-app [app-name]

# Update an app
bench update [app-name]

# List all apps
bench list-apps
```

### Development Commands

```bash
# Start development server
bench start

# Clear cache
bench --site [site-name] clear-cache

# Build assets
bench build

# Run tests
bench run-tests

# Create a new app
bench new-app [app-name]
```

### Maintenance Commands

```bash
# Update bench
bench update

# Migrate database
bench --site [site-name] migrate

# Reset admin password
bench --site [site-name] set-admin-password [new-password]

# Clear website cache
bench --site [site-name] clear-website-cache
```

## Important Notes

1. Always run Docker commands from the directory containing the `docker-compose.yml` file (typically `/Users/fajar/Documents/GitHub/frappe_docker/devcontainer-example`)
2. When running bench commands, make sure to specify the working directory with `-w /workspace/development/frappe-bench`
3. After making changes to templates or CSS, clear the cache with `bench clear-cache` to see the changes
4. For persistent customizations, create files in the custom app rather than modifying core Frappe files

## Nginx, Routing and App Layer

### Nginx Configuration

Frappe Docker uses Nginx as the web server to handle HTTP requests. The Nginx configuration is managed through the following components:

```
├── resources/
│   ├── nginx-entrypoint.sh
│   └── nginx-template.conf
```

- **nginx-template.conf**: The template configuration file that gets populated with environment variables
- **nginx-entrypoint.sh**: Script that runs when the Nginx container starts, setting up the configuration

### Routing Architecture

1. **Request Flow**:
   - Client request → Nginx → Gunicorn (Python WSGI server) → Frappe application

2. **URL Routing**:
   - `/app/*`: Routes to the Desk (Frappe's admin interface)
   - `/api/*`: Routes to API endpoints
   - `/*`: Routes to website pages (like the login page)

3. **Custom Routes**:
   - Custom routes can be defined in your app's `hooks.py` file:
     ```python
     website_route_rules = [
         {"from_route": "/custom", "to_route": "custom_page"},
     ]
     ```

### App Layer Structure

The Frappe application is organized in layers:

1. **Presentation Layer**:
   - **Templates**: Located in `apps/[app_name]/[app_name]/templates/`
   - **Web Pages**: Located in `apps/[app_name]/[app_name]/www/`
   - **Public Assets**: Located in `apps/[app_name]/[app_name]/public/`

2. **Application Layer**:
   - **Controllers**: Located in `apps/[app_name]/[app_name]/[module_name]/`
   - **API**: Located in `apps/[app_name]/[app_name]/api/`

3. **Data Layer**:
   - **DocTypes**: Defined through the Frappe UI or JSON files

### Customizing Routes and Templates

1. **Override Core Templates**:
   - Create the same file path in your custom app to override core templates
   - Example: To override `/frappe/www/login.html`, create `/custom_login/www/login.html`

2. **Static File Serving**:
   - Static files are served from `/assets/[app_name]/`
   - Example: `/custom_login/public/css/custom.css` is served at `/assets/custom_login/css/custom.css`

3. **Nginx Location Blocks**:
   - The Nginx configuration includes location blocks for different URL patterns
   - Static files are cached with appropriate headers
   - Dynamic requests are proxied to the Gunicorn server

### Debugging Routing Issues

1. **Check Nginx Logs**:
   ```bash
   docker compose logs -f nginx
   ```

2. **Test Static File Access**:
   ```bash
   curl -I http://localhost:8000/assets/custom_login/css/custom.css
   ```

3. **Clear Browser Cache**:
   - Use Ctrl+Shift+R or Cmd+Shift+R to force reload without cache

4. **Clear Server Cache**:
   ```bash
   docker compose exec -w /workspace/development/frappe-bench frappe bench clear-cache
   ```