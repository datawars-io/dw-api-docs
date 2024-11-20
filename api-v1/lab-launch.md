# Lab Launch Flow

Launching a lab refers to initiating a lab session for a Platform (LMS) user so they can complete the lab using DataWars' infrastructure and app.

The user may or may not have an existing session or account on DataWars. The launch process ensures that the user exists and is properly authenticated before redirecting them to the labs app.

The launch flow is illustrated in the following diagram:

![diagram](https://github.com/user-attachments/assets/61b374b2-ac33-4d46-b289-b6b7a7682135)

If you're a platform looking to integrate with DataWars labs, you'll only need to make a POST request to the `/api/v1/launch` endpoint (see details below), and then redirect the user to the `url` provided in the response.

```
POST https://api.dev.namespace.im/api/v1/launch
Authorization: Token <TOKEN>
{
    "user_id": "LMS-Us3r1D",
    "content_id": "8715dfc6-56ac-4ca2-99cf-17419d36cebb",
    "resource_id": "abc123",
    "context": {
        "course": "Databases 101",
        "chapter": "Intro to DBs",
        "unit": "Unit 2",
    },
    "email": "john.doe@test.com",
    "given_name": "John",
    "family_name": "Doe",
    "roles": [
        "admin",
    ],
    "licenses": [
        "program-1"
    ],
    "teams": [
        "Summer 2024 - Class 1"
    ]
}
```

> NOTE: Both `datawars.io` and `namespace.im` domains are supported. If you are integrating from a third-party platform, we recommend using `namespace.im` to launch labs in whitelabel mode, without exposing DataWars' branding to your customers.

### Authentication

To authenticate the request, you must provide an `Authorization` header with a valid token.

The token is an alphanumeric 40-character string. It must be kept private, **make sure to NOT expose this token to any customer or user outside of your organization**.

Contact DataWars' support team to obtain a token.


### Payload

* `user_id`: (required) A unique identifier for the user in the Platform (LMS). This is external to DataWars and will be stored as `external_id` in the authenticated user.

* `content_id`: (required) The UUID of the project within DataWars where the user will be redirected.

* `resource_id`: A unique identifier of the content in the Platform (LMS) that links to this project, where the grading service should report back when the user makes progress. This is optional unless grading reports are required, in which case it is mandatory.

* `context`: Optional JSON containing additional information about the context in which the launch was generated (e.g., course name, module, unit). There is no specific format for this JSON, and any provided fields will be stored as metadata for this launch instance.

* `email`: Email address of the Platform (LMS) user.

* `given_name`: First name of the Platform (LMS) user.

* `family_name`: Last name of the Platform (LMS) user.

* `roles`: A list of strings containing the roles of the Platform (LMS) user. Any string is valid. Roles must be mapped to DataWars' roles as part of the account integration.

* `licenses`: If one or more licenses need to be assigned to the authenticated user, include the license slugs in this list. If you are unsure which license to use for each integration, contact your account manager.

* `teams`: Users within the account might be grouped into teams. A user can belong to zero, one, or many teams within the account. If you want to include the authenticated user in a specific team, include the team name in this list. If the provided team does not already exist, it will be created as part of the launch process.


### Response

Upon success, the API will return a `200 OK` status code, and the response will look like this:

```json
{
    "url": "https://api.dev.namespace.im/api/v1/launch/auth?token=2a42bebd-37cf-4e8d-806e-c47551169c1d&next=https%3A%2F%2Fapp.dev.namespace.im%2Fproject%2Fc7a2d7b8-c8a0-4834-b757-d0e21ed3e788"
}
```

The `url` in the response points to the `/api/v1/launch/auth` endpoint, which will trigger the user’s redirect flow, authenticating them and forwarding them to the desired lab. The `token` parameter in the URL is a short-lived token that can be used once to exchange for a valid session cookie. The `next` parameter is the destination URL (URL-encoded) where the user will be redirected.

### Errors

`400 BAD REQUEST` will be returned in case any of the required field in payload are missing. Example:

```json
{"error": "Must provide `user_id` and `content_id` in payload."}
```

`500 INTERNAL SERVER ERROR` will be returned if something fails on the server side. A detailed error message will be included in the response. Example:

```json
{"error": "Some error message"}
```