# pyodbc-mssql-lambda-layer-gconv-extra

AWS Lambda Layer for connecting to Microsoft SQL Server with `pyodbc` and Microsoft ODBC Driver, including `glibc-gconv-extra` for Amazon Linux 2023.

## Why this layer exists

AWS Lambda runtimes for Python 3.12 and later are based on Amazon Linux 2023.

Amazon Linux 2023 does not include `glibc-gconv-extra` by default. This can cause pyodbc and Microsoft ODBC Driver to fail when retrieving Unicode data from SQL Server with an error such as:

```text
Unicode conversion failed (22) (SQLGetData)
```

A connection to SQL Server may succeed while retrieving actual Unicode data fails.

This layer includes the required `glibc-gconv-extra` conversion modules in addition to the pyodbc and SQL Server ODBC dependencies.

## When you may need this

This layer may be useful if all of the following apply:

- AWS Lambda
- Python 3.12 or later
- Amazon Linux 2023 based runtime
- pyodbc
- Microsoft ODBC Driver for SQL Server
- Unicode data is retrieved from SQL Server
- You encounter `Unicode conversion failed (22) (SQLGetData)`

Typical examples include SQL Server databases containing Japanese, Chinese, Korean, or other non-ASCII text.

## Important

Set the following environment variable in your Lambda function:

```text
GCONV_PATH=/opt/lib/gconv
```

Without this setting, the conversion modules included in the layer may not be discovered correctly.

## Example

```python
import os
import pyodbc

# GCONV_PATH should normally be configured as a Lambda
# environment variable.
print(os.environ.get("GCONV_PATH"))

connection = pyodbc.connect(
    "DRIVER={ODBC Driver 18 for SQL Server};"
    "SERVER=your-sql-server;"
    "DATABASE=your-database;"
    "UID=your-user;"
    "PWD=your-password;"
    "Encrypt=yes;"
    "TrustServerCertificate=yes;"
)

cursor = connection.cursor()

# Test Unicode retrieval, not just connectivity.
cursor.execute("""
    SELECT
        N'日本語テスト' AS japanese_text,
        N'Unicode test' AS unicode_text
""")

print(cursor.fetchone())
```

A simple connection test or ASCII-only query may not reproduce the Unicode conversion problem. Test retrieval of actual Unicode data.

## Supported runtimes and architectures

See the Dockerfiles and releases in this repository for the currently available Python runtime and CPU architecture combinations.

The repository contains build definitions for multiple Python versions and architectures.

## Background

This project is based on:

[davidcomerford/pyodbc-mssql-lambda-layer](https://github.com/davidcomerford/pyodbc-mssql-lambda-layer)

The upstream project provides a Lambda Layer for pyodbc and Microsoft SQL Server connectivity.

This fork additionally addresses the `glibc-gconv-extra` dependency required for Unicode conversion on Amazon Linux 2023 based Lambda runtimes.

## glibc-gconv-extra only

If you already have your own pyodbc or ODBC Driver layer and only need the character conversion modules, see:

[YasuhiroHarada/glibc-gconv-extra-lambda-layer](https://github.com/YasuhiroHarada/glibc-gconv-extra-lambda-layer)

## Building it yourself

For production workloads, building and publishing the layer into your own AWS account is recommended rather than depending on a third-party public Lambda Layer.

This also makes it possible to control:

- pyodbc version
- Microsoft ODBC Driver version
- Python runtime
- CPU architecture
- Build provenance
- Dependency updates

See the Dockerfiles and GitHub Actions workflow in this repository for the reproducible build process.

## Security

The layer contains native libraries used by Lambda functions that may have access to SQL Server credentials and databases.

For security-sensitive or production workloads, consider reviewing the build definition and producing the layer in your own CI/CD environment.

Avoid automatically consuming newly published dependency versions without review.

Recommended controls include:

- Pin dependency versions
- Verify downloaded package checksums where practical
- Review changes before publishing a new layer version
- Publish layers into an AWS account controlled by your organization
- Grant the Lambda execution role only the permissions it requires
- Store database credentials in AWS Secrets Manager or an equivalent service

## Troubleshooting

### `Unicode conversion failed (22) (SQLGetData)`

Verify that:

1. The Lambda runtime is Python 3.12 or later and is based on Amazon Linux 2023.
2. This layer is attached to the function.
3. `GCONV_PATH` is set to:

   ```text
   /opt/lib/gconv
   ```

4. The function has been redeployed after changing its layers or environment variables.
5. The problem is reproduced using actual Unicode data rather than an ASCII-only connectivity test.

### SQL Server connection itself fails

This layer specifically addresses pyodbc and ODBC Driver dependencies, including the Amazon Linux 2023 Unicode conversion issue.

The following must be diagnosed separately:

- VPC routing
- Security Groups and Network ACLs
- Site-to-Site VPN or Direct Connect routing
- SQL Server authentication
- TLS and certificate settings
- SQL Server permissions
- Firewall rules
- SQL Server TCP/IP configuration

## License

See [LICENSE](LICENSE).
