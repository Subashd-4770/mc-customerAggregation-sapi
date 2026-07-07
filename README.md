Flow Functionality:

When a client requests customer details, the API performs the following steps:

Receives the Customer ID from the HTTP request and routes the request using APIKit.
Extracts the Customer ID from the URI parameters for backend processing.
Executes three backend requests simultaneously using Scatter-Gather to improve performance:
Retrieves customer profile information from Salesforce.
Retrieves customer order history from the Database.
Retrieves customer billing information from Snowflake.
Each backend request is executed inside its own Try Scope with dedicated error handling, ensuring failures in one system do not interrupt processing of the remaining systems.
Logs the request using a Correlation ID to support end-to-end request tracing and easier troubleshooting.
Uses DataWeave to aggregate the responses from all three backend systems into a single Customer 360 JSON response.
Returns the consolidated customer information along with timestamps and status details to the client.

Error Handling

The application implements error handling at the API, connector, and application levels to provide consistent and meaningful responses.

API-Level Error Handling

APIKit validates all incoming requests before they reach the business logic.

The API returns appropriate HTTP status codes for:

400 – Bad Request
404 – Resource Not Found
405 – Method Not Allowed
406 – Not Acceptable
415 – Unsupported Media Type
501 – Not Implemented

This ensures invalid requests are rejected with clear error messages.

Connector-Level Error Handling:

Each backend integration is wrapped inside a Try Scope with dedicated error handling using On Error Continue.

Salesforce Handles:

Connectivity failures

If Salesforce is unavailable, the route returns a standardized error object while the remaining routes continue processing.

Database Handles:

Database connectivity failures
SQL query execution failures

Database errors are captured without stopping the Scatter-Gather execution.

Snowflake Handles:

Connectivity failures
Query execution failures

Snowflake errors are converted into structured error responses while allowing other backend requests to complete.

Scatter-Gather Error Handling:

The API retrieves customer information from Salesforce, Database, and Snowflake in parallel using Scatter-Gather.

Each route executes independently. If one backend service encounters an error, its route returns a structured error payload while the successful routes continue processing. The final response still contains the available data from all completed routes.

A timeout of 20 seconds is configured for the Scatter-Gather component. If the parallel execution exceeds this limit, the API returns a 504 Gateway Timeout response.
