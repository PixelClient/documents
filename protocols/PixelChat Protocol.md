# PixelChat Protocol

> [!WARNING]
> This service is in early stages of development and might be discontinued at any time without notice.

The default endpoint is located at `wss://api.pixelclient.xyz/chat`.

Our backend does not produce any logs when connecting or using this service.
The only metrics visible to us are recorded by [Cloudflare](<https://www.cloudflare.com/privacypolicy/>). These are very generic and only show traffic and geographic metrics.

## Client to Server

### 1. Create identity

```json
{
  "type": "identity",
  "username": "real_person"
}
```

`username` must match `^[a-zA-Z0-9_]{3,16}$`. Identity can only be set once per connection. Further attempts are rejected.


### 2. Join group

#### Create new group

```json
{
  "type": "group"
}
```

#### Join existing group

```json
{
  "type": "group",
  "id": "abcdef"
}
```

Requires an identity to already be set. A client can only be in one group per connection.

### 3. Send message

```json
{
  "type": "message",
  "content": "your chat message!"
}
```

`content` must be 1-256 characters and cannot contain control characters, `\r`, or `\n`. Requires an identity and an active group.

## Server to Client

### Error message

```json
{
  "error": "ERROR_ENUM",
  "message": "Useful error message"
}
```

| Enum              | Description                                                                                                                                                            |
| :---              | :---                                                                                                                                                                   |
| `INVALID_FORMAT`  | The incoming message didn't match any accepted message format. This could be due to malformed JSON, wrong field types, or a field that didn't pass validation.         |
| `NO_IDENTITY`     | The client tried to create/join a group or send a message before assigning a username via `identity`                                                                   |
| `INVALID_STATE`   | The client tried to do something that conflicts with its current state: setting an identity when one is already set, or creating/joining a group while already in one. |
| `GROUP_NOT_FOUND` | The client tried to join a group that doesn't exist.                                                                                                                   |
| `USERNAME_TAKEN`  | The client tried to join a group with an already taken username.                                                                                                       |
| `NO_GROUP`        | The client tried to send a message before creating or joining a group.                                                                                                 |

### Identity confirmation

```json
{
  "type": "identity",
  "username": "real_person"
}
```

Sent back after a successful identity request.

### Group information

```json
{
  "type": "group",
  "id": "abcdef"
}
```

Informs the client of the group it is in. Sent after a `group` packet is received by the client in a valid state.

### Chat message

#### User message

```json
{
  "type": "message",
  "source": "USER",
  "name": "cool_username",
  "content": "my username is so cool!"
}
```

#### System message

```json
{
  "type": "message",
  "source": "SYSTEM",
  "content": "system announcement message"
}
```
