# Frappe/ERPNext Customization Guide for Cloud Containers

## Overview

This comprehensive guide covers how to customize Frappe/ERPNext applications running in Docker containers in cloud environments. Whether you need to modify the UI, create custom modules, or deploy custom applications, this guide provides step-by-step instructions for containerized cloud deployments.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Development Environment Setup](#development-environment-setup)
3. [Basic UI Customizations](#basic-ui-customizations)
4. [Custom Module Development](#custom-module-development)
5. [Custom App Development](#custom-app-development)
6. [Cloud Deployment Strategies](#cloud-deployment-strategies)
7. [Advanced Customizations](#advanced-customizations)
8. [Troubleshooting](#troubleshooting)
9. [Best Practices](#best-practices)

## Prerequisites

### System Requirements
- **Docker** (20.10+)
- **Docker Compose** v2
- **Git**
- **VS Code** (recommended for development)
- **4GB+ RAM** allocated to Docker
- **Cloud Platform Account** (AWS, GCP, Azure, DigitalOcean, etc.)

### Knowledge Requirements
- Basic understanding of Docker and containerization
- Python programming (for backend customizations)
- JavaScript/HTML/CSS (for frontend customizations)
- Frappe Framework basics

## Development Environment Setup

### 1. Clone and Setup Development Environment

```bash
# Clone the frappe_docker repository
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker

# Copy development container configuration
cp -R devcontainer-example .devcontainer

# Copy VS Code configuration for debugging
cp -R development/vscode-example development/.vscode

# Copy environment file
cp example.env .env
```

### 2. Configure Development Environment

Edit `.env` file for your development needs:

```bash
# Development-specific settings
FRAPPE_VERSION=version-15
ERPNEXT_VERSION=version-15
FRAPPE_SITE_NAME_HEADER=development.localhost
DB_PASSWORD=your_secure_password
```

### 3. Start Development Environment

#### Option A: Using VS Code Dev Containers (Recommended)

1. Install VS Code Dev Containers extension
2. Open the project in VS Code
3. Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac)
4. Select "Dev Containers: Reopen in Container"
5. Wait for the container to build and start

#### Option B: Using Docker Compose

```bash
# Start development environment
docker compose -f .devcontainer/docker-compose.yml up -d

# Access the development container
docker compose -f .devcontainer/docker-compose.yml exec frappe bash
```

### 4. Create Your First Development Site

```bash
# Inside the development container
cd /workspace/development

# Create a new bench
bench init --frappe-branch version-15 frappe-bench
cd frappe-bench

# Create a new site
bench new-site development.localhost --admin-password admin

# Set the site as default
bench use development.localhost

# Start the development server
bench start
```

## Basic UI Customizations

### 1. Custom CSS Styling

#### Method 1: Through Web Interface

1. Login to your site as Administrator
2. Go to **Setup > Customize > Custom CSS**
3. Add your custom CSS:

```css
/* Example: Change primary color */
:root {
    --primary-color: #2e7d32;
    --primary-light: #4caf50;
    --primary-dark: #1b5e20;
}

/* Custom button styling */
.btn-primary {
    background-color: var(--primary-color);
    border-color: var(--primary-color);
}

/* Custom header styling */
.navbar {
    background-color: var(--primary-dark) !important;
}
```

#### Method 2: Through File System (Development)

Create custom CSS files in your app:

```bash
# Inside your custom app directory
mkdir -p public/css
touch public/css/custom.css
```

Add CSS content and include it in your app's hooks:

```python
# In your app's hooks.py
app_include_css = [
    "assets/css/custom.css"
]
```

### 2. Custom JavaScript

#### Client-side Scripts

1. Go to **Setup > Customize > Client Script**
2. Create new client script:

```javascript
// Example: Auto-fill fields based on conditions
frappe.ui.form.on('Customer', {
    customer_type: function(frm) {
        if (frm.doc.customer_type === 'Company') {
            frm.set_value('customer_group', 'Commercial');
        }
    }
});

// Example: Custom button
frappe.ui.form.on('Sales Invoice', {
    refresh: function(frm) {
        frm.add_custom_button(__('Custom Action'), function() {
            frappe.msgprint('Custom button clicked!');
        });
    }
});
```

#### Server-side Scripts

1. Go to **Setup > Customize > Server Script**
2. Create server script for backend logic:

```python
# Example: Auto-calculate field on save
if doc.doctype == "Sales Invoice":
    total_qty = sum([item.qty for item in doc.items])
    doc.custom_total_quantity = total_qty
```

### 3. Custom Print Formats

1. Go to **Setup > Printing > Print Format**
2. Create new print format with custom HTML/CSS:

```html
<!-- Custom Invoice Print Format -->
<div class="custom-invoice">
    <div class="header">
        <h1>{{ doc.company }}</h1>
        <p>Invoice #{{ doc.name }}</p>
    </div>
    
    <div class="customer-details">
        <h3>Bill To:</h3>
        <p>{{ doc.customer_name }}</p>
        <p>{{ doc.customer_address }}</p>
    </div>
    
    <table class="items-table">
        <thead>
            <tr>
                <th>Item</th>
                <th>Qty</th>
                <th>Rate</th>
                <th>Amount</th>
            </tr>
        </thead>
        <tbody>
            {% for item in doc.items %}
            <tr>
                <td>{{ item.item_name }}</td>
                <td>{{ item.qty }}</td>
                <td>{{ item.rate }}</td>
                <td>{{ item.amount }}</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</div>

<style>
.custom-invoice {
    font-family: Arial, sans-serif;
}
.header {
    text-align: center;
    margin-bottom: 30px;
}
.items-table {
    width: 100%;
    border-collapse: collapse;
}
.items-table th, .items-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
}
</style>
```

## Custom Module Development

### 1. Create a New Custom App

```bash
# Inside the development container
cd /workspace/development/frappe-bench

# Create new app
bench new-app custom_app

# Install the app to your site
bench --site development.localhost install-app custom_app

# Add app to apps.txt for container builds
echo "custom_app" >> sites/apps.txt
```

### 2. Create Custom DocTypes

#### Using Web Interface

1. Go to **Setup > Customize > DocType**
2. Create new DocType with fields, permissions, and workflows

#### Using Code (Recommended for version control)

```python
# custom_app/custom_app/doctype/custom_product/custom_product.py
from frappe.model.document import Document

class CustomProduct(Document):
    def validate(self):
        # Custom validation logic
        if not self.product_code:
            self.product_code = self.generate_product_code()
    
    def generate_product_code(self):
        # Custom logic to generate product code
        return f"PROD-{frappe.utils.random_string(6)}"
    
    def on_submit(self):
        # Custom logic when document is submitted
        self.update_inventory()
    
    def update_inventory(self):
        # Custom inventory update logic
        pass
```

```json
// custom_app/custom_app/doctype/custom_product/custom_product.json
{
    "actions": [],
    "creation": "2024-01-01 00:00:00.000000",
    "doctype": "DocType",
    "editable_grid": 1,
    "engine": "InnoDB",
    "field_order": [
        "product_name",
        "product_code",
        "description",
        "price",
        "category"
    ],
    "fields": [
        {
            "fieldname": "product_name",
            "fieldtype": "Data",
            "label": "Product Name",
            "reqd": 1
        },
        {
            "fieldname": "product_code",
            "fieldtype": "Data",
            "label": "Product Code",
            "unique": 1
        },
        {
            "fieldname": "description",
            "fieldtype": "Text",
            "label": "Description"
        },
        {
            "fieldname": "price",
            "fieldtype": "Currency",
            "label": "Price"
        },
        {
            "fieldname": "category",
            "fieldtype": "Link",
            "label": "Category",
            "options": "Item Group"
        }
    ],
    "modified": "2024-01-01 00:00:00.000000",
    "module": "Custom App",
    "name": "Custom Product",
    "owner": "Administrator",
    "permissions": [
        {
            "create": 1,
            "delete": 1,
            "email": 1,
            "export": 1,
            "print": 1,
            "read": 1,
            "report": 1,
            "role": "System Manager",
            "share": 1,
            "write": 1
        }
    ],
    "sort_field": "modified",
    "sort_order": "DESC"
}
```

### 3. Create Custom APIs

```python
# custom_app/custom_app/api.py
import frappe
from frappe import _

@frappe.whitelist()
def get_custom_products(category=None):
    """API to get custom products with optional category filter"""
    filters = {}
    if category:
        filters['category'] = category
    
    products = frappe.get_all(
        'Custom Product',
        filters=filters,
        fields=['name', 'product_name', 'product_code', 'price', 'category']
    )
    
    return products

@frappe.whitelist(allow_guest=True)
def public_product_catalog():
    """Public API for product catalog"""
    products = frappe.get_all(
        'Custom Product',
        filters={'disabled': 0},
        fields=['product_name', 'description', 'price'],
        limit=50
    )
    
    return {
        'status': 'success',
        'data': products
    }

@frappe.whitelist()
def bulk_update_prices(products_data):
    """Bulk update product prices"""
    try:
        for product in products_data:
            doc = frappe.get_doc('Custom Product', product['name'])
            doc.price = product['new_price']
            doc.save()
        
        frappe.db.commit()
        return {'status': 'success', 'message': 'Prices updated successfully'}
    
    except Exception as e:
        frappe.db.rollback()
        return {'status': 'error', 'message': str(e)}
```

### 4. Create Custom Reports

```python
# custom_app/custom_app/report/product_sales_report/product_sales_report.py
import frappe
from frappe import _

def execute(filters=None):
    columns = get_columns()
    data = get_data(filters)
    return columns, data

def get_columns():
    return [
        {
            "label": _("Product"),
            "fieldname": "product_name",
            "fieldtype": "Link",
            "options": "Custom Product",
            "width": 200
        },
        {
            "label": _("Total Sales"),
            "fieldname": "total_sales",
            "fieldtype": "Currency",
            "width": 150
        },
        {
            "label": _("Quantity Sold"),
            "fieldname": "qty_sold",
            "fieldtype": "Int",
            "width": 120
        }
    ]

def get_data(filters):
    conditions = ""
    if filters.get("from_date"):
        conditions += f" AND si.posting_date >= '{filters.get('from_date')}'"
    if filters.get("to_date"):
        conditions += f" AND si.posting_date <= '{filters.get('to_date')}'"
    
    query = f"""
        SELECT 
            cp.product_name,
            SUM(sii.amount) as total_sales,
            SUM(sii.qty) as qty_sold
        FROM 
            `tabSales Invoice Item` sii
        JOIN 
            `tabSales Invoice` si ON sii.parent = si.name
        JOIN 
            `tabCustom Product` cp ON sii.item_code = cp.product_code
        WHERE 
            si.docstatus = 1 {conditions}
        GROUP BY 
            cp.name
        ORDER BY 
            total_sales DESC
    """
    
    return frappe.db.sql(query, as_dict=True)
```

## Custom App Development

### 1. Create apps.json for Custom Apps

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/your-username/custom_app",
    "branch": "main"
  },
  {
    "url": "https://github.com/your-org/private_app",
    "branch": "production"
  }
]
```

### 2. Build Custom Docker Image

```bash
# Generate base64 encoded apps.json
export APPS_JSON_BASE64=$(base64 -w 0 /path/to/apps.json)

# Build custom image using layered approach (faster)
docker build \
  --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
  --build-arg=FRAPPE_BRANCH=version-15 \
  --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
  --tag=your-registry.com/custom-erpnext:latest \
  --file=images/layered/Containerfile .

# Push to registry
docker push your-registry.com/custom-erpnext:latest
```

### 3. Custom Compose Configuration

Create `docker-compose.custom.yml`:

```yaml
version: '3.8'

services:
  backend:
    image: your-registry.com/custom-erpnext:latest
    environment:
      - CUSTOM_APP_SETTING=value
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - logs:/home/frappe/frappe-bench/logs
      - ./custom_configs:/home/frappe/frappe-bench/sites/common_site_config.json

  frontend:
    image: your-registry.com/custom-erpnext:latest
    command: nginx-entrypoint.sh
    environment:
      - BACKEND=backend:8000
      - SOCKETIO=websocket:9000
    ports:
      - "80:8080"

  websocket:
    image: your-registry.com/custom-erpnext:latest
    command: node /home/frappe/frappe-bench/apps/frappe/socketio.js

volumes:
  sites:
  logs:
```

## Cloud Deployment Strategies

### 1. AWS Deployment with ECS

#### ECS Task Definition

```json
{
  "family": "custom-erpnext",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::account:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "backend",
      "image": "your-registry.com/custom-erpnext:latest",
      "portMappings": [
        {
          "containerPort": 8000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "DB_HOST",
          "value": "your-rds-endpoint.amazonaws.com"
        },
        {
          "name": "REDIS_CACHE",
          "value": "your-elasticache-endpoint:6379"
        }
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:region:account:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/custom-erpnext",
          "awslogs-region": "us-west-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

#### CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Custom ERPNext Deployment'

Parameters:
  ImageURI:
    Type: String
    Description: Docker image URI
    Default: your-registry.com/custom-erpnext:latest

Resources:
  ECSCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: custom-erpnext-cluster

  ECSService:
    Type: AWS::ECS::Service
    Properties:
      Cluster: !Ref ECSCluster
      TaskDefinition: !Ref TaskDefinition
      DesiredCount: 2
      LaunchType: FARGATE
      NetworkConfiguration:
        AwsvpcConfiguration:
          SecurityGroups:
            - !Ref SecurityGroup
          Subnets:
            - !Ref PrivateSubnet1
            - !Ref PrivateSubnet2

  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Type: application
      Scheme: internet-facing
      SecurityGroups:
        - !Ref ALBSecurityGroup
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2
```

### 2. Google Cloud Platform with Cloud Run

#### Cloud Run Deployment

```bash
# Build and push to Google Container Registry
docker build -t gcr.io/your-project/custom-erpnext:latest .
docker push gcr.io/your-project/custom-erpnext:latest

# Deploy to Cloud Run
gcloud run deploy custom-erpnext \
  --image gcr.io/your-project/custom-erpnext:latest \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated \
  --memory 2Gi \
  --cpu 1 \
  --set-env-vars DB_HOST=your-cloud-sql-ip \
  --set-env-vars REDIS_CACHE=your-memorystore-ip:6379
```

#### Cloud Run YAML Configuration

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: custom-erpnext
  annotations:
    run.googleapis.com/ingress: all
spec:
  template:
    metadata:
      annotations:
        run.googleapis.com/cpu-throttling: "false"
        run.googleapis.com/memory: "2Gi"
        run.googleapis.com/cpu: "1000m"
    spec:
      containerConcurrency: 100
      containers:
      - image: gcr.io/your-project/custom-erpnext:latest
        ports:
        - containerPort: 8000
        env:
        - name: DB_HOST
          value: "your-cloud-sql-ip"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          limits:
            memory: "2Gi"
            cpu: "1000m"
```

### 3. Azure Container Instances

```bash
# Create resource group
az group create --name custom-erpnext-rg --location eastus

# Deploy container
az container create \
  --resource-group custom-erpnext-rg \
  --name custom-erpnext \
  --image your-registry.com/custom-erpnext:latest \
  --cpu 2 \
  --memory 4 \
  --ports 80 \
  --environment-variables \
    DB_HOST=your-azure-db.database.windows.net \
    REDIS_CACHE=your-redis.redis.cache.windows.net:6380 \
  --secure-environment-variables \
    DB_PASSWORD=your-secure-password
```

### 4. Kubernetes Deployment

#### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-erpnext
  labels:
    app: custom-erpnext
spec:
  replicas: 3
  selector:
    matchLabels:
      app: custom-erpnext
  template:
    metadata:
      labels:
        app: custom-erpnext
    spec:
      containers:
      - name: backend
        image: your-registry.com/custom-erpnext:latest
        ports:
        - containerPort: 8000
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: db-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /api/method/ping
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/method/ping
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: custom-erpnext-service
spec:
  selector:
    app: custom-erpnext
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  db-host: "your-database-host"
  redis-cache: "your-redis-host:6379"

---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: <base64-encoded-password>
```

## Advanced Customizations

### 1. Custom Workflows

```python
# custom_app/custom_app/custom_workflow.py
import frappe
from frappe.workflow.doctype.workflow_action.workflow_action import apply_workflow

@frappe.whitelist()
def custom_approval_workflow(doc, action):
    """Custom approval workflow with additional business logic"""
    
    # Pre-workflow validation
    if action == "Approve" and not validate_approval_criteria(doc):
        frappe.throw("Approval criteria not met")
    
    # Apply standard workflow
    apply_workflow(doc, action)
    
    # Post-workflow actions
    if action == "Approve":
        send_approval_notifications(doc)
        update_related_documents(doc)

def validate_approval_criteria(doc):
    """Custom validation logic for approval"""
    if doc.doctype == "Purchase Order" and doc.grand_total > 10000:
        # Require additional approver for high-value orders
        return check_additional_approval(doc)
    return True

def send_approval_notifications(doc):
    """Send custom notifications after approval"""
    frappe.sendmail(
        recipients=get_stakeholders(doc),
        subject=f"{doc.doctype} {doc.name} has been approved",
        message=f"The {doc.doctype} {doc.name} has been approved and is ready for processing."
    )
```

### 2. Custom Integrations

```python
# custom_app/custom_app/integrations/external_api.py
import frappe
import requests
from frappe.integrations.utils import make_post_request

class ExternalAPIIntegration:
    def __init__(self, base_url, api_key):
        self.base_url = base_url
        self.api_key = api_key
        self.headers = {
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        }
    
    def sync_customers(self):
        """Sync customers with external CRM"""
        customers = frappe.get_all('Customer', fields=['name', 'customer_name', 'email_id'])
        
        for customer in customers:
            try:
                response = self.push_customer_to_crm(customer)
                if response.get('success'):
                    self.update_sync_status(customer['name'], 'Synced')
                else:
                    self.update_sync_status(customer['name'], 'Failed')
            except Exception as e:
                frappe.log_error(f"Customer sync failed: {str(e)}", "External API Integration")
    
    def push_customer_to_crm(self, customer):
        """Push customer data to external CRM"""
        url = f"{self.base_url}/customers"
        data = {
            'name': customer['customer_name'],
            'email': customer['email_id'],
            'external_id': customer['name']
        }
        
        response = requests.post(url, json=data, headers=self.headers)
        return response.json()
    
    def update_sync_status(self, customer_name, status):
        """Update sync status in customer record"""
        customer_doc = frappe.get_doc('Customer', customer_name)
        customer_doc.custom_sync_status = status
        customer_doc.custom_last_sync = frappe.utils.now()
        customer_doc.save(ignore_permissions=True)

# Scheduled job to run sync
def sync_with_external_crm():
    """Scheduled function to sync data with external CRM"""
    settings = frappe.get_single('External API Settings')
    if settings.enable_sync:
        integration = ExternalAPIIntegration(settings.api_url, settings.api_key)
        integration.sync_customers()
```

### 3. Custom Dashboard and Analytics

```python
# custom_app/custom_app/dashboard/custom_dashboard.py
import frappe
from frappe.utils import flt, cint

@frappe.whitelist()
def get_dashboard_data(filters=None):
    """Get custom dashboard data"""
    
    data = {
        'summary_cards': get_summary_cards(filters),
        'charts': get_chart_data(filters),
        'recent_activities': get_recent_activities(filters)
    }
    
    return data

def get_summary_cards(filters):
    """Get summary card data"""
    return [
        {
            'title': 'Total Sales',
            'value': get_total_sales(filters),
            'change': get_sales_change(filters),
            'color': 'green'
        },
        {
            'title': 'Active Customers',
            'value': get_active_customers(filters),
            'change': get_customer_change(filters),
            'color': 'blue'
        },
        {
            'title': 'Pending Orders',
            'value': get_pending_orders(filters),
            'change': get_orders_change(filters),
            'color': 'orange'
        }
    ]

def get_chart_data(filters):
    """Get chart data for dashboard"""
    return {
        'sales_trend': get_sales_trend_data(filters),
        'product_performance': get_product_performance_data(filters),
        'customer_segments': get_customer_segment_data(filters)
    }

def get_sales_trend_data(filters):
    """Get sales trend data for chart"""
    query = """
        SELECT 
            DATE(posting_date) as date,
            SUM(grand_total) as total_sales
        FROM 
            `tabSales Invoice`
        WHERE 
            docstatus = 1
            AND posting_date >= %(from_date)s
            AND posting_date <= %(to_date)s
        GROUP BY 
            DATE(posting_date)
        ORDER BY 
            date
    """
    
    return frappe.db.sql(query, filters, as_dict=True)
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Container Build Issues

**Problem**: Custom app not found during build
```bash
# Solution: Ensure apps.json is correctly formatted
echo $APPS_JSON_BASE64 | base64 -d | jq .

# Verify app accessibility
git clone <your-app-repo-url>
```

**Problem**: Permission denied errors
```bash
# Solution: Fix file permissions
docker run --rm -v $(pwd):/workspace alpine chmod -R 755 /workspace
```

#### 2. Runtime Issues

**Problem**: Database connection failed
```bash
# Check database connectivity
docker compose exec backend ping db

# Verify database credentials
docker compose exec backend env | grep DB_
```

**Problem**: Redis connection issues
```bash
# Test Redis connectivity
docker compose exec backend redis-cli -h redis-cache ping

# Check Redis configuration
docker compose exec backend bench get-config
```

#### 3. Custom App Issues

**Problem**: Custom app not loading
```bash
# Check if app is installed
docker compose exec backend bench list-apps

# Reinstall app
docker compose exec backend bench --site <site-name> install-app <app-name>

# Check app status
docker compose exec backend bench --site <site-name> list-apps
```

**Problem**: Migration errors
```bash
# Run migrations manually
docker compose exec backend bench --site <site-name> migrate

# Check migration status
docker compose exec backend bench --site <site-name> show-config
```

### Debugging Techniques

#### 1. Container Debugging

```bash
# Access container shell
docker compose exec backend bash

# Check container logs
docker compose logs backend -f

# Monitor resource usage
docker stats

# Inspect container configuration
docker inspect <container-name>
```

#### 2. Application Debugging

```bash
# Enable developer mode
docker compose exec backend bench --site <site-name> set-config developer_mode 1

# Check error logs
docker compose exec backend tail -f logs/worker.error.log

# Python debugging
docker compose exec backend bench --site <site-name> console
```

#### 3. Performance Monitoring

```bash
# Monitor database queries
docker compose exec backend bench --site <site-name> mariadb

# Check slow queries
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

# Monitor Redis
docker compose exec redis-cache redis-cli monitor
```

## Best Practices

### 1. Development Best Practices

- **Version Control**: Always use Git for custom apps and configurations
- **Environment Separation**: Use different environments for development, staging, and production
- **Code Quality**: Implement linting, testing, and code review processes
- **Documentation**: Document all customizations and deployment procedures

### 2. Security Best Practices

- **Secrets Management**: Use proper secret management (AWS Secrets Manager, Azure Key Vault, etc.)
- **Network Security**: Implement proper network segmentation and firewall rules
- **Container Security**: Regularly update base images and scan for vulnerabilities
- **Access Control**: Implement proper RBAC and authentication mechanisms

### 3. Performance Best Practices

- **Resource Limits**: Set appropriate CPU and memory limits for containers
- **Caching**: Implement proper caching strategies (Redis, CDN)
- **Database Optimization**: Optimize database queries and indexes
- **Monitoring**: Implement comprehensive monitoring and alerting

### 4. Deployment Best Practices

- **Blue-Green Deployment**: Implement zero-downtime deployment strategies
- **Health Checks**: Configure proper health checks for all services
- **Backup Strategy**: Implement automated backup and disaster recovery
- **Scaling**: Design for horizontal scaling and load balancing

### 5. Maintenance Best Practices

- **Regular Updates**: Keep Frappe, ERPNext, and custom apps updated
- **Monitoring**: Monitor application performance and resource usage
- **Backup Testing**: Regularly test backup and restore procedures
- **Security Audits**: Conduct regular security audits and vulnerability assessments

## Conclusion

This guide provides comprehensive coverage of customizing Frappe/ERPNext applications in containerized cloud environments. From basic UI modifications to complex custom module development and cloud deployment strategies, these practices will help you build robust, scalable, and maintainable business applications.

Remember to always test customizations thoroughly in development environments before deploying to production, and maintain proper documentation for all customizations to ensure long-term maintainability.

For additional support and community resources, refer to:
- [Frappe Framework Documentation](https://frappeframework.com/docs)
- [ERPNext Documentation](https://docs.erpnext.com)
- [Frappe Docker GitHub Repository](https://github.com/frappe/frappe_docker)
- [Frappe Community Forum](https://discuss.frappe.io)