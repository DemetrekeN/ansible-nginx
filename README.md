# Ansible for nginx

## Playbook для конфигурации nginx и запуска статичного сайта

Репозиторий содержит **playbook**, который проводит установку и конфигурацию **nginx**, с последующей разверткой [статичного сайта](https://torpeda.uberlegenheit.ru/), а также все необходимые файлы для этого *(тело сайта, файлы конфигурации, cron задачу и т.д)*.

### Основные составляющие playbook'а

- Таск для установки пакетов, которые понадобятся для работы **playbook'а**;
```
  - name: Install pkgs
    apt:
      pkg:
        - nginx
        - certbot
      force_apt_get: true
```

- Таск для копирования файлов конфигурации **nginx** и замена вирт. хоста **default** на новый; 
```
  - name: Copy config files for nginx 
    copy:
      src: nginx/
      dest: /etc/nginx/

  - name: Remove default symlink
    file:
      path: /etc/nginx/sites-enabled/default
      state: absent

  - name: Create symlink of configuration
    file:
      src: /etc/nginx/sites-available/gena
      dest: /etc/nginx/sites-enabled/gena
      group: root
      owner: root
      state: link
```

- Таск для переноса тела сайта
```
  - name: Move directory with site
    synchronize:
      src: files/site
      dest: /home/genadiy/practice/project-portfolio/portfolio
      owner: true
      group: true
```

- Таск для создания **SSL** сертификатов (*для протокола HTTPS*) и перенос **cron** задачи для их автообновления;
```
  - name: Create SSL certificates
    shell:
      cmd: certbot certonly --webroot -w /home/genadiy/practice/project-portfolio/portfolio --email root@rooted.onion --agree-tos --non-interactive -d torpeda.uberlegenheit.ru

  - name: Move file to auto renew SSL certificates
    copy:
      src: files/certbot
      dest: /etc/cron.d/
```

- Заключительный таск для ребута **nginx**, чтобы применить новые конфигурации.

```
  - name: Reload nginx for new confing
    service:
      name: nginx
      state: reloaded
```