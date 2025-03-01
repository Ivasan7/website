---
id: best
title: "Best Practices"
---

The following guide is a list of the best practices collected and that we usually recommend to all users. Do not take this guide as mandatory, you might pick some of them according your needs.

**Feel free to suggest your best practices to the Verdaccio community**.

## Private Registry

Vous pouvez ajouter des utilisateurs et gérer quels utilisateurs peuvent accéder à quels paquets.

It is recommended that you define a prefix for your private packages, for example `local-*` or scoped `@my-company/*`, so all your private things will look like this: `local-foo`. De cette façon, vous pouvez clairement séparer les paquets publiques et privés.

    yaml
      packages:
        '@my-company/*':
          access: $all
          publish: $authenticated
         'local-*':
          access: $all
          publish: $authenticated
        '@*/*':
          access: $all
          publish: $authenticated
        '**':
          access: $all
          publish: $authenticated

Always remember, **the order of packages access is important**, packages are matched always top to bottom.

### Utilisation des paquets publics à partir de npmjs.org

Si un paquet n'existe pas dans l'archive, le serveur essaiera de le récupérer à partir de npmjs.org. Si npmjs.org ne fonctionne pas, il ne fournira les paquets mis en cache que s'il n'y avait pas d'autres paquets. **Verdaccio will download only what's needed (requested by clients)**, and this information will be cached, so if client will ask the same thing second time, it can be served without asking npmjs.org for it.

**Example:**

If you successfully request `express@4.0.1` from this server once, you'll be able to do it again (with all it's dependencies) anytime even if npmjs.org is down. But say `express@4.0.0` will not be downloaded until it's actually needed by somebody. And if npmjs.org is offline, this server would say that only `express@4.0.1` (only what's in the cache) is published, but nothing else.

### Annuler les paquets publiques

If you want to use a modified version of some public package `foo`, you can just publish it to your local server, so when your type `npm install foo`, **it'll consider installing your version**.

Il y a deux options ici:

1. You want to create a separate **fork** and stop synchronizing with public version.
    
    Si vous voulez faire cela, vous devriez modifier votre fichier de configuration pour que verdaccio ne fasse plus de demande à propos de ce paquet pour npjms. Add a separate entry for this package to `config.yaml` and remove `npmjs` from `proxy` list and restart the server.
    
    ```yaml
    packages:
      '@my-company/*':
        access: $all
        publish: $authenticated
        # comment it out or leave it empty
        # proxy:
    ```
    
    When you publish your package locally, **you should probably start with version string higher than existing one**, so it won't conflict with existing package in the cache.

2. Vous souhaitez utiliser temporairement votre propre version, mais revenir à la version public dès sa mise à jour.
    
    In order to avoid version conflicts, **you should use a custom pre-release suffix of the next patch version**. For example, if a public package has version 0.1.2, you can upload `0.1.3-my-temp-fix`.
    
    ```bash
    npm version 0.1.3-my-temp-fix
    npm publish --tag fix --registry http://localhost:4873
    ```
    
    This way your package will be used until its original maintainer updates his public package to `0.1.3`.

## Security

The security starts in your environment, for such thing we totally recommend read **[10 npm Security Best Practices](https://snyk.io/blog/ten-npm-security-best-practices/)** and follow the recommendation.

### Paquet d'accès

By default all packages are you publish in Verdaccio are accessible for all public, we totally recommend protect your registry from external non authorized users updating `access` property to `$authenticated`.

```yaml
  packages:
    '@my-company/*':
      access: $authenticated
      publish: $authenticated
    '@*/*':
      access: $authenticated
      publish: $authenticated
    '**':
      access: $authenticated
      publish: $authenticated
   ```

That way, **nobody will take advantage of your registry unless it's authorized and private packages won't be displayed in the User Interface**.

## Server

### Secured Connections

Using **HTTPS** is a common recomendation, for such reason we recommend read the [SSL](ssl.md) section to make Verdaccio secure or using a HTTPS [reverse proxy](reverse-proxy.md) on top of Verdaccio.

### Expiring Tokens

In `verdaccio@3.x` the tokens have no expiration date. For such reason we introduced in the next `verdaccio@4.x` the JWT feature [PR#896](https://github.com/verdaccio/verdaccio/pull/896)

```yaml
security:
  api:
    jwt:
      sign:
        expiresIn: 15d
        notBefore: 0
  web:
    sign:
      expiresIn: 7d
```

**Using this configuration will override the current system and you will be able to control how long the token will live**.

Using JWT also improves the performance with authentication plugins, the old system will perform an unpackage and validating the credentials in each request, while JWT will rely on the token signature avoiding the overhead for the plugin.

As a side note, at **npmjs the token never expires**.