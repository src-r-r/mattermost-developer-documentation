---
title: "Bindings"
heading: "Bindings"
description: "TODO"
weight: 400
---

Bindings
([godoc](https://pkg.go.dev/github.com/mattermost/mattermost-plugin-apps/apps#Binding))
tell the Mattermost Client what UI elements to display for the app, and what to
do if the "feature" is interacted with. To display anything in the Client the
app needs to handle the bindings call. When it is invoked, your app needs to
provide the list of bindings available according to the context
([godoc](https://pkg.go.dev/github.com/mattermost/mattermost-plugin-apps/apps#Context)).
Some fields included in the context:

- Your app's bot user access token
- The Mattermost Site URL
- The ID of the user requesting the bindings (acting user ID)
- The ID of the team the user is currently focused on
- The ID of the channel the user is currently focused on
- The ID of the post the user is currently focused on (if applicable)

**Note:** Bindings are fetched (and refreshed) on every channel switch. When the user moves to a different context (like opening a thread, or a post in a search view) new bindings may be fetched to provide the correct bindings for the thread/post context. Bindings are also fetched when an OAuth2 process is completed and when the application gets installed.

The expected response should include the following:

| Name   | Type | Description           |
| :----- | :------- | :-------------------- |
| `data` | Bindings | The list of (top-level) bindings and their sub-bindings. |

Each Binding must provide one and only one of:
| Name   | Type | Description           |
| :----- | :------- | :-------------------- |
| `submit` | `Call` |  The call to perform if the Binding is invoked by the user. |
| `form` | `Form` | Modal form to open.  |
<!-- TODO: "open_as" -->
| `bindings` | `Bindings` | Sub-bindings, mostly used for subcommands.  |

Bindings are organized by top level locations. Top level bindings just need to define:

| Name       | Type     | Description                             |
| :--------- | :------- | :-------------------------------------- |
| `location` | string   | Top level location.                     |
| `bindings` | Bindings | A list of bindings under this location. |
<!-- TODO: how to customise top-level /-command -->
<!-- TODO: defaulting for label, location -->

`/in_post` bindings don't need to be defined in this call.

### `/post_menu` bindings

| Name       | Type   | Description                                                                                                       |
| :--------- | :----- | :---------------------------------------------------------------------------------------------------------------- |
| `location` | string | Name of this location. The whole path of locations will be added in the context. Must be unique in its level.     |
| `icon`     | string | (Optional) Either a fully-qualified URL, or a path for an app's static asset.                                     |
| `label`    | string | (Optional) Text to show in the item. Defaults to location. Must be unique in its level.                           |

The call for these bindings will include in the context the user ID, the post ID, the root post ID if any, the channel ID, and the team ID.

### `/channel_header` bindings

| Name       | Type   | Description                                                                                                                 |
| :--------- | :----- | :-------------------------------------------------------------------------------------------------------------------------- |
| `location` | string | Name of this location. The whole path of locations will be added in the context. Must be unique in its level.               |
| `icon`     | string | (Optional/Web App required) Either a fully-qualified URL, or a path for an app's static asset.                              |
| `label`    | string | (Optional) Text to show in the item on mobile and webapp collapsed view. Defaults to location. Must be unique in its level. |
| `hint`     | string | (Optional/Web App required) Text to show in tooltip.                                                                        |

The context of the call for these bindings will include the user ID, the channel ID, and the team ID.

### `/command` bindings

For commands we can distinguish between leaf commands (executable subcommand) and partial commands.

A partial command must include:

| Name          | Type     | Description                                                                                                               |
| :------------ | :------- | :------------------------------------------------------------------------------------------------------------------------ |
| `location`    | string   | Name of this location. The whole path of locations will be added in the context. Must be unique in its level.             |
| `label`       | string   | The label to use to define the command. Cannot include spaces or tabs. Defaults to location. Must be unique in its level. |
| `hint`        | string   | (Optional) Hint line on command autocomplete.                                                                             |
| `description` | string   | (Optional) Description line on command autocomplete.                                                                      |
| `bindings`    | Bindings | List of subcommands.                                                                                                      |

A leaf command must include either `submit` or a `form`, and the following fields:

| Name          | Type   | Description                                                                                                                                  |
| :------------ | :----- | :------------------------------------------------------------------------------------------------------------------------------------------- |
| `location`    | string | Name of this location. The whole path of locations will be added in the context. Must be unique in its level.                                |
| `label`       | string | The label to use to define the command. Cannot include spaces or tabs. Defaults to location. Must be unique in its level.                    |
| `hint`        | string | (Optional) Hint line on command autocomplete.                                                                                                |
| `description` | string | (Optional) Description line on command autocomplete.                                                                                         |

The context of the call for these bindings will include the user ID, the post ID, the root post ID (if any), the channel ID, and the team ID. It will also include the raw command.

## Example data flow

<details><summary>Client Bindings Request</summary>

`GET /plugins/com.mattermost.apps/api/v1/bindings?user_id=ws4o4macctyn5ko8uhkkxmgfur&channel_id=qphz13bzbf8c7j778tdnaw3huc&scope=webapp`

</details>

<details><summary>Mattermost Bindings Request</summary>

`POST /plugins/com.mattermost.apps/example/hello/bindings`

```json
{
    "path": "/bindings",
    "context": {
        "app_id": "helloworld",
        "bot_user_id": "i4wzxbk1hbbufq8rnecso96oxr",
        "acting_user_id": "81bqom3kjjbo7bcjcnzs6dc8uh",
        "user_id": "81bqom3kjjbo7bcjcnzs6dc8uh",
        "team_id": "",
        "channel_id": "ytqokpzzcinszf7ywrbdfitusw",
        "mattermost_site_url": "http://localhost:8065",
        "user_agent": "webapp",
        "bot_access_token": "gcn6r3ac178zbxwiw5pc38e8zc"
    }
}
```
</details>

<details><summary>App Binding Response</summary>

```json
{
    "type": "ok",
    "data": [
        {
            "location": "/channel_header",
            "bindings": [
                {
                    "location": "send-button",
                    "icon": "icon.png",
                    "label": "send hello message",
                    "form": {
                        "--form--": "--definition--"
                    }
                }
            ]
        },
        {
            "location": "/post_menu",
            "bindings": [
                {
                    "location": "send-button",
                    "icon": "icon.png",
                    "label": "send hello message",
                    "submit": {
                        "path": "/send",
                        "expand": {
                            "post": "all"
                        }
                    }
                }
            ]
        },
        {
            "location": "/command",
            "bindings": [
                {
                    "icon": "icon.png",
                    "description": "Hello World app",
                    "hint": "[send]",
                    "bindings": [
                        {
                            "location": "send",
                            "label": "send",
                            "submit": {
                                "path": "/send-modal"
                            }
                        }
                    ]
                }
            ]
        }
    ]
}
```
</details>
