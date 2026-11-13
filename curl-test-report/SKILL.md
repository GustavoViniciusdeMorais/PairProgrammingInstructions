---
name: curl-test-report
description: Tests API endpoints with curl directly from the host server CLI, without entering the container. Use when: validating an endpoint end to end, confirming a route returns the expected outcome, reproducing a request against the running API, checking a payload or query string, or producing a curl test report. Triggers: "test this endpoint", "curl test", "call the API", "validate the route", "run curl", "check the response", "endpoint test report".
---

# Curl Test Report

## Goal

Exercise a real API endpoint from the host shell, capture the actual response, and report
whether the endpoint produced the desired outcome. No container entry, no Postman, no code
changes unless the user asks for them.

## Ground Rules

- Run every command from the host CLI, against the running environment. Do not `docker exec`
  into the container unless the user explicitly asks.
- This project is a RESTful API. Payload messages in Portuguese must stay in Portuguese.
- Do not invent routes, payload fields, or query parameters. Read the code first.
- Collect every missing value with `vscode_askQuestions`: server, port, endpoint path, email,
  password and payload params. Never assume them.
- The host CLI has Python. Use Python to format JSON responses, extract fields, compare
  values and any other helper task the report needs.
- You may write `.sh` files in `/tmp` to hold the shell variables, and `.json` files in
  `/tmp` to hold the payload and the captured response.
- One step at a time. Ask the user to approve the mounted curl before running it. One
  approval is enough for read-only and state-changing calls.
- Report only what the command actually returned. Never report assumptions.

## Workflow

### Step 1 - Collect the Inputs

Ask with `vscode_askQuestions` for whatever the prompt did not provide:

- API server host and port.
- Login endpoint path under test.
- Auth header name and its value, since the auth mechanism differs per API.
- Email and password, only when the login route requires them.

Mount the login call in this shape, with the header supplied by the user:

```bash
curl --request POST 'http://{server}:{port}/{login-path}' \
--header '{auth-header}: {auth-value}'
```

Capture the returned token into a shell variable and reuse it in every later curl.

Decision point: if login returns a non-2xx status, stop and report the raw response. Do not
continue to the endpoint under test with an invalid token.

### Step 2 - Read the Code

Locate the endpoint before writing any curl:

- Route definition (method, URI, middleware, auth guard).
- Controller method and the Action it delegates to.
- Validation rules, required and optional fields.
- Response shape (Resource, Collection, or plain JSON) and status codes.
- Base URI prefix for the version being tested.

Extract the exact path parameters, query string keys, body keys, and header requirements.

### Step 3 - Prepare the Request

Build either:

- URL query string, when the endpoint takes filters or pagination, or
- JSON payload body, when the endpoint receives data.

If any payload param or query key was not passed in the prompt, ask for it with
`vscode_askQuestions`. List the values chosen for each parameter and where they came from in
the code.

### Step 4 - Mount the Curl

Assemble the full command with:

- Method and full URL.
- Auth header carrying the token variable from Step 1.
- `Content-Type: application/json` for body requests.
- `--data-raw` for the JSON body.
- `--silent --show-error` optional, only when the user wants a clean capture.

Shape of the mounted call:

```bash
curl --request POST 'http://{server}:{port}/{endpoint}' \
--header '{auth-header}: {auth-value}' \
--header 'Content-Type: application/json' \
--data-raw '{...}'
```

### Step 5 - Ask for Review

Present the mounted curl to the user and ask for approval with the questions tool. Include:

- The endpoint and method being tested.
- The expected outcome.

Wait for confirmation before executing. Never run a curl without approval.

### Step 6 - Run the Curl

Execute the approved command from the host shell. When the run needs variables or files,
write them in `/tmp`:

- A `.sh` file holding the shell variables, the token for instance.
- The `.json` payload file used by `--data-raw '@file'`.
- The `.json` response file written from the curl output.

If the request needs interactive input, collect values with the questions tool, never
through a pipe. Do not expose secrets.

### Step 7 - Format the Result

Normalize the raw output for reading:

- HTTP status code.
- Response headers, when relevant to the assertion.
- Body formatted as JSON, with the key fields surfaced.

Use Python from the host CLI to pretty print the response file, pull the fields that matter
and compare them against the expected values.

### Step 8 - Analyze the Outcome

Compare the actual result against the expected outcome from Step 2:

- Status code matches the code contract.
- Response structure matches the Resource or Collection.
- Values reflect the payload sent.
- Error messages are the expected Portuguese texts.

State clearly whether the endpoint behaved as intended, and cite the evidence from the
actual response.

## Report Format

Deliver the report with these topics, skipping one only when it does not apply:

- **Summary**: one short paragraph on what was tested and the endpoint behavior.
- **Request**: method, URL, headers, payload or query string.
- **Response**: status code and formatted body.
- **Outcome**: intended versus actual, with the evidence.
- **Result**: whether the test was successful or finished with errors.
- **JSON Response**: the full response body, formatted as JSON.
- **References**: routes, controller, action, resource files read, and the endpoint host.

## Completion Check

- Token obtained and valid.
- Endpoint contract read from the code, not guessed.
- Curl approved by the user before execution.
- Real output captured and formatted, files kept in `/tmp` when used.
- Outcome compared against the expected behavior.
