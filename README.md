## Домашнее задание к занятию "08.04 Создание собственных modules"

#### 1-3. 
    Файл создан: my_new_module.py, заполнен и отредактирован

#### 4. Локальный тест

    vagrant@vagrant:~/gitwork/8.6-module$ python -m ansible.modules.my_new_module input.json
    
    {"invocation": {"module_args": {"content": "some data \nnew line", "path": "/tmp/test.txt"}}, "message": "file was written", "changed": true, "original_message": "some data \nnew line"}
    
    vagrant@vagrant:~/gitwork/8.6-module$ cat /tmp/test.txt
    some data 
    new line

#### 5-6. single task playbook+модуль + проверка  playbook

    vagrant@vagrant:~/gitwork/8.6-module$ ansible-playbook test_pb.yml
    
    [WARNING]: You are running the development version of Ansible. You should only run
    Ansible from "devel" if you are modifying the Ansible engine, or trying out features
    under development. This is a rapidly changing source of code and can become unstable at
    any point.
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the
    implicit localhost does not match 'all'
    [WARNING]: ansible.utils.display.initialize_locale has not been called, this may result
    in incorrectly calculated text widths that can cause Display to print incorrect line
    lengths
    
    PLAY [test my module] *******************************************************************
    
    TASK [Gathering Facts] ******************************************************************
    ok: [localhost]
    
    TASK [run my module] ********************************************************************
    changed: [localhost]
    
    TASK [dump out_msg] *********************************************************************
    ok: [localhost] => {
        "msg": {
            "changed": true,
            "failed": false,
            "message": "file was written",
            "original_message": "new content"
        }
    }
    
    PLAY RECAP ******************************************************************************
    localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


    Проверка на идемпотентность

    vagrant@vagrant:~/gitwork/8.6-module$ ansible-playbook test_pb.yml 
    [WARNING]: You are running the development version of Ansible. You should only run Ansible from "devel"
    if you are modifying the Ansible engine, or trying out features under development. This is a rapidly
    changing source of code and can become unstable at any point.
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
    does not match 'all'
    [WARNING]: ansible.utils.display.initialize_locale has not been called, this may result in incorrectly
    calculated text widths that can cause Display to print incorrect line lengths
    
    PLAY [test my module] **********************************************************************************
    
    TASK [Gathering Facts] *********************************************************************************
    ok: [localhost]
    
    TASK [run my module] ***********************************************************************************
    ok: [localhost]
    
    TASK [dump out_msg] ************************************************************************************
    ok: [localhost] => {
        "msg": {
            "changed": false,
            "failed": false,
            "message": "file was written",
            "original_message": "new content"
        }
    }
    
    PLAY RECAP *********************************************************************************************
    localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

#### 7. Выйдите из виртуального окружения.

    vagrant@vagrant:~/gitwork/8.6-module$  deactivate

#### 8. Инициализируйте новую collection:

    vagrant@vagrant:~/gitwork/8.6-module$ ansible-galaxy collection init netology.my_collection

#### 9-11. Playbook преобразован в роль, роль запущена и проверена на идемпотентность

8-11. Роль создана, запущена + идемпотентность

    vagrant@vagrant:~/gitwork/8.6-module/1 $ ansible-playbook site2.yml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
    does not match 'all'
    
    PLAY [localhost] ***************************************************************************************
    
    TASK [Gathering Facts] *********************************************************************************
    ok: [localhost]
    
    TASK [myrole : run my module] **************************************************************************
    changed: [localhost]
    
    PLAY RECAP *********************************************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    
    
    vagrant@vagrant:~/gitwork/8.6-module/1 $ ansible-playbook site2.yml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
    does not match 'all'
    
    PLAY [localhost] ***************************************************************************************
    
    TASK [Gathering Facts] *********************************************************************************
    ok: [localhost]
    
    TASK [myrole : run my module] **************************************************************************
    ok: [localhost]
    
    PLAY RECAP *********************************************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

#### 12-13. Коллекция выложена, cоздан .tar.gz

    vagrant@vagrant:~/gitwork/8.6-module/netology/my_collection$ ansible-galaxy collection build
    Created collection for my_netology.my_collection at /home/vagrant/gitwork/8.6-module/netology/my_collection/my_netology-my_collection-1.0.0.tar.gz

#### 14-16. Запуск Playbook из коллекции

    vagrant@vagrant:~/gitwork/8.6-module/1 $ ansible-playbook site.yml
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
    does not match 'all'
    
    PLAY [test my module] **********************************************************************************
    
    TASK [Gathering Facts] *********************************************************************************
    ok: [localhost]
    
    TASK [run my module] ***********************************************************************************
    ok: [localhost]
    
    TASK [dump test_out] ***********************************************************************************
    ok: [localhost] => {
        "msg": {
            "changed": false,
            "failed": false,
            "message": "file exists",
            "original_message": "new content"
        }
    }
    
    PLAY RECAP *********************************************************************************************
    localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


#### site.yml

    - name: test my module
      hosts: localhost
      tasks:
      - name  : run my module
        netology.my_collection.my_new_module:
          path: "/tmp/file.txt"
          content: "new content"
        register: out_msg
      - name: dump out_msg
        debug:
          msg: "{{ out_msg }}"
