Use the `influxdb3` CLI or the HTTP API to regenerate the operator (`_admin`) token for your {{% product-name %}} instance.
Regenerate a token to rotate it as part of your security practices or if you suspect
the token has been compromised.

Regenerating the operator token deactivates the previous token,
stores the SHA512 hash and metadata of the new token, and returns the new token string.

## Prerequisite

To regenerate an operator token, you need the current token string.

## Use the CLI or HTTP API to regenerate the operator token

> [!Important]
> #### Regenerating the operator token
> Regenerating the operator token invalidates the previous token.
> Make sure to update any applications or scripts that use the operator token.

To regenerate the operator token, use the [`influxdb3 serve create token` command](/influxdb3/version/reference/cli/influxdb3/create/token/) (CLI) or the
[`/api/v3/configure/token/admin/regenerate` endpoint](/influxdb3/version/api/v3/configure/token/admin/regenerate) (HTTP API):

{{< tabs-wrapper >}}
{{% tabs %}}
[CLI](#cli-regenerate)
[HTTP API](#http-api-regenerate)
{{% /tabs %}}
{{% tab-content %}}
<!---------------------------------BEGIN CLI----------------------------------->
Use the `--regenerate` flag with the
[`influxdb3 create token --admin`](/influxdb3/version/reference/cli/influxdb3/create/token/) subcommand--for example:

{{% code-placeholders "OPERATOR_TOKEN" %}}
```bash
influxdb3 create token --admin \
  --regenerate
  OPERATOR_TOKEN
```
{{% /code-placeholders %}}

In your command,
replace {{% code-placeholder-key %}}`OPERATOR_TOKEN`{{% /code-placeholder-key %}}
with the current operator (`_admin`) token string.

The output contains the new token string and InfluxDB deactivates the previous token string. 
<!----------------------------END CLI------------------------------------------>
{{% /tab-content %}}
{{% tab-content %}}
<!----------------------------BEGIN HTTP API----------------------------------->
Use the following HTTP API endpoint:

{{% api-endpoint method="POST" endpoint="/api/v3/configure/token/admin/regenerate" api-ref="/influxdb3/version/api/v3/configure/token/admin/regenerate" %}}

In your request, send an `Authorization` header with your current operator token string 
--for example:

{{% code-placeholders "OPERATOR_TOKEN" %}}
```bash
curl -X POST "http://{{< influxdb/host >}}/api/v3/configure/token/admin/regenerate" \
  --header "Authorization: Bearer OPERATOR_TOKEN" \
  --header "Accept: application/json"
```
{{% /code-placeholders %}}

In your command, replace {{% code-placeholder-key %}}`OPERATOR_TOKEN`{{% /code-placeholder-key %}} with the current token string.

The response body contains the new operator token string in plain text, and InfluxDB deactivates the previous token string.
<!------------------------END HTTP API ---------------------------------------->
{{% /tab-content %}}
{{< /tabs-wrapper >}}

To use the token as the default for later commands, and to persist the token
across sessions, assign the token string to the `INFLUXDB3_AUTH_TOKEN` environment variable.

## Lost admin token recovery

If you've lost your admin token and cannot regenerate it using the standard method, you can use the admin token recovery server:

1. Start {{< product-name >}} with the `--admin-token-recovery-http-bind` option:
   ```bash
   influxdb3 serve --admin-token-recovery-http-bind
   ```

2. In a separate terminal, regenerate the admin token using the recovery endpoint:
   
   ```bash
   influxdb3 create token --admin --regenerate --host http://127.0.0.1:8182
   ```

3. The recovery server automatically shuts down after successful token regeneration.

> [!Warning]
> The recovery server provides unauthenticated access to regenerate admin tokens. Only use this option when necessary and ensure the recovery endpoint (by default `127.0.0.1:8182`) is only accessible from trusted networks.

## Important considerations

- Regenerating the operator token invalidates the previous token.
- If you lose the operator token, use the recovery server method described above.
- `--regenerate` only works for the operator token. You can't use the `--regenerate` flag with the `influxdb3 create token --admin` command to regenerate a named admin token.
- Ensure that you update any applications or scripts that use the operator token with the new token string.
- Always store your operator token securely and consider implementing proper secret management practices.
