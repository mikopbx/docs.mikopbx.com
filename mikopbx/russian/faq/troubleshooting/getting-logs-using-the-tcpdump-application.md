# Получение логов с помощью приложения tcpdump

**tcpdump** — это утилита в Linux для захвата и анализа сетевого трафика. Она позволяет в реальном времени отслеживать пакеты, проходящие через сетевой интерфейс, и сохранять их для дальнейшего анализа.

1. Подключитесь к вашей АТС по SSH ([инструкция](connecting-to-a-pbx-using-ssh/putty.md))
2. Выполните команду:

```php
tcpdump -i eth0 -n -s 0 -vvv -w /tmp/capturefilename.pcap
```

<figure><img src="../../.gitbook/assets/sshCode.png" alt=""><figcaption><p>Команда в интерфейсе SSH</p></figcaption></figure>

3. Воспроизведите Вашу ситуацию, выполните телефонный звонок. Далее нажмите в SSH консоли **CTRL + С**. Работа приложения **tcpdump** будет завершена.

<figure><img src="../../.gitbook/assets/sshResult.png" alt=""><figcaption><p>Результат работы tcpdump</p></figcaption></figure>

4. Подключитесь к АТС с помощью WinSCP ([инструкция](connecting-to-a-pbx-using-winscp.md))
5. Лог вызова **/tmp/capturefilename.pcap** отправляйте в техническую поддержку

<figure><img src="../../.gitbook/assets/fileinWinSCP.png" alt=""><figcaption><p>Файл "capturefilename.pcap"</p></figcaption></figure>
