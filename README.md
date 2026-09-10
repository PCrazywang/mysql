# MySQL binary builds from vendored source

This branch builds the MySQL source committed under `source/` for Linux ARM64 (UOS 20-compatible ABI) and Windows x64. The build version is read directly from [`source/MYSQL_VERSION`](source/MYSQL_VERSION); no separate workflow version needs to be maintained.

Normal pushes and pull requests run the build and runtime smoke tests. Their artifacts remain available in GitHub Actions for 14 days.

## Create a release

Push a stable version tag that exactly matches the version in `source/MYSQL_VERSION`:

```sh
git tag v8.0.25
git push origin v8.0.25
```

The workflow rejects malformed, mismatched, and prerelease tags (such as `v8.0.25-rc.1`) before platform builds start. After both platform builds and their smoke tests succeed, it creates one GitHub Release with GitHub-generated release notes and these assets:

- `mysql-<version>-linux-arm64-uos20.tar.gz`
- `mysql-<version>-linux-arm64-uos20.tar.gz.sha256`
- `mysql-<version>-windows-x64.zip`
- `mysql-<version>-windows-x64.zip.sha256`

## Linux ARM64 package

Verify the downloaded archive before extracting it:

```sh
sha256sum --check mysql-8.0.25-linux-arm64-uos20.tar.gz.sha256
```

The usable MySQL installation is intentionally not at the archive root. The archive layout is:

```text
mysql-8.0.25-linux-arm64-uos20/
└── usr/local/mysql/
```

Extract it while preserving that package directory:

```sh
tar -xzf mysql-8.0.25-linux-arm64-uos20.tar.gz
./mysql-8.0.25-linux-arm64-uos20/usr/local/mysql/bin/mysqld --version
```

To install the contained `usr/local/mysql` tree below `/usr/local`, use:

```sh
sudo tar -C / --strip-components=1 -xzf mysql-8.0.25-linux-arm64-uos20.tar.gz \
  mysql-8.0.25-linux-arm64-uos20/usr
/usr/local/mysql/bin/mysqld --version
```

## Windows x64 package

Verify the ZIP checksum in PowerShell, extract it, then run `bin\mysqld.exe`:

```powershell
$expected = (Get-Content .\mysql-8.0.25-windows-x64.zip.sha256).Split()[0]
$actual = (Get-FileHash .\mysql-8.0.25-windows-x64.zip -Algorithm SHA256).Hash.ToLower()
if ($actual -ne $expected) { throw 'SHA-256 mismatch' }
Expand-Archive .\mysql-8.0.25-windows-x64.zip -DestinationPath .\mysql
.\mysql\mysql-8.0.25-windows-x64\bin\mysqld.exe --version
```
