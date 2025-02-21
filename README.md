# WhatsApp-PHP-message-parser

0. Intro
   WhatsApp (как и другие сервисы, например Youtube - revanced Youtube) блокирует модифицированные приложения (например, YoWhatsApp ). Это послужило причиной отказа от него, а желание сохранить историю чатов как часть личной жизни послужило причиной написания парсера базы данных сообщений SQLite msgstore.db. Схема базы постоянно меняется, так что у меня есть по меньшей мере 2 версии базы данных msgstore.db с разними схемами организации таблиц.  

Шаг 1: получить рашифрованную версию баз данных. Проще всего это было сделать в модифицириванных версиях приложения (YoWhatsApp и десятки подобных модов). В классическом приложении WhatsApp вам скорее всего придется колдовать с рутованием телефона, вытаскиванием из недр системы ключей и использованием утилит рашифровки *.crypt*-файлов.

