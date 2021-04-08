# Генерация самоподписанных сертификатов
## Команда openssl и её параметры    
Встроенная в линукс криптографическая библиотека openssl имеет инструменты, предназначенные для генерации приватных ключей RSA и Certificate Singing Requests (CSR-запросов), управление сертификатами и выполнение кодирования/декодирования.    
Для создания сертификатов вам понадобятся директории: `crl`, `certs`, `requests`, `newcerts`. А также файлы `index.txt` и `index.txt.attr`. И ещё файл `serial` с записью 01.    
Для генерации ключа, запроса и подписания сертификатов мы используем утилиту openssl. Для генерации ключа существует опция genrsa, для запроса req, для подписания ca.    

openssl:
+ genrsa
    + out - указывает путь сохранения ключа
    + aes256 - шифрование ключа
    + 4096 - длина ключа
+ req
    + config - указывает используемый файл конфигурации
    + new / x509 -  certificate for CA or not?
    + nodes - без шифрования
    + extensions - расширение
        + v3_ca
        + v3_intermediate_ca
        + server_cert
    + subj - указание значений полей темы сертификата
    + key - путь до приватного ключа
    + out - путь куда сохранить запрос
+ ca
    + config - указывает используемый файл конфигурации
    + extensions - расширение
    + in - запрос для подписания
    + out - путь куда сохранить подписанный сертификат    

## Пример скрипта    
Часто, для быстрой генерации большого количества сертификатов куда разумнее будет использовать самонаписанный скирпт. Вот небольшой пример генерации корневого сертификата.    
```bash
#/bin/bash
certs_dir="/etc/pki/"
home_dir=$(pwd)

certificates=(CA)

value_subj="/C=RU/ST=Moscow\
/L=Moscow/O=InfoWatch\
/OU=IT/CN=?\
/emailAddress=support@demo.lab"

if ! [[ -d $certs_dir ]]; then
  mkdir -p $certs_dir
  cd $certs_dir
fi

create_files () {
  touch index.txt
  echo 01 > serial
  mkdir crl certs requests newcerts
}

create_certificate () {
  export SAN=${value_subject_alt_name//\?/${2}}

  openssl req \
    -config openssl.cnf \
    ${3} \
    -nodes \
    -extensions $1 \
    -subj ${value_subj/\?/${2}} \
    -key private.pem \
    -out public.pem
}

sign_certificate () {
  openssl ca \
    -config openssl.cnf \
    -notext \
    -extensions $1 \
    -in "${work_dir}/public.pem" \
    -out "${work_dir}/public.pem"
}

for cert in ${certificates[@]}; do
  work_dir="$(pwd)/${cert}"

  mkdir $work_dir && cd $work_dir
  cp ${home_dir}/openssl.cnf .
  create_files
  openssl genrsa -out private.pem &> /dev/null

  create_certificate "v3_ca" "CA" "-x509"

  cd $certs_dir
done
```
