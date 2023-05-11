## Sops и .env

1. Установите необходимые утилиты:

[SOPS](https://github.com/mozilla/sops/releases)

```shell
wget https://github.com/mozilla/sops/releases/download/v3.7.3/sops_3.7.3_amd64.deb -O sops

sudo dpkg -i sops
```

[AGE](https://github.com/FiloSottile/age/releases)

```shell
wget https://github.com/FiloSottile/age/releases/download/v1.1.1/age-v1.1.1-linux-amd64.tar.gz -O age.tar.gz

tar -zvxf age.tar.gz

sudo chmod +x age/age-keygen age/age
sudo mv age/age /usr/local/bin/age
sudo mv age/age-keygen /usr/local/bin/age-keygen
```

### Очистка

```shell
rm -rf age*
rm -rf sops
```

### Сгенерируйте age ключи

```shell
age-keygen -o keys.txt
```

### Добавьте публичный ключ в файл .sops.yaml

```yaml
creation_rules:
  - path_regex: '\.env'
    age: >-
      <укажите здесь публичный ключ>
```

### Закодируйте файл keys.txt в base64

```shell
cat keys.txt | base64 | tr -d "\n"
```

### Зашифровать .env файл

```shell
sops -e -i --input-type dotenv .env
```

### Расшифровать файл (опционально)

```shell
SOPS_AGE_KEY_FILE=<путь до приватного ключа age> sops -d -i .env
```

### Добавьте полученный результат в секрет Github

В Github Action ожидается переменная `SOPS_AGE_FILE`

```yaml
      - name: Import SOPS AGE KEY
        uses: timheuer/base64-to-file@v1.2
        with:
          encodedString: ${{ secrets.SOPS_AGE_FILE }}
          fileName: "agefile"
          fileDir: "/tmp"
```

Если вы добавляете переменную под другим именем, то так же поменяйте имя в экшене
