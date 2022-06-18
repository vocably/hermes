# @vocably/hermes

A typesafe and promise-based wrapper for `chrome.runtime.sendMessage`, `chrome.runtime.onMessage.addListener`, and `chrome.runtime.onMessageExternal.addListener`.

## API

The `@vocably/hermes` exports 2 methods: `createMessage` and `createExternalMessage`.

### `createMessage`

`createMessage` could be used for messaging between content script and service worker when extension ID is not necessary.

```ts
const [messageSender, messageSubscriber] = createMessage<
  PayloadType,
  ResponseType
>('messageIdentifier');
```

`messageSender` function expects `PayloadType` as the only param. `PayloadType` could be `void`. Returns `Promise<ResponseType>`;

`messageSubscriber` function should be used in Service Worker to receive data of `PayloadType`, asynchronously process it, and return value of `ResponseType`.

### `createExternalMessage`

`createExternalMessage` could be used for messaging between `externally_connectable` URLs and service worker when extension ID is necessary.

```ts
const [messageSender, messageSubscriber] = createExternalMessage<
  PayloadType,
  ResponseType
>('messageIdentifier');
```

`messageSender` function expects `extensionId: string` as the first parameter and `PayloadType` as the second. `PayloadType` could be `void`. Returns `Promise<ResponseType>`;

`messageSubscriber` function should be used in Service Worker to receive data of `PayloadType`, asynchronously process it, and return value of `ResponseType`.

## Comprehensive Example

Let's imagine we need to create a user, pass it to a Service Worker and wait for the response from the Service Worker.

```ts
// save-user.ts

import { createMessage } from '@vocably/hermes';

type User = {
  id?: string;
  name: string;
  email: string;
};

type SaveUserResponse = {
  id: string;
};

export const [saveUser, onSaveUserRequest] = createMessage<
  User,
  SaveUserResponse
>('saveUser');
```

```ts
// content-script.ts

import { saveUser } from './save-user.ts';

saveUser({
  name: 'Speedy Gonzales',
  email: 'speedy@disney.com',
}).then(({ id }) => {
  console.log(`The saved user ID is ${id}`);
});
```

```ts
// sevice-worker.ts

import { onSaveUserRequest } from './save-user.ts';

onSaveUserRequest(async (sendResponse, user) => {
  let id: string;

  if (user.id) {
    // api.updateUser should be implemmented
    // individually. It's not a part of @vocably/hermes
    await api.updateUser(user);
    id = user.id;
  } else {
    // api.createUser should be implemmented
    // individually. It's not a part of @vocably/hermes
    id = await api.createUser(user);
  }

  // The callback function must return sendResponse() with the message return value passed into it.
  return sendResponse({
    id,
  });
});
```
