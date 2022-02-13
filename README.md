# Домашнее задание к занятию "08.03 Использование Yandex Cloud"

```
---
- name: Install Elasticsearch
  hosts: elasticsearch
  handlers:
    - name: restart Elasticsearch #Хендлер, который запустится после настройки Elastic
      become: true
      service:
        name: elasticsearch
        state: restarted
  tasks:
    - name: "Download Elasticsearch's rpm" #Скачивание RPM пакета
      get_url:
        url: "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
        dest: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
      register: download_elastic
      until: download_elastic is succeeded #Ждать пока не скачается
    - name: Install Elasticsearch #Установка под sudo скаченного RPM пакета менеджером yum
      become: true
      yum:
        name: "/tmp/elasticsearch-{{ elk_stack_version }}-x86_64.rpm"
        state: present
    - name: Configure Elasticsearch #Настройка Elastic по шаблону j2
      become: true
      template:
        src: elasticsearch.yml.j2
        dest: /etc/elasticsearch/elasticsearch.yml
      notify: restart Elasticsearch #Запуск хендлера после настройки
- name: Install Kibana
  hosts: kibana
  handlers:
    - name: restart kibana  #Хендлер, который запустится после настройки Kibana
      become: true
      service:
        name: kibana
        state: restarted
  tasks:
    - name: "Download Kibana's rpm" #Скачивание RPM пакета
      get_url:
        url: "https://artifacts.elastic.co/downloads/kibana/kibana-{{ elk_stack_version }}-x86_64.rpm"
        dest: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
      register: download_kibana
      until: download_kibana is succeeded #Ждать пока не скачается пакет
    - name: Install Kibana #Установка Kibana c повышенными правами с помощью yum из rpm
      become: true
      yum:
        name: "/tmp/kibana-{{ elk_stack_version }}-x86_64.rpm"
        state: present
    - name: Configure Kibana #Настройка kibana по шаблону j2
      become: true
      template:
        src: kibana.yml.j2
        dest: /etc/kibana/kibana.yml 
      notify: restart kibana #Перезапуск (вызов хендлера) kibana после настройки 
- name: Install Filebeat
  hosts: filebeat
  handlers:
    - name: restart Filebeat #Хендлер для перезапуска filebeat
      become: true
      service:
        name: filebeat
        state: restarted
  tasks:
    - name: "Download Filebeat's rpm" #Скачивание rpm пакета
      get_url:
        url: "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-{{ elk_stack_version }}-x86_64.rpm"
        dest: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
      register: download_filebeat
      until: download_filebeat is succeeded #:Ждать пока не скачается
      tags: filebeat
    - name: Install Filebeat #Установка из rpm
      become: true
      yum:
        name: "/tmp/filebeat-{{ elk_stack_version }}-x86_64.rpm"
        state: present
      tags: filebeat
    - name: Configure Filebeat #Настройка по шаблону j2
      become: true
      template:
        src: filebeat.yml.j2
        dest: /etc/filebeat/filebeat.yml
      notify: restart Filebeat
      tags: filebeat
    - name: Filebeat systemwork #Инициализация filebeat в системе
      become: true
      command:
        cmd: filebeat modules enable system
        chdir: /usr/share/filebeat/bin
      register: filebeat_modules
      changed_when: filebeat_modules.stdout != 'Module system is already enabled'
      tags: filebeat
    - name: Loads the Kibana dashboards #Предварительная настройка kibana для отображения данных
      become: true
      command:
        cmd: filebeat setup
        chdir: /usr/share/filebeat/bin
      register: filebeat_setup
      changed_when: false
      until: filebeat_setup is succeeded
      tags: filebeat


```
