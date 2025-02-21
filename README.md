# WhatsApp-PHP-message-parser

## Intro
   WhatsApp (как и другие сервисы, например Youtube - revanced Youtube) блокирует модифицированные приложения (например, YoWhatsApp ). Это послужило причиной отказа от него, а желание сохранить историю чатов как часть личной жизни послужило причиной написания парсера базы данных сообщений SQLite msgstore.db. Схема базы постоянно меняется, так что у меня есть по меньшей мере 2 версии базы данных msgstore.db с разними схемами организации таблиц.

## Ресурсы для изучения:
- Python утилита https://github.com/B16f00t/whapa/blob/master/libs/whapa.py
- статья Forensic Analysis of WhatsApp Messenger on Android Smartphones.
Это устаревшие, но весьма полезные источники для понимания общей организации хранения данных в WhatsApp Messenger.
AI ChatGPT / DeepSeek  тоже могут быть полезны, но они дают много недостоверной информации .

## Tools
DB Browser SQLite
PHP Lite Admin

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

//	print tables from "msgstore.db"
$res = $msg_db->query("SELECT type,name,sql,tbl_name FROM main.sqlite_master; ");
while ($row = $res->fetchArray())
{
	$sql= $row[2];
	$tbl= $row[3]; 
	// echo "$row[0] $row[1] $row[3] <br>";
	//	table
	if ($row[0]=="table") 
		echo "$tbl (".$msg_db->querySingle("SELECT COUNT(*) FROM $tbl ; ").") <br>";
	//	view
	if ($row[0]=="view") 
		echo "$tbl (".$msg_db->querySingle("SELECT COUNT(*) FROM $tbl ; ").") <br>";
}
	*	*	*
 ## пример списка таблиц в базе данных wa.db YoWhatsApp (одна из из версий!)
 в скобках указано количество строк 

 	android_metadata (1)
	wa_contacts (10414)
	sqlite_sequence (8)
	system_contacts_version_table (45752)
	wa_vnames (225)
	wa_vnames_localized (0)
	wa_contact_storage_usage (6369)
	wa_biz_profiles (225)
	wa_biz_profiles_websites (55)
	wa_biz_profiles_hours (506)
	wa_group_descriptions (1)
	wa_group_admin_settings (13)
	wa_block_list (243)
	wa_biz_profiles_categories (240)
	wa_group_add_black_list (0)
	wa_props (17)
	wa_last_entry_point (0)
	wa_last_seen_block_list (0)
	wa_profile_photo_block_list (0)
	wa_about_block_list (0)
	wa_trusted_contacts (427)
	wa_trusted_contacts_send (1013)
	subgroup_info (0)
	group_relationship (0)
	wa_biz_profiles_linked_accounts_table (0)
 
 ## пример списка таблиц в базе данных msgstore.db YoWhatsApp (одна из из версий!) с комментариями 
 в скобках указано количество строк 

messages (293.102)
		* Основные идентификаторы
		_id — INTEGER Уникальный идентификатор сообщения.
		key_remote_jid — (TEL_NUM) JID (Jabber ID) of contact ..."@s.whatsapp.net" or group ..."@g.us" or ..."@temp" (rare) or "status@broadcast" (rare)
		key_from_me — Флаг message direction: '0'=incoming, '1'=outgoing
		key_id — (32_HEX) Уникальный идентификатор сообщения в чате.
		* Статус и таймстампы
		status — Статус доставки сообщения ('0'=received, '4'=waiting on server, '5'=received by the destination (?),'6'=control System message, 8 - - Audio played for target (key_from_me=1, media_wa_type=2), 10 - Audio played for me, '9' - Audio ??? for me, forwarded=32768 (rare), 12/13 - Seen)
			???what is this:	media_wa_type=15 AND edit_version=7 AND status=5
			when status==6:
			media_size=1 - changed subject
			media_size=2 - create group??? m/b change group avatar
			media_size=4 - was added to the group
			media_size=5 - left the group
			media_size=6 - changed the group icon
			media_size=7 - Removed from the list
			media_size=9 - created a broadcast list with recipients
			media_size=10 - changed to
			media_size=11 - created the group
			media_size=12 - added to the group
			media_size=13 - left the group
			media_size=14 - eliminated from the group
			media_size=15 - They made you administrator
			media_size=18 - The security code changed
			media_size=19 - Messages and calls in this chat are now protected with end-to-end encryption
			media_size=20 - joined using an invitation link from this group
			media_size=22 - This chat could be with a company account
			media_size=27 - changed the group description to
			media_size=28 - changed his phone number
			media_size=46 - This chat is with a company account
			media_size=50 - ?
			media_size=58 - block
			67 - ? Messages protected with end-to-end encryption???
			media_size=68 - ?
		needs_push — '2' if broadcast message, '0' otherwise (always 0)
		timestamp — Время отправки сообщения.
		received_timestamp — Время получения сообщения на устройстве.
		send_timestamp — unused (always set to '-1')
		receipt_server_timestamp — Время подтверждения доставки сервером.
		receipt_device_timestamp — Время подтверждения доставки устройством получателя.
		read_device_timestamp — Время, когда сообщение было прочитано.
		played_device_timestamp — Время, когда голосовое сообщение было воспроизведено.
		* Содержимое сообщения
		data — Текст сообщения (when message type: '0'=text)
		media_url — URL вложенного медиафайла when media_wa_type={1,2,3}
		media_mime_type — MIME-тип вложенного медиафайла (например, image/jpeg) when media_wa_type={1,2,3}
		media_wa_type — message type:
			-1="Start DB"
			0'=text
			1'=image, '2'=audio, '3'=video, '4'=contact vCard, '5'=geo position, 7-?GIF,8-call, 9-document, 
			10-? some empty with status=6,
			11-?
			13 - sticker/GIF? *all have mp4 mime_type
			14-anim.sticker
			15 - ?emoji-реакция???
			16-live location?
			20-image/webp kastom stikers
			24 - ? link to group???
			36 - ?
			42 - ? all jpeg mime_type
			???what is this:	media_wa_type=15 AND edit_version=7 AND status=5
		raw_data - thumbnail of the transmitted file when media_wa_type={'1','3'}
		media_size — Размер медиафайла в байтах when media_wa_type={1,2,3}; also see 'status'
		media_name — Имя файла when media_wa_type={1,2,3}
		media_caption — Подпись к медиафайлу.
		media_hash — Хеш файла для проверки целостности.
		media_duration — Длительность медиа (например, для видео/голосовых сообщений) when media_wa_type={1,2,3}
		origin - ???
			0 - ?
			1 - always media_wa_type=2
			3 - ? voices ???media_wa_type=2
		latitude / longitude — Координаты местоположения when media_wa_type=5
		thumb_image — BLOB Эскиз (превью) изображения или видео в Base64
		* Цепочки сообщений и цитаты
		remote_resource — JID отправителя в групповом чате
		quoted_row_id — Ссылка на цитируемое сообщение (при ответах).
		mentioned_jids — Список пользователей, упомянутых в сообщении (very-very rare)
		participant_hash — Идентификатор участника (в групповых чатах). связывает таблицу messages с таблицей group_participants
		* Пересылка, редактирование, удаление
		forwarded — битовaя маскa:
			??? Сообщение было переслано (forwarded).
			??? Сообщение является частью цепочки пересылки (например, переслано несколько раз).
			??? Сообщение было отправлено через широковещательный список (broadcast).
			??? Сообщение является уведомлением (например, системное сообщение).
			??? Сообщение было отправлено через WhatsApp Web или Desktop.
			??? Сообщение было отправлено через API (WhatsApp Business).
			??? Сообщение является голосовым сообщением.
			??? Сообщение является статусом (историей).
			??? Сообщение было отправлено через WhatsApp Business.
			??? Сообщение является частью шаблона (WhatsApp Business).
			??? Сообщение является частью мультимедийного сообщения (например, альбом).
			??? Сообщение было отправлено через групповой чат.
			??? Сообщение является частью голосового или видеозвонка.
			??? Сообщение является частью реакции (например, эмодзи).
			??? Сообщение является частью опроса.
			??? Сообщение является частью редактируемого сообщения.
		edit_version — Версия сообщения (для отредактированных сообщений).
		messages_revoked — Флаг (0/1), указывающий, было ли сообщение удалено для всех.
		* Дополнительные параметры
		recipient_count — Количество получателей (in groups).
		multicast_id — ID массовой рассылки (если сообщение было отправлено нескольким получателям).
		preview_type — Тип предварительного просмотра ссылки (например, карточка превью).
		send_count — Количество попыток отправки.
		lookup_tables — Битовая маска, указывающая, в каких индексах или FTS-таблицах содержится сообщение.
		future_message_type — Тип запланированного сообщения (если оно отложенное).
		message_add_on_flags — Флаги для дополнительных функций (rare 0/1, always NULL).
			??? 1- Сообщение содержит реакции (например, эмодзи).???
		
	sqlite_sequence (33)
	props (28)
	messages_fts (128488)
	messages_fts_content (128488)
	messages_fts_segments (551)
	messages_fts_segdir (11)
	messages_quotes (25932)
	messages_vcards (25)
	messages_vcards_jids (25)
	messages_edits (0)
	messages_links (1189)
	message_product (0)
	quoted_message_product (0)
	message_group_invite (1)
	message_quoted_group_invite_legacy (0)
	message_template (0)
	message_template_button (0)
	message_template_quoted (0)
	message_location (0)
	message_quoted_location (0)
	frequents (5)
	receipts (7688)
	message_mentions (0)
	group_participants (7610)
	group_participants_history (0)
	media_refs (227)
	message_thumbnails (3816)
	status_list (0)
	labels (0)
	labeled_messages (0)
	labeled_jids (0)
	pay_transactions (0)
	call_log (1549)
	call_log_participant_v2 (13)
	missed_call_logs (387)
	missed_call_log_participant (4)
	
	jid (39.533)
		id
		user - tel num / status(broadcast) / groups (g.us) / status_me / temp
		agent - always 0
		type:
			5 - status(broadcast)
			1 - groups (g.us)
			0/17 - user (s.whatsapp.net)
			2 - temp
		raw_string
		device
	
	chat (8.673)
		_id	Уникальный идентификатор чата (автоинкремент).
		jid_row_id	Уникальный идентификатор чата (связан с jid в таблице jid).
		hidden	Флаг скрытия чата (1 – скрыт, 0 – виден).
		subject	Название чата (используется в группах).
		created_timestamp	Время создания чата (Unix timestamp) - for groups
		display_message_row_id	ID сообщения, отображаемого в списке чатов (eq. last_message_row_id)
		last_message_row_id	ID последнего сообщения в чате.
		last_read_message_row_id	ID последнего прочитанного сообщения.
		last_read_receipt_sent_message_row_id	ID последнего сообщения, для которого отправлена квитанция о прочтении.
		last_important_message_row_id	ID последнего важного сообщения.
		archived	Флаг архивирования (1 – чат в архиве, 0 – нет).
		sort_timestamp	Время последнего изменения в чате (используется для сортировки).
		mod_tag	Внутренний идентификатор версии изменений чата.
		gen	Версия изменений чата в виде числа с плавающей запятой.
		spam_detection	Флаг спама (1 – подозрительный, 0 – нет).
		unseen_earliest_message_received_time	Время получения самого раннего непрочитанного сообщения.
		unseen_message_count	Количество непрочитанных сообщений.
		unseen_missed_calls_count	Количество пропущенных вызовов.
		unseen_row_count	Общее количество непросмотренных строк в чате (сообщения, реакции и т.д.).
		plaintext_disabled	Отключение простого текста (1 – включено шифрование, 0 – текст доступен).
		vcard_ui_dismissed	Флаг скрытия интерфейса VCard (1 – скрыто).
		change_number_notified_message_row_id	ID сообщения об изменении номера в чате.
		show_group_description	Флаг отображения описания группы (1 – показывать).
		ephemeral_expiration	Время исчезновения сообщений (если включены временные сообщения).
		last_read_ephemeral_message_row_id	ID последнего прочитанного исчезающего сообщения.
		ephemeral_setting_timestamp	Время последнего изменения настройки исчезающих сообщений.
		unseen_important_message_count	Количество непрочитанных важных сообщений.
		ephemeral_disappearing_messages_initiator	Кто включил исчезающие сообщения (1 – админ, 2 – пользователь).
		group_type	Тип группы (0 – обычная, 1 – сообщество, 2 – канал).
		last_message_reaction_row_id	ID последней реакции на сообщение в чате.
		last_seen_message_reaction_row_id	ID последней просмотренной реакции.
		unseen_message_reaction_count	Количество непросмотренных реакций в чате.
		growth_lock_level	Уровень блокировки роста чата (ограничение на добавление новых участников).
		growth_lock_expiration_ts	Время истечения блокировки роста (если есть).
		last_read_message_sort_id	ID последнего прочитанного сообщения для сортировки.
		display_message_sort_id	ID сообщения, отображаемого в списке чатов для сортировки.
		last_message_sort_id	ID последнего сообщения в чате для сортировки.
	
	message_forwarded (96)
	message (1)
	message_quoted (0)
	message_orphaned_edit (0)
	deleted_chat_job (0)
	receipt_device (144928)
	message_streaming_sidecar (93)
	message_quoted_product (0)
	message_quoted_group_invite (0)
	message_media (6821)
	message_media_interactive_annotation (0)
	message_media_interactive_annotation_vertex (0)
	message_quoted_mentions (0)
	message_quoted_media (0)
	message_vcard (0)
	message_vcard_jid (20)
	message_quoted_vcard (0)
	message_text (0)
	message_quoted_text (0)
	message_future (0)
	message_send_count (0)
	message_thumbnail (0)
	user_device (7472)
	message_link (0)
	receipt_user (154158)
	group_participant_device (10963)
	message_revoked (0)
	message_payment (0)
	message_payment_transaction_reminder (0)
	message_payment_status_update (0)
	message_system (0)
	message_system_group (0)
	message_system_value_change (0)
	message_system_number_change (0)
	message_system_photo_change (0)
	message_system_chat_participant (0)
	frequent (0)
	status (11)
	labeled_jid (0)
	message_ftsv2 (283452)
	message_ftsv2_content (283452)
	message_ftsv2_segments (1732)
	message_ftsv2_segdir (31)
	message_ftsv2_docsize (283452)
	message_ftsv2_stat (1)
	messages_hydrated_four_row_template (0)
	receipt_orphaned (51854)
	group_participant_user (7987)
	pay_transaction (0)
	media_hash_thumbnail (240)
	message_system_device_change (0)
	message_ephemeral_setting (9)
	message_system_block_contact (0)
	conversion_tuples (0)
	labeled_messages_fts (0)
	labeled_messages_fts_content (0)
	labeled_messages_fts_segments (0)
	labeled_messages_fts_segdir (0)
	away_messages (0)
	away_messages_exemptions (0)
	quick_replies (0)
	quick_reply_usage (0)
	quick_reply_keywords (0)
	keywords (0)
	quick_reply_attachments (0)
	message_media_vcard_count (0)
	message_external_ad_content (0)
	group_notification_version (15)
	message_system_ephemeral_setting_not_applied (0)
	message_ui_elements (0)
	user_device_info (3250)
	message_ephemeral (19)
	message_order (0)
	quoted_message_order (0)
	message_quoted_order (0)
	message_view_once_media (4)
	mms_thumbnail_metadata (4)
	message_system_initial_privacy_provider (0)
	message_system_business_state (0)
	message_quoted_ui_elements (0)
	message_ui_elements_reply (0)
	message_quoted_ui_elements_reply (0)
	message_privacy_state (0)
	message_invoice (0)
	message_quote_invoice (0)
	invoice_transactions (0)
	payment_background (0)
	payment_background_order (0)
	message_quoted_ui_elements_reply_legacy (0)
	played_self_receipt (0)
	message_payment_invite (0)
	message_quoted_payment_invite (0)
	messages_quotes_payment_invite_legacy (0)
	message_system_payment_invite_setup (0)
	message_broadcast_ephemeral (0)
	primary_device_version (3503)
	audio_data (465)
	joinable_call_log (0)
	message_rating (0)
	message_system_linked_group_call (0)
	message_status_psa_campaign (0)
	message_add_on (11)
	message_add_on_orphan (0)
	message_add_on_receipt_device (3)
	message_add_on_reaction (11)
	message_quoted_blank_reply (0)
	message_system_community_link_changed (0)

виртуальное представление (VIEW)	6 tables
	legacy_available_messages_view (293.081)
	message_view (293081)
	available_message_view (293.081)
	deleted_messages_view (0)
	deleted_messages_ids_view (0)
	chat_view (8673)
