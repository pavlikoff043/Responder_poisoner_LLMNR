# Responder/MultiRelay #

IPv6/IPv4 LLMNR/NBT-NS/mDNS Poisoner and NTLMv1/2 Relay.

Author: Laurent Gaffie <laurent.gaffie@gmail.com >  https://g-laurent.blogspot.com

YOUTUBE: https://www.youtube.com/watch?v=hlANaZIPn90

## Intro ##

Responder - отравитель LLMNR, NBT-NS и MDNS. 

## Features ##

- Dual IPv6/IPv4 stack.

- Built-in SMB Auth server.
	
Поддерживает хэши NTLMv1, NTLMv2 с расширенной безопасностью NTLMSSP по умолчанию. Успешно протестирован с Windows 95 до Server 2022, Samba и Mac OSX Lion. Для NT4 поддерживаются пароли с открытым текстом и понижение хэширования LM при установке опции --lm. Если задана опция --disable-ess, то для аутентификации NTLMv1 будет отключена расширенная защита сеанса. SMBv2 также реализован и поддерживается по умолчанию.

- Built-in MSSQL Auth server.

Данный сервер поддерживает хэши NTLMv1, LMv2. Данная функциональность была успешно протестирована на серверах Windows SQL Server 2005, 2008, 2012, 2019.

- Built-in HTTP Auth server.

Данный сервер поддерживает хэши NTLMv1, NTLMv2 *и* базовую аутентификацию. Данный сервер был успешно протестирован на IE 6 - IE 11, Edge, Firefox, Chrome, Safari.

Примечание: Данный модуль также работает для WebDav NTLM-аутентификации, выдаваемой Windows WebDav-клиентами (WebClient). Теперь вы можете отправлять жертве свои пользовательские файлы.

- Built-in HTTPS Auth server.

Аналогично описанному выше. Папка certs/ содержит 2 ключа по умолчанию, включая фиктивный закрытый ключ. Это *намеренно*, цель - чтобы Responder работал из коробки. На случай, если вам понадобится сгенерировать собственную пару ключей с собственной подписью, был добавлен скрипт.

- Built-in LDAP Auth server.

Данный сервер поддерживает хэши NTLMSSP и простую аутентификацию (аутентификация открытым текстом). Данный сервер был успешно протестирован на Windows Support tool "ldp" и LdapAdmin.

- Built-in DCE-RPC Auth server.

Данный сервер поддерживает хэши NTLMSSP. Данный сервер был успешно протестирован на платформах Windows XP - Server 2019.

- Built-in FTP, POP3, IMAP, SMTP Auth servers.

Эти модули собирают учетные данные в открытом виде.

- Built-in DNS server.

Этот сервер будет отвечать на запросы типа SRV и A. Это очень удобно в сочетании с ARP-спуфингом. 

- Built-in WPAD Proxy Server.

Этот модуль перехватывает все HTTP-запросы от всех, кто запускает Internet Explorer в сети, если у них включена функция "Автоопределение настроек". Этот модуль очень эффективен. Вы можете настроить свой собственный PAC-скрипт в файле Responder.conf и внедрять HTML в ответы сервера. См. раздел Responder.conf.

- Browser Listener

Этот модуль позволяет обнаружить PDC в скрытом режиме.

- Icmp Redirect

    python tools/Icmp-Redirect.py

Для MITM на Windows XP/2003 и более ранних версиях членов домена. Эта атака в сочетании с модулем DNS достаточно эффективна.

- Rogue DHCP

    python tools/DHCP.py

DHCP Inform Spoofing. Позволяет выдать реальному DHCP-серверу IP-адреса, а затем отправить ответ DHCP Inform для установки вашего IP-адреса в качестве основного DNS-сервера и собственного WPAD URL. Для введения DNS-сервера, домена, маршрута во всех версиях Windows и в любом linux-комплексе используйте команду -R

- Analyze mode.

Этот модуль позволяет видеть NBT-NS, BROWSER, LLMNR, DNS-запросы в сети без отравления ответов. Также можно пассивно отобразить домены, MSSQL-серверы, рабочие станции, посмотреть, возможны ли атаки ICMP Redirects в вашей подсети. 

## Hashes ##

Все хэши выводятся в stdout и сбрасываются в уникальный файл, совместимый с John Jumbo, в таком формате:

    (MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt

Файлы журнала располагаются в папке "logs/". Запись и печать хэшей будет производиться только один раз для каждого пользователя для каждого типа хэша, если только вы не используете режим Verbose (-v).

- Responder будет записывать все свои действия в файл Responder-Session.log
- Режим анализа будет записываться в файл Analyzer-Session.log
- Отравление будет записываться в Poisoners-Session.log

Кроме того, все перехваченные хэши записываются в базу данных SQLite, которую можно настроить в файле Responder.conf


## Considerations ##

Соображения

- Данный инструмент прослушивает несколько портов: UDP 137, UDP 138, UDP 53, UDP/TCP 389,TCP 1433, UDP 1434, TCP 80, TCP 135, TCP 139, TCP 445, TCP 21, TCP 3141,TCP 25, TCP 110, TCP 587, TCP 3128, Multicast UDP 5355 и 5353.

- Если в вашей системе используется Samba, остановите smbd и nmbd, а также все остальные службы, прослушивающие эти порты.

- Для пользователей Ubuntu:

Отредактируйте этот файл /etc/NetworkManager/NetworkManager.conf и закомментируйте строку: `dns=dnsmasq`. Затем уничтожьте dnsmasq этой командой (от имени root): `killall dnsmasq -9`.

- Любой неавторизованный сервер может быть отключен в файле Responder.conf.

- Данный инструмент не предназначен для работы под Windows.

- Для OSX, пожалуйста, обратите внимание: Responder должен быть запущен с IP-адресом для флага -i (например, -i YOUR_IP_ADDR). В OSX нет встроенной поддержки привязки пользовательских интерфейсов. Использование флага -i en1 не будет работать. Кроме того, чтобы запустить Responder с наилучшими впечатлениями, выполните следующие действия от имени root:

    launchctl unload /System/Library/LaunchDaemons/com.apple.Kerberos.kdc.plist

    launchctl unload /System/Library/LaunchDaemons/com.apple.mDNSResponder.plist

    launchctl unload /System/Library/LaunchDaemons/com.apple.smbd.plist

    launchctl unload /System/Library/LaunchDaemons/com.apple.netbiosd.plist

## Использование ##

Прежде всего, ознакомьтесь с файлом Responder.conf и настройте его под свои нужды.

Запуск инструмента:

    ./Responder.py [options].

Типичный пример использования:

    ./Responder.py -I eth0 -Pv

Опции:

    --version показать номер версии программы и выйти
    -h, --help показать справочное сообщение и выйти
    -A, --analyze Режим анализа. Эта опция позволяет просматривать NBT-NS,
                        BROWSER, LLMNR запросы, не отвечая на них.
    -I eth0, --interface=eth0
                        Сетевой интерфейс для использования, можно использовать 'ALL' в качестве
                        подстановочный знак для всех интерфейсов
    -i 10.0.0.21, --ip=10.0.0.21
                        Используемый локальный IP-адрес (только для OSX)
    -6 2002:c0a8:f7:1:3ba8:aceb:b1a9:81ed, --externalip6=2002:c0a8:f7:1:3ba8:aceb:b1a9:81ed
                        Отравить все запросы с адресом IPv6, отличным от адреса
                        адреса ответчика.
    -e 10.0.0.22, --externalip=10.0.0.22
                        Отравить все запросы с IP-адресом, отличным от адреса
                        ответчика.
    -b, --basic Возвращать базовую HTTP-аутентификацию. По умолчанию: NTLM
    -d, --DHCP Включить ответы на широковещательные запросы DHCP. Эта
                        опция будет вставлять WPAD-сервер в DHCP-ответ.
                        По умолчанию: False
    -D, --DHCP-DNS Эта опция будет включать DNS-сервер в DHCP-ответ.
                        в ответ на запрос DHCP, в противном случае будет добавлен WPAD-сервер.
                        По умолчанию: False
    -w, --wpad Запуск неавторизованного прокси-сервера WPAD. Значение по умолчанию
                        False
    -u UPSTREAM_PROXY, --upstream-proxy=UPSTREAM_PROXY
                        Upstream HTTP-прокси, используемый неавторизованным WPAD-прокси для
                        исходящих запросов (формат: host:port)
    -F, --ForceWpadAuth Принудительная NTLM/Basic аутентификация при получении файла wpad.dat
                        при получении файла wpad.dat. Это может привести к появлению запроса на вход в систему. По умолчанию:
                        False
    -P, --ProxyAuth Принудительная NTLM (прозрачно)/Basic (с подсказкой)
                        аутентификации для прокси-сервера. WPAD не обязательно должен быть
                        ON. По умолчанию: False
    --lm Принудительное понижение хэширования LM для Windows XP/2003 и более ранних версий.
                        более ранних версий. По умолчанию: False
    --disable-ess Принудительное понижение уровня ESS. По умолчанию: False
    -v, --verbose Усилить многословность.


	

## Пожертвование ##

Вы можете внести свой вклад в этот проект, сделав пожертвование на следующий адрес $XLM (Stellar Lumens):

"GCGBMO772FRLU6V4NDUKIEXEFNVSP774H2TVYQ3WWHK4TEKYUUTLUKUH".

Paypal:

https://paypal.me/PythonResponder


## Благодарности ##

Разработка Late Responder стала возможной благодаря пожертвованиям, полученным от частных лиц и компаний.

Мы хотели бы поблагодарить основных спонсоров:

- SecureWorks: https://www.secureworks.com/

- Synacktiv: https://www.synacktiv.com/

- Black Hills Information Security: http://www.blackhillsinfosec.com/

- TrustedSec: https://www.trustedsec.com/

- Red Siege Information Security: https://www.redsiege.com/

- Open-Sec: http://www.open-sec.com/

- И все, ВСЕ пентестеры по всему миру, которые пожертвовали на этот проект.

Спасибо.


## Copyright ##

NBT-NS/LLMNR Responder

Responder - набор инструментов для захвата сети, созданный и поддерживаемый Лораном Гаффи (Laurent Gaffie).

e-mail: laurent.gaffie@gmail.com

Эта программа является свободным программным обеспечением: вы можете распространять ее и/или модифицировать
в соответствии с условиями Стандартной общественной лицензии GNU, опубликованной
Фондом свободного программного обеспечения, либо версии 3 этой лицензии, либо
(по вашему выбору) любой более поздней версии.

Эта программа распространяется в надежде, что она окажется полезной,
но БЕЗ КАКИХ-ЛИБО ГАРАНТИЙ; даже без подразумеваемых гарантий
товарности или пригодности для определенной цели.  См.
GNU General Public License для получения более подробной информации.

Вы должны были получить копию Стандартной общественной лицензии GNU
вместе с этой программой.  Если это не так, см. <http://www.gnu.org/licenses/>.
