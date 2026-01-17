# DevOps Pipeline Challenge

```
Time: 75 Minutes
```

## Problem

You are given a Python FastAPI application (Task Management API) that runs locally. Your task is to prepare it for production deployment by containerizing the application, creating Kubernetes manifests, and setting up a CI/CD pipeline.

## Application Overview

The application is an advanced Task Management API built with FastAPI featuring:
- **Full CRUD operations** with Pydantic validation
- **Advanced filtering, sorting, and pagination**
- **Task dependencies and relationships**
- **Background job processing**
- **Heavy analytics computations** (statistics, productivity metrics)
- **Bulk operations** for creating multiple tasks
- **Auto-generated API documentation** at `/docs`

**Key endpoints:**
- `GET /health` - Health check with system metrics
- `GET /tasks` - Get tasks (filters: status, priority, category, assignee, tags)
- `POST /tasks` - Create task with validation
- `PUT /tasks/{id}` - Update task
- `DELETE /tasks/{id}` - Delete task (handles dependencies)
- `GET /tasks/stats/summary` - Comprehensive statistics
- `GET /tasks/analytics/productivity` - Productivity analytics
- `POST /tasks/bulk` - Bulk create tasks

**Files provided:**
```
do-uno/
├── src/
│   ├── app.py              # Main FastAPI application (450+ lines)
│   └── RUN_APP.md          # Quick start guide for running the app
├── test_app.py             # Comprehensive unit tests (17 test cases)
├── requirements.txt        # Python dependencies
└── README.md               # This file (exercise instructions)
```

## Requirements

### 1. Dockerization

Create a `Dockerfile` that:
- Uses an appropriate Python base image
- Installs dependencies efficiently
- Runs the application with a production server
- Exposes the application port
- Follows Docker best practices for security and optimization

### 2. Kubernetes Deployment

Create Kubernetes manifests:

**Deployment (`deployment.yaml`):**
- Deploy the containerized application
- Configure resource limits and requests
- Set up liveness and readiness probes
- Use multiple replicas for high availability
- Configure necessary environment variables

**Service (`service.yaml`):**
- Expose the application
- Configure appropriate service type and ports

### 3. CI/CD Pipeline

Create a CI/CD pipeline configuration for your preferred platform (GitHub Actions, GitLab CI, or Jenkins).

The pipeline should:
- Run tests
- Build and tag Docker image
- Push image to a container registry
- Deploy to Kubernetes cluster

### 4. Documentation

Create a `DEPLOYMENT.md` file that includes:
- Instructions for building and running the Docker container locally
- Instructions for deploying to Kubernetes
- Explanation of your CI/CD pipeline workflow
- Any assumptions or prerequisites
- Security considerations you've implemented

## Running the Application Locally

Before starting your DevOps work, verify the application runs:

```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
python src/app.py

# In another terminal, run tests
pytest test_app.py -v

# Access the API documentation
# Open http://localhost:8000/docs in your browser
```

For detailed instructions, see `src/RUN_APP.md`

## Submission

- Create a public git repository containing your submission and share the repository link
- Do not fork this repository or create pull requests
