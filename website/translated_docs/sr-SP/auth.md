---
id: authentication
title: "Authentication"
---

The authentication is tied to the auth [plugin](plugins.md) you are using. The package restrictions are also handled by the [Package Access](packages.md).

The client authentication is handled by `npm` client itself. Once you login to the application:

```bash
npm adduser --registry http://localhost:4873
```

Токен се генерише у фајлу за конфигурацију `npm` који се налази у home фолдеру корисника. Како бисте сазнали више о `.npmrc` прочитајте [official documentation](https://docs.npmjs.com/files/npmrc).

```bash
cat .npmrc
registry=http://localhost:5555/
//localhost:5555/:_authToken="secretVerdaccioToken"
//registry.npmjs.org/:_authToken=secretNpmjsToken
```

#### Анонимно публиковање

`verdaccio` allows you to enable anonymous publish, to achieve that you will need to set up correctly your [packages access](packages.md).

Пример:

```yaml
  'my-company-*':
    access: $anonymous
    publish: $anonymous
    proxy: npmjs
```

Као што је описано, [on issue #212](https://github.com/verdaccio/verdaccio/issues/212#issuecomment-308578500) све док `npm@5.3.0` и све верзије не буду усаглашене **неће Вам бити омогућено да публикујете без токена**.

## Understanding Groups

### The meaning of `$all` and `$anonymous`

As you know *Verdaccio* uses the `htpasswd` by default. That plugin does not implement the methods `allow_access`, `allow_publish` and `allow_unpublish`. Thus, *Verdaccio* will handle that in the following way:

* If you are not logged in (you are anonymous), `$all` and `$anonymous` means exactly the same.
* If you are logged in, `$anonymous` won't be part of your groups and `$all` will match any logged user. A new group `$authenticated` will be added to the list.

As a takeaway, `$all` **will match all users, independently whether is logged or not**.

**The previous behavior only applies to the default authentication plugin**. If you are using a custom plugin and such plugin implements `allow_access`, `allow_publish` or `allow_unpublish`, the resolution of the access depends on the plugin itself. Verdaccio will only set the default groups.

Let's recap:

* **logged**: `$all`, `$authenticated`, + groups added by the plugin
* **anonymous (logged out)**: `$all` and `$anonymous`.

## Подразумевана htpasswd

In order to simplify the setup, `verdaccio` uses a plugin based on `htpasswd`. Since version v3.0.x the `verdaccio-htpasswd` plugin is used by default.

```yaml
auth:
  htpasswd:
    file: ./htpasswd
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    #max_users: 1000
```

| Својство  | Тип    | Неопходно | Пример     | Подршка | Опис                                   |
| --------- | ------ | --------- | ---------- | ------- | -------------------------------------- |
| file      | string | Да        | ./htpasswd | all     | фајл који садржи шифроване credentials |
| max_users | number | Не        | 1000       | all     | подешава максимални број корисника     |

In case you decide to not allow users to sign up, you can set `max_users: -1`.