# MySQL binary builds

GitHub Actions builds MySQL binaries for Linux ARM64 (UOS 20-compatible ABI) and Windows x64. Normal pushes and pull requests run the build and runtime smoke tests; the generated artifacts are retained in GitHub Actions for 14 days.

## Create a release

1. Update `MYSQL_VERSION` in [`.github/workflows/build-and-release.yml`](.github/workflows/build-and-release.yml) and in each reusable platform workflow when changing the MySQL release being built.
2. Push a stable tag that exactly matches that version:

   ```sh
   git tag v8.0.25
   git push origin v8.0.25
   ```

The workflow validates the tag before starting either platform build. Tags with a mismatched version, malformed tags, and prerelease suffixes (such as `v8.0.25-rc.1`) fail without publishing a GitHub Release.

After both platform builds and their smoke tests succeed, one GitHub Release is created with GitHub-generated release notes and these assets:

- `mysql-<version>-linux-arm64-uos20.tar.gz`
- `mysql-<version>-linux-arm64-uos20.tar.gz.sha256`
- `mysql-<version>-windows-x64.zip`
- `mysql-<version>-windows-x64.zip.sha256`

## Linux ARM64 package

Verify the downloaded archive before extracting it:

```sh
sha256sum --check mysql-8.0.25-linux-arm64-uos20.tar.gz.sha256
```

The tar archive intentionally has a package directory at its root; its usable MySQL installation is below `usr/local/mysql` within that directory:

```text
mysql-8.0.25-linux-arm64-uos20/
└── usr/local/mysql/
```

To extract the package while keeping that layout:

```sh
# The archive already contains its own top-level directory.
tar -xzf mysql-8.0.25-linux-arm64-uos20.tar.gz
./mysql-8.0.25-linux-arm64-uos20/usr/local/mysql/bin/mysqld --version
```

To install its `usr/local/mysql` tree under `/usr/local`, use the same path-preserving extraction that the runtime validation uses:

```sh
sudo tar -C / --strip-components=1 -xzf mysql-8.0.25-linux-arm64-uos20.tar.gz \
  mysql-8.0.25-linux-arm64-uos20/usr
/usr/local/mysql/bin/mysqld --version
```

## Windows x64 package

Verify the `.zip.sha256` file with PowerShell, then extract the ZIP and run `bin\mysqld.exe` from the extracted package directory:

```powershell
$expected = (Get-Content .\mysql-8.0.25-windows-x64.zip.sha256).Split()[0]
$actual = (Get-FileHash .\mysql-8.0.25-windows-x64.zip -Algorithm SHA256).Hash.ToLower()
if ($actual -ne $expected) { throw 'SHA-256 mismatch' }
Expand-Archive .\mysql-8.0.25-windows-x64.zip -DestinationPath .\mysql
.\mysql\mysql-8.0.25-windows-x64\bin\mysqld.exe --version
```
