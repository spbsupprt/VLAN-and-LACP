# VLAN-and-LACP

Что нужно сделать?
- в Office1 в тестовой подсети появляется сервера с доп интерфейсами и адресами

- в internal сети testLAN:

  testClient1 - 10.10.10.254

  testClient2 - 10.10.10.254

  testServer1- 10.10.10.1

  testServer2- 10.10.10.1

- Равести вланами:

testClient1 <-> testServer1

testClient2 <-> testServer2

- Между centralRouter и inetRouter "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд, проверить работу c отключением интерфейсов


---

Дано:

![image](https://github.com/user-attachments/assets/06fe42a6-2849-476d-8b3f-ed3ef1a97bf5)


Выполним плейбук https://github.com/spbsupprt/VLAN-and-LACP/blob/main/lacp.yml


![image](https://github.com/user-attachments/assets/c60984b3-0e23-45ac-bd76-7ca0ef82c474)
