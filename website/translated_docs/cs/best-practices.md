---
id: osvědčené
title: "Osvědčené postupy"
---

Následující příručka obsahuje seznam nejlepších praktických postupů, které obvykle doporučujeme všem uživatelům. Neberte tuto příručku jako povinnou, vyberte si podle svých potřeb.

**Feel free to suggest your best practices to the Verdaccio community**.

## Soukromý registr

You can add users and manage which users can access which packages.

Doporučujeme, abyste definovali předponu pro vaše soukromé balíčky, například `local-*` nebo `@my-company/*`, takže všechny vaše soukromé balíčky budou vypadat takto: `local-foo`. This way you can clearly separate public packages from private ones.

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

### Using public packages from npmjs.org

If some package doesn't exist in the storage, server will try to fetch it from npmjs.org. If npmjs.org is down, it serves packages from cache pretending that no other packages exist. **Verdaccio will download only what's needed (requested by clients)**, and this information will be cached, so if client will ask the same thing second time, it can be served without asking npmjs.org for it.

**Příklad:**

If you successfully request `express@4.0.1` from this server once, you'll be able to do it again (with all it's dependencies) anytime even if npmjs.org is down. Ale např. `express@4.0. ` nebude staženo, dokud ho někdo nepotřebuje. And if npmjs.org is offline, this server would say that only `express@4.0.1` (only what's in the cache) is published, but nothing else.

### Override public packages

Chcete-li použít upravenou verzi nějakého veřejného balíčku `foo`, můžete jej publikovat pouze na místní server, takže když spustíte `npm install foo`, **bude stažena Vámi vytvořená verze**.

There's two options here:

1. Chcete vytvořit samostatý **fork** a zastavit synchronizaci s veřejnou verzí.
    
    If you want to do that, you should modify your configuration file so verdaccio won't make requests regarding this package to npmjs anymore. Přidejte do `config.yaml` samostatnou položku pro tento balíček a odeberte `npmjs` ze seznamu `proxy` a restartujte server.
    
    ```yaml
    packages:
      '@my-company/*':
        access: $all
        publish: $authenticated
        # zakomentujte nebo ponechte prázdné
        # proxy:
    ```
    
    Když svůj balíček publikujete lokálně, **měli byste pravděpodobně začít s řetězcem verzí vyšším, než je stávající **, takže nebude v konfliktu s existujícím balíčkem ve vyrovnávací paměti.

2. You want to temporarily use your version, but return to public one as soon as it's updated.
    
    Chcete-li se vyhnout konfliktům verzí, **měli byste použít vlastní příponu předběžného vydání další verze opravy**. Pokud má například veřejný balíček verzi 0.1.2, můžete nahrát `0.1.3-moje-docasna-oprava`.
    
    ```bash
    npm version 0.1.3-my-temp-fix
    npm publish --tag fix --registry http://localhost:4873
    ```
    
    Tímto způsobem bude váš balíček používán, dokud jeho původní správce nezmění svůj veřejný balíček na `0.1.3`.

## Bezpečnost

The security starts in your environment, for such thing we totally recommend read **[10 npm Security Best Practices](https://snyk.io/blog/ten-npm-security-best-practices/)** and follow the recommendation.

### Přístup k balíčkům

Ve výchozím nastavení jsou všechny balíčky, které publikujete ve Verdaccio, přístupné všem uživatelům. Doporučujeme chránit registr před externími neoprávněnými uživateli aktualizací vlastnost `access` na `$authenticated`.

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

### Zabezpečená připojení

Používání ** HTTPS ** je běžným doporučením, z tohoto důvodu doporučujeme přečíst si sekci [SSL](ssl.md), abyste mohli Verdaccio zabezpečit nebo používat HTTPS[reverse proxy](reverse-proxy.md) nad Verdaccio.

### Platnost tokenů

Ve `verdaccio@3.x` nemají tokeny datum vypršení platnosti. Z tohoto důvodu jsme přidali v další verzi "verdaccio@4.x" funkci JWT [PR#896](https://github.com/verdaccio/verdaccio/pull/896)

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

**Použití této konfigurace přepíše současný systém a budete moci řídit, jak dlouho bude token platný**.

Použití JWT také zlepšuje výkon s autentizačními pluginy, starý systém bude provádět rozbalování a ověřování pověření v každém požadavku, zatímco JWT bude spoléhat na podpis tokenu, který se vyhne režii pro plugin.

Mimo jiné, v **npmjs token nikdy nevyprší**.