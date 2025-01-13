# Домашнее задание к занятию «Управляющие конструкции в коде Terraform»

### Цели задания

1. Отработать основные принципы и методы работы с управляющими конструкциями Terraform.
2. Освоить работу с шаблонизатором Terraform (Interpolation Syntax).

------

### Чек-лист готовности к домашнему заданию

1. Зарегистрирован аккаунт в Yandex Cloud. Использован промокод на грант.
2. Установлен инструмент Yandex CLI.
3. Доступен исходный код для выполнения задания в директории [**03/src**](https://github.com/netology-code/ter-homeworks/tree/main/03/src).
4. Любые ВМ, использованные при выполнении задания, должны быть прерываемыми, для экономии средств.

------

### Внимание!! Обязательно предоставляем на проверку получившийся код в виде ссылки на ваш github-репозиторий!
Убедитесь что ваша версия **Terraform** ~>1.8.4
Теперь пишем красивый код, хардкод значения не допустимы!
------

### Задание 1

1. Изучите проект.
2. Инициализируйте проект, выполните код. 


Приложите скриншот входящих правил «Группы безопасности» в ЛК Yandex Cloud .

## Ответ на задание 1

1.  ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-1-1.png)

------

### Задание 2

1. Создайте файл count-vm.tf. Опишите в нём создание двух **одинаковых** ВМ  web-1 и web-2 (не web-0 и web-1) с минимальными параметрами, используя мета-аргумент **count loop**. Назначьте ВМ созданную в первом задании группу безопасности.(как это сделать узнайте в документации провайдера yandex/compute_instance )
2. Создайте файл for_each-vm.tf. Опишите в нём создание двух ВМ для баз данных с именами "main" и "replica" **разных** по cpu/ram/disk_volume , используя мета-аргумент **for_each loop**. Используйте для обеих ВМ одну общую переменную типа:
```
variable "each_vm" {
  type = list(object({  vm_name=string, cpu=number, ram=number, disk_volume=number }))
}
```  
При желании внесите в переменную все возможные параметры.
4. ВМ из пункта 2.1 должны создаваться после создания ВМ из пункта 2.2.
5. Используйте функцию file в local-переменной для считывания ключа ~/.ssh/id_rsa.pub и его последующего использования в блоке metadata, взятому из ДЗ 2.
6. Инициализируйте проект, выполните код.

## Ответ на задание 2

1.  count-vm.tf
    ```
    data "yandex_compute_image" "ubuntu" {
      family = "${var.vm_os_family}"
    }

    resource "yandex_compute_instance" "web" {
      count = 2
      name        = "web-${count.index + 1}"
      hostname    = "web-${count.index + 1}"
      platform_id = var.platform_id.pi1

      resources {
        cores         = var.vms_resurces.vm_web_resources.core
        memory        = var.vms_resurces.vm_web_resources.memory
        core_fraction = var.vms_resurces.vm_web_resources.core_fraction
      }

      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }

        scheduling_policy {
        preemptible = var.preemptible.yes
      }

      network_interface {
        subnet_id = yandex_vpc_subnet.develop.id
        security_group_ids = [yandex_vpc_security_group.example.id]
        nat = var.nat.yes
      }

      metadata = local.metadata
  
      allow_stopping_for_update = var.stop_vm.yes

      depends_on = [yandex_compute_instance.platform2]
    }
    ```
2.  for_each-vm.tf
    ```
    resource "yandex_compute_instance" "platform2" {
      for_each = {
        "0" = "main"
        "1" = "replica"
      }
      name        = "${var.each_vm[each.key]["vm_name"]}${each.value}"
      hostname    = "${var.each_vm[each.key]["vm_name"]}${each.value}"
      platform_id = var.platform_id.pi1
      resources {
        cores         = "${var.each_vm[each.key]["cpu"]}"
        memory        = "${var.each_vm[each.key]["ram"]}"
        core_fraction = "${var.each_vm[each.key]["core_fraction"]}"
      }
      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
          size   = "${var.each_vm[each.key]["disk_volume"]}"
        }
      }

      scheduling_policy {
        preemptible = var.preemptible.yes
      }

      network_interface {
        subnet_id = yandex_vpc_subnet.develop.id
        nat = var.nat.yes
      }

      metadata = local.metadata

      allow_stopping_for_update = var.stop_vm.yes
    }
    ```
3.  local.tf
    ```
    locals {
        metadata = {
            "serial-port-enable" = "1"
            "ssh-keys"           = "${var.ssh_username}:${file(var.vms_ssh_root_key)}"
        }
    }
    ```
4.  variables.tf
    ```
    variable "vms_ssh_root_key" {
      description = "Путь к публичному SSH ключу"
      type        = string
      default     = "/home/vlad/.ssh/id_ed25519.pub"
    }

    variable "ssh_username" {
      description = "Имя пользователя для SSH ключей"
      type        = string
      default     = "ubuntu"
    }

    variable "vms_resurces" {
      description = "Ресурсы VM web"
      type        = map(number)
      default = {
        core           = 2
        memory         = 1
        core_fraction  = 5
      }
    }

    variable "each_vm" {
      description = "Ресурсы VM db"
      type        = list(object({
        vm_name = string
        cpu  = number
        ram  = number
        core_fraction = number
        disk_volume = number
      }))
      efault     = [{
        vm_name = ""
        cpu = 2
        ram = 2
        core_fraction = 20
        disk_volume = 10
      },
      {
        vm_name = ""
        cpu = 2
        ram = 1
        core_fraction = 5
        disk_volume = 5
      }
      ]
    }
    ```
5. ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-2-1.png)

------

### Задание 3

1. Создайте 3 одинаковых виртуальных диска размером 1 Гб с помощью ресурса yandex_compute_disk и мета-аргумента count в файле **disk_vm.tf** .
2. Создайте в том же файле **одиночную**(использовать count или for_each запрещено из-за задания №4) ВМ c именем "storage"  . Используйте блок **dynamic secondary_disk{..}** и мета-аргумент for_each для подключения созданных вами дополнительных дисков.

## Ответ на задание 3

1.  disc_vm.tf
    ```
    resource "yandex_compute_disk" "additional_disk" {
      count = 3

      name = "disk-${count.index + 1}"
      size = 1 

      type = "network-hdd"
    }

    resource "yandex_compute_instance" "storage" {
      name        = "storage"
      hostname    = "storage"
      platform_id = var.platform_id.pi1

      resources {
        cores         = var.vms_resurces.core
        memory        = var.vms_resurces.memory
        core_fraction = var.vms_resurces.core_fraction
      }

      boot_disk {
        initialize_params {
          image_id = data.yandex_compute_image.ubuntu.image_id
        }
      }

      network_interface {
        subnet_id = yandex_vpc_subnet.develop.id
        security_group_ids = [yandex_vpc_security_group.example.id]
        nat = var.nat.yes
      }

      dynamic "secondary_disk" {
        for_each = yandex_compute_disk.additional_disk 

        content {
          disk_id     = secondary_disk.value.id
        }
      }
      metadata = local.metadata

      allow_stopping_for_update = var.stop_vm.yes
    }
    ```
2. ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-3-1.png)
------

### Задание 4

1. В файле ansible.tf создайте inventory-файл для ansible.
Используйте функцию tepmplatefile и файл-шаблон для создания ansible inventory-файла из лекции.
Готовый код возьмите из демонстрации к лекции [**demonstration2**](https://github.com/netology-code/ter-homeworks/tree/main/03/demo).
Передайте в него в качестве переменных группы виртуальных машин из задания 2.1, 2.2 и 3.2, т. е. 5 ВМ.
2. Инвентарь должен содержать 3 группы и быть динамическим, т. е. обработать как группу из 2-х ВМ, так и 999 ВМ.
3. Добавьте в инвентарь переменную  [**fqdn**](https://cloud.yandex.ru/docs/compute/concepts/network#hostname).
``` 
[webservers]
web-1 ansible_host=<внешний ip-адрес> fqdn=<полное доменное имя виртуальной машины>
web-2 ansible_host=<внешний ip-адрес> fqdn=<полное доменное имя виртуальной машины>

[databases]
main ansible_host=<внешний ip-адрес> fqdn=<полное доменное имя виртуальной машины>
replica ansible_host<внешний ip-адрес> fqdn=<полное доменное имя виртуальной машины>

[storage]
storage ansible_host=<внешний ip-адрес> fqdn=<полное доменное имя виртуальной машины>
```
Пример fqdn: ```web1.ru-central1.internal```(в случае указания переменной hostname(не путать с переменной name)); ```fhm8k1oojmm5lie8i22a.auto.internal```(в случае отсутвия перменной hostname - автоматическая генерация имени,  зона изменяется на auto). нужную вам переменную найдите в документации провайдера или terraform console.
4. Выполните код. Приложите скриншот получившегося файла. 

Для общего зачёта создайте в вашем GitHub-репозитории новую ветку terraform-03. Закоммитьте в эту ветку свой финальный код проекта, пришлите ссылку на коммит.   
**Удалите все созданные ресурсы**.

## Ответ на задание 4
1.  ansible.tf
    ```
    resource "local_file" "inventory" {
      filename = "${path.module}/inventory.ini"

     content = templatefile("${path.module}/host.tftpl",
        {
          webservers  = yandex_compute_instance.web,
          databases   = yandex_compute_instance.platform2,
          storage     = yandex_compute_instance.storage
        }
      )
    }
    ```
2.  host.tftpl
    ```
    [webservers]

    %{~ for i in webservers ~}
    ${i["name"]} ansible_host=${i["network_interface"][0]["nat_ip_address"]} fqdn=${i["fqdn"]}

    %{~ endfor ~}

    [databases]

    %{~ for i in databases ~}
    ${i["name"]} ansible_host=${i["network_interface"][0]["nat_ip_address"]} fqdn=${i["fqdn"]}

    %{~ endfor ~}

    [storage]
    ${storage["name"]} ansible_host=${storage["network_interface"][0]["nat_ip_address"]} fqdn=${storage["fqdn"]}
    ```
3.  ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-4-1.png)
------

## Дополнительные задания (со звездочкой*)

**Настоятельно рекомендуем выполнять все задания со звёздочкой.** Они помогут глубже разобраться в материале.   
Задания со звёздочкой дополнительные, не обязательные к выполнению и никак не повлияют на получение вами зачёта по этому домашнему заданию. 

### Задание 5* (необязательное)
1. Напишите output, который отобразит ВМ из ваших ресурсов count и for_each в виде списка словарей :
``` 
[
 {
  "name" = 'имя ВМ1'
  "id"   = 'идентификатор ВМ1'
  "fqdn" = 'Внутренний FQDN ВМ1'
 },
 {
  "name" = 'имя ВМ2'
  "id"   = 'идентификатор ВМ2'
  "fqdn" = 'Внутренний FQDN ВМ2'
 },
 ....
...итд любое количество ВМ в ресурсе(те требуется итерация по ресурсам, а не хардкод) !!!!!!!!!!!!!!!!!!!!!
]
```
Приложите скриншот вывода команды ```terrafrom output```.

## Ответ на задание 5
1.  output.tf
    ```
    output "vm" {
      description = "Список всех виртуальных машин с именем, ID и FQDN"
      value = concat(

        [
          for vm in yandex_compute_instance.web : {
            name = vm.name
            id   = vm.id
            fqdn = vm.hostname
          }
        ],
    
        [
          for key, vm in yandex_compute_instance.platform2 : {
            name = vm.name
            id   = vm.id
            fqdn = vm.hostname
          }
        ],
    
        [
          {
            name = yandex_compute_instance.storage.name
            id   = yandex_compute_instance.storage.id
            fqdn = yandex_compute_instance.storage.hostname
          }
        ]
      )
    }   
    ```
2.  ![alt text](https://github.com/VN351/ter-hw-3/raw/main/images/task-5-1.png)
------

### Задание 6* (необязательное)

1. Используя null_resource и local-exec, примените ansible-playbook к ВМ из ansible inventory-файла.
Готовый код возьмите из демонстрации к лекции [**demonstration2**](https://github.com/netology-code/ter-homeworks/tree/main/03/demo).
3. Модифицируйте файл-шаблон hosts.tftpl. Необходимо отредактировать переменную ```ansible_host="<внешний IP-address или внутренний IP-address если у ВМ отсутвует внешний адрес>```.

Для проверки работы уберите у ВМ внешние адреса(nat=false). Этот вариант используется при работе через bastion-сервер.
Для зачёта предоставьте код вместе с основной частью задания.

## Ответ на задание 6
1.  ansible.tf
    ```
    resource "local_file" "inventory" {
      filename = "${path.module}/inventory.ini"

      content = templatefile("${path.module}/host.tftpl",
        {
          webservers  = yandex_compute_instance.web,
          databases   = yandex_compute_instance.platform2,
          storage     = yandex_compute_instance.storage
        }
      )
    }

    resource "null_resource" "web_hosts_provision" {
      #Ждем создания инстанса
      depends_on = [yandex_compute_instance.storage, local_file.inventory]
  
      #Добавление ПРИВАТНОГО ssh ключа в ssh-agent
      provisioner "local-exec" {
        command    = "eval $(ssh-agent) && cat /home/vlad/.ssh/id_ed25519 | ssh-add -"
        on_failure = continue #Продолжить выполнение terraform pipeline в случае ошибок
      }

      #Запуск ansible-playbook
      provisioner "local-exec" {
        # without secrets
        command = "ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i ${abspath(path.module)}/inventory.ini ${abspath(path.module)}/test.yml"
        on_failure  = continue #Продолжить выполнение terraform pipeline в случае ошибок
        environment = { ANSIBLE_HOST_KEY_CHECKING = "False" }
      }

      #срабатывание триггера при изменении переменных
      triggers = {
        always_run      = "${timestamp()}" #всегда т.к. дата и время постоянно изменяются
        always_run_uuid = "${uuid()}"
        #playbook_src_hash = file("${abspath(path.module)}/test.yml") # при изменении содержимого playbook файла
        #ssh_public_key    = var.public_key                           # при изменении переменной with ssh
        #template_rendered = "${local_file.hosts_templatefile.content}" #при изменении inventory-template
        #password_change = jsonencode( {for k,v in random_password.each: k=>v.result})
      }
    }
    ```
2. ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-6-1.png)
3.  host.tftpl
    ```
    [webservers]

    %{~ for i in webservers ~}
    ${i["name"]} ansible_host=${i["network_interface"][0]["nat_ip_address"] != "" ? i["network_interface"][0]["nat_ip_address"] : i["network_interface"][0]["ip_address"]} fqdn=${i["fqdn"]}

    %{~ endfor ~}

    [databases]

    %{~ for i in databases ~}
    ${i["name"]} ansible_host=${i["network_interface"][0]["nat_ip_address"] != "" ? i["network_interface"][0]["nat_ip_address"] : i["network_interface"][0]["ip_address"]} fqdn=${i["fqdn"]}

    %{~ endfor ~}

    [storage]
    ${storage["name"]} ansible_host=${storage["network_interface"][0]["nat_ip_address"] != "" ? storage["network_interface"][0]["nat_ip_address"] : storage["network_interface"][0]["ip_address"]} fqdn=${storage["fqdn"]}
    ```
4.  ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-6-2.png)

### Правила приёма работы

В своём git-репозитории создайте новую ветку terraform-03, закоммитьте в эту ветку свой финальный код проекта. Ответы на задания и необходимые скриншоты оформите в md-файле в ветке terraform-03.

В качестве результата прикрепите ссылку на ветку terraform-03 в вашем репозитории.

Важно. Удалите все созданные ресурсы.

### Задание 7* (необязательное)
Ваш код возвращает вам следущий набор данных: 
```
> local.vpc
{
  "network_id" = "enp7i560tb28nageq0cc"
  "subnet_ids" = [
    "e9b0le401619ngf4h68n",
    "e2lbar6u8b2ftd7f5hia",
    "b0ca48coorjjq93u36pl",
    "fl8ner8rjsio6rcpcf0h",
  ]
  "subnet_zones" = [
    "ru-central1-a",
    "ru-central1-b",
    "ru-central1-c",
    "ru-central1-d",
  ]
}
```
Предложите выражение в terraform console, которое удалит из данной переменной 3 элемент из: subnet_ids и subnet_zones.(значения могут быть любыми) Образец конечного результата:
```
> <некое выражение>
{
  "network_id" = "enp7i560tb28nageq0cc"
  "subnet_ids" = [
    "e9b0le401619ngf4h68n",
    "e2lbar6u8b2ftd7f5hia",
    "fl8ner8rjsio6rcpcf0h",
  ]
  "subnet_zones" = [
    "ru-central1-a",
    "ru-central1-b",
    "ru-central1-d",
  ]
}
```
## Ответ на задание 7

1.  ![alt text](https://github.com/VN351/ter_hw_3/raw/main/images/task-7-1.png)

### Задание 8* (необязательное)
Идентифицируйте и устраните намеренно допущенную в tpl-шаблоне ошибку. Обратите внимание, что terraform сам сообщит на какой строке и в какой позиции ошибка!
```
[webservers]
%{~ for i in webservers ~}
${i["name"]} ansible_host=${i["network_interface"][0]["nat_ip_address"] platform_id=${i["platform_id "]}}
%{~ endfor ~}
```

## Ответ на задание 8

Исправленный вариант
```
[webservers]
%{~ for i in webservers ~}
${i["name"]} ansible_host=${i["network_interface"][0]["nat_ip_address"]} platform_id=${i["platform_id"]}
%{~ endfor ~}

```
Закрывающая скобка "}" была перенесена с конца строки и поставлена после  `{i["network_interface"][0]["nat_ip_address"]`

### Задание 9* (необязательное)
Напишите  terraform выражения, которые сформируют списки:
1. ["rc01","rc02","rc03","rc04",rc05","rc06",rc07","rc08","rc09","rc10....."rc99"] те список от "rc01" до "rc99"
2. ["rc01","rc02","rc03","rc04",rc05","rc06","rc11","rc12","rc13","rc14",rc15","rc16","rc19"....."rc96"] те список от "rc01" до "rc96", пропуская все номера, заканчивающиеся на "0","7", "8", "9", за исключением "rc19"

## Ответ на задание 9

1.  range.tf
    ```
    locals {
      rc_list_01_to_99 = [
        for i in range(1, 100) : format("rc%02d", i)
      ]

      rc_list_01_to_96_filtered = [
        for i in range(1, 97) : format("rc%02d", i)
        if (
          i == 19
          ||
          (i % 10 != 0) &&
          (i % 10 != 7) &&
          (i % 10 != 8) &&
          (i % 10 != 9)
        )
      ]
    }
    ```
2.  output.tf
    ```
    output "rc_list_01_to_99" {
      value = local.rc_list_01_to_99
    }

    output "rc_list_filtered" {
      description = "Отфильтрованный список от rc01 до rc96"
      value       = local.rc_list_01_to_96_filtered
    }
    ```

## Ссылка на репозиторий с выполненой работой

[Ссылка на конфигурацию Terraform](https://github.com/VN351/dev/tree/master/src)


### Критерии оценки

Зачёт ставится, если:

* выполнены все задания,
* ответы даны в развёрнутой форме,
* приложены соответствующие скриншоты и файлы проекта,
* в выполненных заданиях нет противоречий и нарушения логики.

На доработку работу отправят, если:

* задание выполнено частично или не выполнено вообще,
* в логике выполнения заданий есть противоречия и существенные недостатки. 


