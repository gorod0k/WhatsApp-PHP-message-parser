# WhatsApp-PHP-message-parser

## Intro
   WhatsApp (как и другие сервисы, например Youtube - revanced Youtube) блокирует модифицированные приложения (например, YoWhatsApp ). Это послужило причиной отказа от него, а желание сохранить историю чатов как часть личной жизни послужило причиной написания парсера базы данных сообщений SQLite msgstore.db. Схема базы постоянно меняется, так что у меня есть по меньшей мере 2 версии базы данных msgstore.db с разними схемами организации таблиц.

## Ресурсы для изучения:
- Python утилита https://github.com/B16f00t/whapa/blob/master/libs/whapa.py
- статья Forensic Analysis of WhatsApp Messenger on Android Smartphones.
Это устаревшие, но весьма полезные источники для понимания общей организации хранения данных в WhatsApp Messenger.
AI ChatGPT / DeepSeek  тоже могут быть полезны, но они дают много недостоверной информации .

## Шаг 1: получить рашифрованную версию баз данных. Проще всего это было сделать в модифицириванных версиях приложения (YoWhatsApp и десятки подобных модов). В классическом приложении WhatsApp вам скорее всего придется колдовать с рутованием телефона, вытаскиванием из недр системы ключей и использованием утилит рашифровки *.crypt*-файлов.

## примеры SQLite-запросов
// print all tables from db with num of rows:
$msg_db = new SQLite3("msgstore.db"); // chat database
$wa_db = new SQLite3("wa.db");   // contacts database
$res = $wa_db->query("SELECT name FROM sqlite_master WHERE type='table'"); // type also m/b 'VIEW','INDEX','TRIGGER'
while ($row = $res->fetchArray())
{
	$tbl= $row[0];
	echo "$tbl ";
	echo '('.$wa_db->querySingle("SELECT COUNT(*) FROM $tbl ; ").') <br>';
}
