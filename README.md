# Azure Notification Hubs for Go(lang)

This library provides a Go module for Microsoft Azure Notification Hubs.

Originally a fork from [Gozure](https://github.com/onefootball/gozure) with patches
from [Martin Etnestad](https://github.com/gnawybol) @ [vippsas](https://github.com/vippsas/gozure).

Now maintained and packaged by [Daresay AB](https://daresay.co), [@daresaydigital](https://github.com/daresaydigital).

Basically a wrapper for this [Rest API](https://docs.microsoft.com/en-us/rest/api/notificationhubs/rest-api-methods)

[![Build Status](https://travis-ci.org/daresaydigital/azure-notificationhubs-go.svg?branch=master)](https://travis-ci.org/daresaydigital/azure-notificationhubs-go)
[![Go](https://github.com/daresaydigital/azure-notificationhubs-go/workflows/Go/badge.svg?branch=master)](https://github.com/daresaydigital/azure-notificationhubs-go/actions)

## Installing

Using go mod

```sh
require ithub.com/daresaydigital/azure-notificationhubs-go v0.1.4

replace github.com/daresaydigital/azure-notificationhubs-go => github.com/tangramaiot/azure-notificationhubs-go v0.0.0-20240723010036-e33978f4f3ec
```

## External dependencies

No external dependencies

## Registering device

```go
package main

import (
  "context"
  "strings"
  "github.com/daresaydigital/azure-notificationhubs-go"
)

func main() {
  var (
    hub      = notificationhubs.NewNotificationHub("YOUR_DefaultFullSharedAccessConnectionString", "YOUR_HubPath")
    template = `{
    "aps":{
      "alert":{
        "title":"$(title)",
        "body":"$(body)",
      },
      "badge":"#(badge)",
      "topic":"co.daresay.app",
      "content-available": 1
    },
    "name1":"$(value1)",
    "name2":"$(value2)"
  }`
  )

  template = strings.ReplaceAll(template, "\n", "")
  template = strings.ReplaceAll(template, "\t", "")

  reg := notificationhubs.NewTemplateRegistration(
    "ABC123",                       // The token from Apple or Google
    nil,                            // Expiration time, probably endless
    "ZXCVQWE",                      // Registration id, if you want to update an existing registration
    "tag1,tag2",                    // Tags that matches this device
    notificationhubs.ApplePlatform, // or GcmPlatform for Android
    template,                       // The template. Use "$(name)" for strings and "#(name)" for numbers
  )

  // or hub.NewRegistration( ... ) without template

  hub.RegisterWithTemplate(context.TODO(), *reg)
  // or if no template:
  hub.Register(context.TODO(), *reg)
}
```

## Sending notification

```go
package main

import (
  "context"
  "fmt"
  "github.com/daresaydigital/azure-notificationhubs-go"
)

func main() {
  var (
    hub     = notificationhubs.NewNotificationHub("YOUR_DefaultFullSharedAccessConnectionString", "YOUR_HubPath")
    payload = []byte(`{"title": "Hello Hub!"}`)
    n, _    = notificationhubs.NewNotification(notificationhubs.Template, payload)
  )

  // Broadcast push
  b, _, err := hub.Send(context.TODO(), n, nil)
  if err != nil {
    panic(err)
  }

  fmt.Println("Message successfully created:", string(b))

  // Tag category push
  tags := "tag1 || tag2"
  b, _, err = hub.Send(context.TODO(), n, &tags)
  if err != nil {
    panic(err)
  }

  fmt.Println("Message successfully created:", string(b))
}
```

## Tag expressions

Read more about how to segment notification receivers in [the official documentation](https://docs.microsoft.com/en-us/azure/notification-hubs/notification-hubs-tags-segment-push-message).

### Example expressions

Example devices:

```json
"devices": {
  "A": {
    "tags": [
      "tag1",
      "tag2"
    ]
  },
  "B": {
    "tags": [
      "tag2",
      "tag3"
    ]
  },
  "C": {
    "tags": [
      "tag1",
      "tag2",
      "tag3"
    ]
  },
}
```

- Send to devices that has `tag1` or `tag2`. Example devices A, B and C.

  ```go
  hub.Send(notification, "tag1 || tag2")
  ```

- Send to devices that has `tag1` and `tag2`. Device A and C.

  ```go
  hub.Send(notification, "tag1 && tag2")
  ```

- Send to devices that has `tag1` and `tag2` but not `tag3`. Device A.

  ```go
  hub.Send(notification, "tag1 && tag2 && !tag3")
  ```

- Send to devices that has not `tag1`. Device B.

  ```go
  hub.Send(notification, "!tag1")
  ```

## Changelog


## TODO

- Implement cancel scheduled notifications using http DELETE.
  [Find inspo from the Java SDK here.](https://github.com/Azure/azure-notificationhubs-java-backend/blob/d293da9db7564dfd2800e45899f0e2425f669c6e/NotificationHubs/src/com/windowsazure/messaging/NotificationHub.java#L646)

- Only Android and iOS is supported today, implement the other supported platforms. Probably limited usecase.

## License

See the [LICENSE](LICENSE.txt) file for license rights and limitations (MIT).
