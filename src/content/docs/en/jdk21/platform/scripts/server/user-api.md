---
title: "User API"
description: "User listing, active sessions, and authentication attempts"
sidebar:
  order: 10
---

User API queries users and session state on the platform. Information about the **current** user is reached not via `ins.*` but through the `user` global in the script context.

## `ins.getAllUsernames()`

Returns a collection of every username in the system (`Collection<String>`).

```javascript
var users = ins.getAllUsernames();
users.forEach(function(u) {
    ins.consoleLog(u);
});
```

## `ins.getLoggedInUsers()`

Returns the collection of users with an active session.

```javascript
var online = ins.getLoggedInUsers();
ins.consoleLog("Active users: " + online.size());
```

## `ins.getLastAuthAttempts()`

Returns recent authentication attempts (`AuthAttemptDto`) — both successful and failed.

```javascript
var attempts = ins.getLastAuthAttempts();
attempts.forEach(function(a) {
    ins.consoleLog(a.username + " @ " + a.remoteAddress + " → " + (a.success ? "OK" : "FAIL"));
});
```

Typical `AuthAttemptDto` fields: `username`, `remoteAddress`, `success`, `createdAt`.
