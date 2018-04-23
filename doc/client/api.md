# Client Daemon API

## Conventions and Notation

1. All endpoints using the `GET` and `DELETE` methods accept input in URL parameters.
2. All other endpoints accept parameters encoded as JSON in the body of the request.
3. Data types are expressed using TypeScript's notation for [basic](https://www.typescriptlang.org/docs/handbook/basic-types.html) and [composite](https://www.typescriptlang.org/docs/handbook/advanced-types.html) types.
4. Complex data types like `Target` and `Module` may be defined in [doc/client/api-types.md](https://github.com/mozilla/mig/blob/client-daemon/doc/client/api-types.md).
5. URLs may contain positional parameter for identifiers, which are always strings. E.g. in `/actions/:id/status`, `:id` is a positional parameter.


## Endpoint Table of Contents

* [Retrieve top-level API documentation](https://github.com/mozilla/mig/blob/client-daemon/doc/client/api.md#endpoint-documentation)
* [Retrieve documentation for a module](https://github.com/mozilla/mig/blob/client-daemon/doc/client/api.md#retrieve-documentation-for-a-module)
* [Create an action](https://github.com/mozilla/mig/blob/client-daemon/doc/client/api.md#create-an-action)
* [Dispatch an action](https://github.com/mozilla/mig/blob/client-daemon/doc/client/api.md#dispatch-an-action)
* [Check the status of a dispatched action](https://github.com/mozilla/mig/blob/client-daemon/doc/client/api.md#check-the-status-of-a-dispatched-action)

## Endpoint Documentation

### Retrieve top-level API documentation

```
GET /v1/documentation
```

Retrieve a JSON document describing all of the endpoints supported by the API.

#### Parameters

None

#### Response

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| currentVersion | string | The most recent version of the API. | "v2" |
| versions | `[]{ version: string, endpoints: []Endpoint }` | An array of objects containing descriptions of endpoints exposed by each version of the API | See below |

##### Example Response

##### Status Codes

* `200` indicates that the request was handled successfully.

### Retrieve documentation for a module

```
GET /v1/module/:name/documentation
```

Retrieve a JSON document describing the configuration accepted by a module
and what it does.

#### Parameters

The `name` positional parameter in the URL must be the name of the module to retrieve
documentation for as a string. E.g. `pkg`, `file`, etc.

#### Response

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| module | string | The name of the module | "pkg" |
| version | string | The semantic version of the mdoule | "v1.2.3" |
| description | string | An explanation of what the module does | "Find packages on the host" |
| configuration | Module | An object describing the configuration parameters | See below |

##### Example Response

##### Status Codes

* `200` indicates that the response was served successfully.
* `400` indicates that the module requested does not exist.

### Create an action

```
POST /v1/actions/create
```

This endpoint can be invoked to create an action.
The action will not be dispatched to the MIG API to be executed by agents right away.
Instead, the action will be retained by the daemon, and its ID will be returned.
This allows investigators to review and modify actions before dispatching them.

#### Parameters

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| module | string | The name of the module the action should invoke. | "pkg" |
| expireAfter | number | The number of seconds after which the action should expire. | 300 |
| target | Target | A description of the agents to have run the action. | [Target example]() |
| moduleConfig | Module | An object providing configuration values for the module specified. | [Module example]() |

#### Response

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| error | `string | null` | If the action cannot be created, an error will be returned. | "Invalid module configuration." |
| action | string | An identifier for the newly-created action. | "abc123...def" |


##### Status Codes

* `200` indicates that the action was created successfully.
* `400` indicates that some data provided as a parameter was incorrectly formatted or otherwise invalid.
* `500` indicates that the daemon encountered an error due to an internal failure.

#### Example Request

#### Example Response

### Dispatch an action

```
PUT /v1/actions/dispatch
```

After an action has been created, this endpoint can be invoked to dispatch that action to the MIG API.

#### Parameters

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| action | string | The identifier of an action to dispatch. | "abc123...def" |

#### Response

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| error | `string | null` | If the action cannot be dispatched, an error will be returned. | "Action has expired." |
| status | string | The new status of the action. | "dispatched" |

##### Status Codes

* `200` indicates that the action was dispatched to the MIG API successfully.
* `400` indicates that the action identifier provided was determined to be invalid.
* `500` indicates that the action could not be dispatched due to an internal failure.

#### Example Request

#### Example Response

### Check the status of a dispatched action

```
GET /v1/actions/:id/status
```

Retrieve the status of an action.

#### Parameters

The only parameter is the `:id` positional parameter in the endpoint URL.

#### Response

| Name | Type | Description | Example |
| ---- | ---- | ----------- | ------- |
| error | `string | null` | If the action's status cannot be retrieved, an error will be returned. | null |
| status | string | The status of the action. | "in-progress" |
| agentsTargeted | number | The number agents that the action will be scheduled for | 32 |
| sent | number | The number of agents to whom the action has been sent | 10 |
| done | number | The number of agents that have finished running the action | 3 |
| succeeded | number | The number of agents that successfully finished running the action | 2 |