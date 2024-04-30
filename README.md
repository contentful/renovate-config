[![Renovate Config Validator](https://github.com/contentful/renovate-config/actions/workflows/renovate-config-validator.yml/badge.svg)](https://github.com/contentful/renovate-config/actions/workflows/renovate-config-validator.yml)

# renovate-config

Configuration presets for renovate in Contentful

## Encrypting Tokens

Secrets should be encrypted prior to storing them in this repository. You can use the [Renovate App secrets encryption tool](https://app.renovatebot.com/encrypt) to do this.

You should specify `contentful` as the organization and leave the repository field empty.

You can now store the encrypted token inside a wrapped block e.g.

```json
{
  "encrypted": {
    "<fieldName>": "wcFMA/..."
  }
}
```

You should substitute `<fieldName>` with the appropriate key based on your package manager. This might be `"token"` `"npmToken"` etc.
