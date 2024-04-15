# **7** REST API Error Handling Best Practices  
  
Error handling is crucial for ensuring the reliability and usability of a REST API service. Proper error handling not only helps developers identify and fix issues but also provides clear and informative responses to clients so they may take corrective action.

# Guidelines
## **7.1** Use Appropriate HTTP Status Codes  
  
HTTP status codes are essential for conveying the outcome of API requests and allow a high level grouping of error responses. Use them consistently to indicate the result of each API call, and avoid using obscure codes that API consumers may not be familiar with.  Common, widely used error codes provide clear semantics and promote consistency and interoperability. These include:  
  
- **400 Bad Request**: Malformed request syntax or invalid parameters  
- **401 Unauthorized**: Authentication required or invalid credentials  
- **403 Forbidden**: The authenticated user does not have permission to perform the operation  
- **404 Not Found**: Requested resource does not exist
- **422 Unprocessable Content**: The request is formed correctly but there are semantical errors in the payload
- **500 Internal Server Error**: Generic server-side error
  
## **7.2** Provide Detailed Error Messages  
  
When an error occurs, include detailed error messages in the response. These messages should provide enough information for consumers to understand the cause of the error and take appropriate actions. However, be cautious not to expose sensitive information or specifics that could be exploited by malicious users.  For example, do not include implementation details or detailed stack traces.  
**Error messages should clarify the problem and communicate the intended functionality**. For example, if a type check fails on an API to fetch a study record by id, the message "Study ID must be an integer" clearly conveys what is expected.

##   **7.3** Implement Consistent Error Response Format

Follow a consistent error response format across all API endpoints, and format error response payloads as JSON. This makes it easier for clients to parse error responses and handle them gracefully. Include fields **status, request_id, error_code, message, timestamp, and trace_id** in your error responses.
  
```json  
{  
    "status": "",               HTTP Status Code for the entire request
    "request_id": "",           Request identifier generated by the API service
    "errors": [{          
          "error_code": "",     Error code (see below)     
          "message": "",        Human readable error details
          "timestamp": "",      Date/time of the error
          "trace_id": ""        Pointer to the log trace
     }]
}
```

## **7.4** Return An Errors Array
It is possible for multiple errors to occur within one transaction, so for consistency return an errors collection even if there is only one error. 

## **7.5** Log Errors for Debugging

Log errors on the server-side to aid in debugging and monitoring. Include relevant details like error message, error code, request URL, source, user ID (if available), stack trace, and timestamp. Log errors at appropriate severity levels based on the error's impact, for example "Critical", "Error", "Warning", "Info", "Debug".  If you would like to learn about BioConnect's logging service, please reference the documentation [here](https://docs.bioconnect.jax.org/core-modules/logging-service/#the-logging-service).

## **7.6** Error Codes
Use meaningful error codes in addition to HTTP status codes to provide more specific information. CS is developing a standard list of error codes that may be used across all of our software applications, providing the following benefits:
**7.6.1** **Clarity and specificity**: HTTP status codes are useful but can be generic and lack context.  In addition to providing status codes, error codes allow us to convey more detailed information about the nature of the error.
**7.6.2** **Consistency**: By aligning on a standard list of error codes, we can ensure that error responses are uniform and predictable, which will make it easier for front end developers to handle errors and provide consistent messaging to users.
**7.6.3** **Error reporting and troubleshooting**: If errors are properly logged, developers will be able to aggregate log data based on error codes, providing insight into errors that occur most frequently.



The following examples show error codes, in combination with status codes and error messages:
1. invalid parameter

```json  
{  
    "status": "400",
    "request_id": "37472a48-a34e-4813-b064-f863170f33fc",
    "errors": {
          "error_code": "parameter_invalid",  
          "message": "Cannot convert 'abc' to integer",  
          "timestamp": "2023-07-02T14:07:01",
          "trace_id": "fb3a02ac6caa"
     }  
}
```

2. value too large for column in database

```json  
{  
    "status": "400",
    "request_id": "a83e5e07-06b2-44f9-a18a-8eefdc3f9bf8",
     "error": {
          "error_code": "value_invalid",  
          "message": "animal name must be 50 characters or less",
          "timestamp": "2023-08-24T01:10:00",
          "trace_id": "3ff84c1df586"

     }  
}
```

Below is the current working list of error codes for CS Rest APIs:

400-Bad Request

- parameter_missing
- parameter_invalid
- parameter_length_exceeded
- header_invalid

401-Unauthorized

- api_key_required
- api_key_invalid
- credentials_required
- credentials_invalid

403-Forbidden

- privileges_insufficient
- request_limit_exceeded

404-Not found

- url_invalid

408-Timeout exceeded

- timeout_exceeded

422-Unprocessable content

- value_missing
- value_invalid
- value_length_exceeded


# Use Cases & Examples

**Missing/Invalid Parameters**
Consider an API that returns a list of animals based on a weight min and max value. A
validation check is performed to ensure that both parameters are present and numeric. If
the min value type check validation fails, a status code of 400 is returned, along with
a "parameter_invalid" error code and the message "min value must be numeric". If the
parameter is missing all together, the following response is issued:

```
        min = request.query_params.get('min_value', None)
        if min is None:
            return Response(
              {
                "status": "400",
                "request_id": "7abfde24-b2e8-49f9-af5b-5f45bd56cce3",
                "errors":[
                  {
                    "error": {
                      "error_code": "parameter_missing",  
                      "message": "Minimum weight value is required",
                      "timestamp": "2023-09-24T01:10:00",
                      "trace_id": "48785f565a27"
                    }
                  }
                ]
              }
            )
```
This example is a good use case for aggregating and reporting on error data in order to improve user experience.  If your error logs are showing high numbers of "parameter_invalid" codes for a certain API, perhaps you should look into front end validation and constraints in order to prevent them.


**Insufficient Privileges**
It is important to remember that errors involving privileges fall under the 403 status
code rather than 401. In the case of user based actions, a 401 status code essentially
means "I don't recognize you" and 403 means "I know who you are but you're not allowed
to do this."

A typical use case involving insufficient privileges is an attempt to update a record
when the user does not have read/write access to the data. The API logic should perform
an authorization check before an attempt to update the record, and if not authorized,
the response should inform the user that their request has been denied due to
insufficient privileges.

```
return Response(
  {
    "status": "403",
    "request_id": "5bbca962-1f57-4016-bac5-5ae9a28d7d2e",
    "errors":[
      {
        "error_code": "privileges_insufficient",  
        "message": "You do not have sufficient privileges to update this record",
        "timestamp": "2023-09-24T01:10:00",
        "trace_id": "48785f565a27"
      }
    ]
  }
)
```

!!! tip "Not all 403 errors are tied to authenticated users"
    They could also be used for anonymous actions that can only be performed under certain
    circumstances (e.g. time based restrictions.)  Another example is a server that only
    accepts requests from a predefined range of IP addresses.

**Database Errors**
Database errors can be due to authentication/authorization issues, lost connection, or
data integrity issues such as constraint violations. Database libraries often have some
built in error handling that can be leveraged. When possible, you should catch and
handle specific exceptions rather than generic "Exception" handlers.
Wrap database operations in try-except blocks to catch and handle exceptions, for
example:

```
        try:
            study= Study.objects.get(pk=study_id)
            return Response({'identifier': study.identifier})
        except Item.DoesNotExist:
            return Response(
               {
                  "status": "404",
                  "request_id": "5bbca962-1f57-4016-bac5-5ae9a28d7d2e",
                  "errors":[
                    {
                      ...
                    }
                  ]
                }
            )
```

!!! tip "APIs should be idempotent"
There is no way to group requests together in a transaction in REST APIs. To compensate
for this, you should endeavor
to [make your APIs idempotent](https://restfulapi.net/idempotent-rest-apis/). With the 
exception of a POST request, sending the same request multiple times should produce the 
same result. To validate idempotency, implement test cases encompassing multiple 
invocations of the same request to ensure consistent outcomes irrespective of the number
of executions.

**Timeout**
Timeout errors can occur when there are network or service issues, or if a call to the
API involves too much data. A retry mechanism may be used to handle timeout errors,
although this would normally be handled in front end client code when a 504 (Gateway
timeout) is received. If timeouts are occurring due to high server load, consider load
balancing and scaling your application to ameliorate them. Also consider monitoring and
alerts to notify you when timeouts become frequent.

**Errors from 3rd Party Libraries**
When calling methods in a 3rd party library, you can follow best practices for handling
specific errors provided you have visibility into the exceptions it may raise. Consider
handling these errors with graceful degradation if it's possible for your application to
continue to function when they occur, for example if you can't retrieve data use cached
data as a fallback. If it's a 3rd party service, and it's possible for it to be down,
consider implementing a retry mechanism.

An API end point that makes a failed call to a 3rd party service should return a 200
response if the error was isolated to the 3rd party service.

**Business Logic Failures**
If a request includes data that fails a back end business logic check, return a status
code of 422 (Unprocessable Entity.)  This indicates that the syntax is correct but the
value(s) prevented the call from being successfully processed. Include data in the
response indicating the specific failure. For example, and end point that creates an
animal record expects a date of birth but the date provided is in the future and fails a
business rule check. A "value_invalid" error code and message stating "Date of birth
cannot be in the future" allows the client to correct the error.