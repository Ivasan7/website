---
id: webui
title: "Interfaz Web de Usuario"
---

![Uplinks](https://user-images.githubusercontent.com/558752/52916111-fa4ba980-32db-11e9-8a64-f4e06eb920b3.png)

Verdaccio ofrece un interfaz web de usuario para mostrar solo los paquetes privados y puede ser personalizable.

```yaml
web:
  enable: true
  title: Verdaccio
  logo: logo.png
  primary_color: "#4b5e40"
  gravatar: true | false
  scope: "@scope"
  sort_packages: asc | desc
  darkMode: false
```

Todo los accesos restringidos definidos para [proteger paquetes](protect-your-dependencies.md) también aplican al interfaz web.

> The `primary_color` and `scope` must be wrapped by quotes: eg: ('#000000' or "#000000")

The `primary_color` **must be a valid hex representation**.

### Internationalization

*Since v4.5.0*, there are translations available

```yaml
i18n:
  web: en-US
```

> ⚠️ Only the languages in this [list](https://github.com/verdaccio/ui/tree/master/i18n/translations) are available, feel free to contribute with more. The default one is en-US

### Configuración

| Propiedad     | Tipo       | Requerido | Ejemplo                                                       | Soporte       | Descripcion                                                                                                              |
| ------------- | ---------- | --------- | ------------------------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------ |
| enable        | boolean    | No        | true/false                                                    | all           | habilita la interfaz web                                                                                                 |
| title         | string     | No        | Verdaccio                                                     | all           | El título de la interfaz web                                                                                             |
| gravatar      | boolean    | No        | true                                                          | `>v4`      | Gravatars will be generated under the hood if this property is enabled                                                   |
| sort_packages | [asc,desc] | No        | asc                                                           | `>v4`      | By default private packages are sorted by ascending                                                                      |
| logo          | string     | No        | `/local/path/to/my/logo.png` `http://my.logo.domain/logo.png` | all           | a URI where logo is located (header logo)                                                                                |
| primary_color | string     | No        | "#4b5e40"                                                     | `>4`       | The primary color to use throughout the UI (header, etc)                                                                 |
| scope         | string     | No        | @myscope                                                      | `>v3.x`    | If you're using this registry for a specific module scope, specify that scope to set it in the webui instructions header |
| darkMode      | boolean    | No        | false                                                         | `>=v4.6.0` | This mode is an special theme for those want to live in the dark side                                                    |

> It is recommended the logo size has the following size `40x40` pixels.
> 
> The `darkMode` can be enabled via UI and is persisted in the local storage, furthermore, also void `primary_color` and dark cannot be customized.