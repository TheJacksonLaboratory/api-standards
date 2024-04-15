# **4** OpenAPI Schemas

!!! success "Overview"
    As part of our commitment to creating well-defined, reliable, and easy-to-use APIs, 
    we adopt OpenAPI[^1] as a standard tool for defining, creating, and documenting our 
    APIs. OpenAPI helps our teams collaborate more efficiently, enhances the developer 
    experience, and maintains the quality of our APIs. This document lays out standards 
    and best practices around the usage and publication of OpenAPI schemas.

## Why OpenAPI?

OpenAPI[^1] is an industry-standard method for describing RESTful APIs. By requiring an 
OpenAPI document[^2] for each API, we aim to:

- **Increase Transparency** 
    
    With an OpenAPI document, every aspect of an API is clearly
described, from endpoints to response formats. This reduces guesswork and the potential 
for misunderstandings.

- **Streamline Collaboration** 

	Frontend and backend teams can reference the OpenAPI 
document to understand what data is available, how to access it, and what kind of 
responses to expect. 

- **Simplify Integration** 

	Other systems or third-party developers can use the OpenAPI 
document to understand how to integrate with our APIs. 

- **Enable API Testing and Monitoring** 

	OpenAPI documents can be used to generate 
testing scripts and monitor APIs for any discrepancies in expected behavior.

- **Improve Developer Experience** 

	An OpenAPI document forms the basis for generating 
interactive documentation, SDKs, and API explorers, enhancing the overall developer 
experience.

## Key Principles

1. **Up-to-date Schema** 

	Each RESTful API must always provide an up-to-date 
`openapi.json` file, reflecting the latest version of the API. 

2. **Automatic Generation** 

	The `openapi.json` file should be automatically generated 
from the source code and annotations as part of the build pipeline.

3. **Public Accessibility** 

	The `openapi.json` file should be easily accessible through
a dedicated endpoint.

    - (3.a) The openapi.json file will be available at the root of your API.

4. **Auto-generated API Documentation** 

	The OpenAPI schema should be used to 
automatically generate and update API navigator pages or interactive API documentation 
(like Swagger UI).

5. **Schema Definitions** 

	Schema definitions should _always_ be as specific as 
possible. Generic schema usage (like `object`) should be avoided.

6. **Direct Object Return** 

	All endpoints returning object structures provide these 
objects directly in the response body without wrapping them in additional container 
objects. This approach minimizes unnecessary parsing and improves the direct usability 
of our API responses.


## Detailed Standards

### **4.1** OpenAPI File Generation

- **4.1.1** 

	Use tools and libraries that support OpenAPI file generation from code annotations 
(e.g., Swagger for Java or SpringFox). 

- **4.1.2** 

	The OpenAPI file should be generated as part of the build process. 

- **4.1.3** 

	Keep your annotations up-to-date as you make changes to your API.

### **4.2** OpenAPI File Hosting

- **4.2.1** 

	The `openapi.json` file should be publicly accessible via a dedicated URL.

- **4.2.2** 

	The file should be placed at a consistent location across all APIs for easy discovery 
  (e.g., `https://myapp.jax.com/openapi.json`).

- **4.2.3** 

	The server should deliver the `openapi.json` file with the correct MIME type 
`application/json`.

- **4.2.4** 

	The openapi.json file will be available at the root of your API. For 
  example, if `myapp.jax.org/api` is the root of your API, the openapi.json file will
  be available at `myapp.jax.org/api/openapi.json` 

### **4.3** Versioning

- **4.3.1** 

	The OpenAPI document should reflect the current version of the API and be updated with
each version change.

- **4.3.2** 

	If multiple versions of the API exist, each version should have its own `openapi.json`
file.


### **4.4** Schema Definitions

- **4.4.1** 

	Schema definitions should _always_ be as specific as possible.

- **4.4.2** 

	Edpoints that return an object **must** define that object as a schema that matches 
the structure of the object to be returned.

- **4.4.3** 

	The generic `object`(`{}`) schema should not be used unless the structure of response 
object is not known until runtime.

### **4.5** Direct Object Return

- **4.5.1** 

	When an endpoint returns an object, the response should directly contain the object's 
properties without nesting it within an outer container schema. 

- **4.5.2** 

	When an endpoint returns a collection of objects, the response should directly return
a list of those objects, without nesting it within an outer container schema

??? tip "Compliant Example"
    ```
    {
      "some_key": "some_value"
    }
    ```

    ```
    [
        {
            "some_key": "some_value"
        },
        {
            "some_other_key": "some_other_value"
        }
    ]
    ```


??? tip "Non-Compliant Example"
    ```
    {
      "mything": {
        "some_key": "some_value"
      }
    }
    ```


### **4.6** Auto-generated API Documentation

- **4.6.1** 

	Use a tool like Swagger UI or ReDoc to automatically generate interactive API 
documentation from the OpenAPI schema.

- **4.6.2** 

	The documentation should be publicly accessible and updated automatically whenever the 
OpenAPI schema is updated.

- **4.6.3** 

	The documentation should provide interactive features such as the ability to send test
requests.

!!! info
    By adopting OpenAPI and following these standards and best practices, we can improve 
    the developer experience, enhance the discoverability of our APIs, and ensure our 
    documentation is always up-to-date.

[^1]: [OpenAPI Initiative](https://www.openapis.org/)
[^2]: [OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0)
