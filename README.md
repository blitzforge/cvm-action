# Composer Version Manager Action

A GitHub Action to install and manage specific Composer versions in your workflow. This action allows you to easily switch between different Composer versions for testing compatibility or using specific features.

## Features

- 🎯 Install specific Composer versions (e.g., `2.7.1`, `1.10.27`)
- 🔄 Support for major version shortcuts (`1`, `2`)
- 📦 Support for channel versions (`latest`, `preview`, `snapshot`)
- ⚡ Fast installation with caching support
- 🛠️ Configurable installation directory
- ✅ Outputs installed version and path for use in subsequent steps

## Usage

### Basic Usage

```yaml
- name: Setup Composer
  uses: blitzforge/cvm-action@v1
```

This will install the latest stable version of Composer.

### Install Specific Version

```yaml
- name: Setup Composer 2.7.1
  uses: blitzforge/cvm-action@v1
  with:
    composer-version: '2.7.1'
```

### Install Major Version

```yaml
- name: Setup Composer 2.x
  uses: blitzforge/cvm-action@v1
  with:
    composer-version: '2'
```

### Install Preview Version

```yaml
- name: Setup Composer Preview
  uses: blitzforge/cvm-action@v1
  with:
    composer-version: 'preview'
```

### Complete Example

```yaml
name: PHP Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        php-version: ['8.1', '8.2', '8.3']
        composer-version: ['2.7.1', '2.6.6']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
      
      - name: Setup Composer
        uses: blitzforge/cvm-action@v1
        with:
          composer-version: ${{ matrix.composer-version }}
      
      - name: Install dependencies
        run: composer install --prefer-dist --no-progress
      
      - name: Run tests
        run: composer test
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `composer-version` | The Composer version to install. Can be a specific version (e.g., `2.7.1`), major version (`1`, `2`), or channel (`latest`, `preview`, `snapshot`) | No | `latest` |
| `working-directory` | Working directory for Composer installation | No | `.` |
| `install-dir` | Directory to install Composer binary | No | `/usr/local/bin` |

## Outputs

| Output | Description |
|--------|-------------|
| `composer-version` | The installed Composer version |
| `composer-path` | Path to the Composer executable |

### Using Outputs

```yaml
- name: Setup Composer
  id: composer-setup
  uses: blitzforge/cvm-action@v1
  with:
    composer-version: '2'

- name: Display Composer info
  run: |
    echo "Installed version: ${{ steps.composer-setup.outputs.composer-version }}"
    echo "Composer path: ${{ steps.composer-setup.outputs.composer-path }}"
```

## Supported Versions

This action supports all Composer versions available from [getcomposer.org](https://getcomposer.org/download/):

- **Specific versions**: `2.7.1`, `2.6.6`, `1.10.27`, etc.
- **Major versions**: `1`, `2`
- **Channels**: `latest`, `preview`, `snapshot`

## License

MIT License - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

If you encounter any issues or have questions, please [open an issue](https://github.com/blitzforge/cvm-action/issues).