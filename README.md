# DNS-01 Exec Plugins

To support multiple DNS providers the `kube-cert-manager` supports an exec based plugin system.

## Overview

DNS-01 plugins are responsible for creating or deleting a single DNS TXT record for a specific DNS provider.

## Exec Interface

### Environment Variables

* APIVERSION - The current API version.
* COMMAND - The action to take. CREATE or DELETE.
  * CREATE: The plugin MUST create a DNS TXT based on FQDN, TOKEN, and DOMAIN.
  * DELETE: The plugin MUST delete a DNS TXT record based on FQDN.
* FQDN - The fqdn of the DNS TXT record
* TOKEN - The value of the DNS TXT record
* DOMAIN - The domain where the TXT record must be created

### Stdin

Arbitrary configuration data will be written on stdin. The contents of the configuration data will be provider specific and must be documented separately.

## Error Handling

### Exit Codes

* 0 - Success. The DNS TXT record was created or deleted successfully.
* 1 - Fail. Generic error such as failure to create or delete a DNS TXT record.
* 2 - Invalid Configuration. Use this when the configuration passed on stdin is invalid.
* 3 - Unsupported API version. Use this when the plugin does not support this API version.

### Error Messages

A single error should be printed to stderr that will be logged by the `kube-cert-manager`

## Examples

The following example executes a binary named `dns01`. The `dns01` plugin does not create or delete DNS records, but it does provide a good example of how to write a DNS-01 plugin.

### Create

```
cat dns01.json | \
  APIVERSION="v1" \
  COMMAND="CREATE" \
  DOMAIN="example.com" \
  FQDN="_acme-challenge.example.com" \
  TOKEN="8bGFl9SNhZzukcwdR7e52gFwq6HaEHB43LbimZQwnLg" \
  dns01
```

```
echo $?
```
```
0
```

### Delete

```
cat dns01.json | \
  APIVERSION="v1" \
  COMMAND="DELETE" \
  DOMAIN="example.com" \
  FQDN="_acme-challenge.example.com" \
  TOKEN="8bGFl9SNhZzukcwdR7e52gFwq6HaEHB43LbimZQwnLg" \
  dns01
```

```
echo $?
```
```
0
```

### API Version Conflict

```
cat dns01.json | \
  APIVERSION="v2" \
  COMMAND="DELETE" \
  DOMAIN="example.com" \
  FQDN="_acme-challenge.example.com" \
  TOKEN="8bGFl9SNhZzukcwdR7e52gFwq6HaEHB43LbimZQwnLg" \
  dns01
```

```
echo $?
```
```
3
```

### Bad Configuration Data

```
cat dns01-bad.json | \
  APIVERSION="v1" \
  COMMAND="DELETE" \
  DOMAIN="example.com" \
  FQDN="_acme-challenge.example.com" \
  TOKEN="8bGFl9SNhZzukcwdR7e52gFwq6HaEHB43LbimZQwnLg" \
  dns01
```

A single error message is printed to stderr.

```
invalid character 'B' looking for beginning of value
```

```
echo $?
```
```
2
```
