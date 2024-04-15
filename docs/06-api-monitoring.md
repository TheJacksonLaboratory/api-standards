# **6** API Monitoring
> These guiding principles will help stability, uptime, and awareness for your applications.

Actuator endpoints let you monitor and interact with your application. 

## **6.1** Monitoring
- **6.1.1** Endpoints for monitoring and info will exist at:

    ```
    /api/monitor/<resource>/<type>
    ```

## **6.2** Health Endpoint

!!! warning "Required"
    This endpoint is **required** for all applications.

- **6.2.1** This endpoint provides health information, mainly the status of the 
application. It will be the primary target for monitoring services.

    Location:
    ```
    http://127.0.0.1/api/monitor/server/health
    ```

Example Response:
````
{"name":"Fancy Application","status":"UP","details":"Everything seems okay."}
````

| Status Text  | Http Status Code |
|--------------|------------------|
| `UP`         | 200              |
| `UNKNOWN`    | 200              |
| `DOWN`       | 503              |
