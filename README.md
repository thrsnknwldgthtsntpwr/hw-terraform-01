# Домашнее задание к занятию «Введение в Terraform»
## Цели задания

- Установить и настроить Terrafrom.
- Научиться использовать готовый код.


## Задание 1

1. Перейдите в каталог src. Скачайте все необходимые зависимости, использованные в проекте.
2. Изучите файл .gitignore. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)
- В файле personal.auto.tfvars
3. Выполните код проекта. Найдите в state-файле секретное содержимое созданного ресурса random_password, пришлите в качестве ответа конкретный ключ и его значение.
- `"result": "bKZ2Ekzq9WWAUgM7"`
4. Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла main.tf. Выполните команду terraform validate. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.
- `All resource blocks must have 2 labels (type, name).` - здесь не хватает имени для создаваемого ресурса
- `A name must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.`- здесь ошибка в том, что имя ресурса не может начинаться с цифры (1nginx)
- `A managed resource "random_password" "random_string_FAKE" has not been declared in the root module.` - здесь неверно задано имя ресурса (_FAKE надо удалить) и буква T должна быть t
5. Выполните код. В качестве ответа приложите: исправленный фрагмент кода и вывод команды docker ps.
```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
  required_version = ">=1.8.4" /*Многострочный комментарий.
 Требуемая версия terraform */
}
provider "docker" {
  host = "unix:///var/run/docker.sock"
}

#однострочный комментарий

resource "random_password" "random_string" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}

resource "docker_image" "nginx"{
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}
```

```
thrsnknwldgthtsntpwr@ubnt:~/NETOLOGY/hw-terraform-01$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS         PORTS                  NAMES
b4363a28c435   66f8bdd3810c   "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds   0.0.0.0:9090->80/tcp   example_bKZ2Ekzq9WWAUgM7
```

6. Замените имя docker-контейнера в блоке кода на hello_world. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду terraform apply -auto-approve. Объясните своими словами, в чём может быть опасность применения ключа -auto-approve. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды docker ps.
- Опасность применения ключа -auto-approve заключается в том, что можно в момент потерять всю инфраструктуру
- Данный ключ может пригодиться в скриптах
7. Уничтожьте созданные ресурсы с помощью terraform. Убедитесь, что все ресурсы удалены. Приложите содержимое файла terraform.tfstate.
```
{
  "version": 4,
  "terraform_version": "1.10.2",
  "serial": 25,
  "lineage": "7d09568a-5f25-6995-2136-5a6b02f79971",
  "outputs": {},
  "resources": [],
  "check_results": null
}
```
8. Объясните, почему при этом не был удалён docker-образ nginx:latest. Ответ ОБЯЗАТЕЛЬНО НАЙДИТЕ В ПРЕДОСТАВЛЕННОМ КОДЕ, а затем ОБЯЗАТЕЛЬНО ПОДКРЕПИТЕ строчкой из документации terraform провайдера docker. (ищите в классификаторе resource docker_image )
- Потому что задан параметр `keep_locally = true`
> keep_locally (Boolean) If true, then the Docker image won't be deleted on destroy operation. If this is false, it will delete the image from the docker ...
