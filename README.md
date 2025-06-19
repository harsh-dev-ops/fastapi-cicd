
## Key Components Explained

### Dockerfile Features
1. Uses official Python 3.11 slim image
2. Creates non-root user for security
3. Optimized layer caching
4. Health check for container readiness
5. Production-ready configuration

### Jenkinsfile Features
1. **Dynamic Port Allocation**: Avoids port conflicts by letting Docker assign ports
2. **Health Check**: Ensures container is fully ready before testing
3. **Automatic Cleanup**: Removes containers and images after tests
4. **Four-Stage Pipeline**:
   - Build Docker image
   - Run container with health checks
   - Test API endpoints
   - Cleanup resources

### Testing Approach
1. **Container Health**: Uses Docker healthcheck status
2. **API Validation**:
   - Verifies root endpoint returns correct message
   - Tests parameterized route with dynamic value
3. **Graceful Failure**: Pipeline fails if any test doesn't pass